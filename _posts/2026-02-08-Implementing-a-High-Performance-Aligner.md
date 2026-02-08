---
layout: post
title: Implementing a High Performance Sequence Aligner in C++
author: Dayo
category: Main 
---

# Building a High-Performance Sequence Aligner


Sequence alignment is the process of finding the optimal placement of a sequence within another, so we can determine probabilistically how related they are, and make further insights from that. As you might imagine, this can tell us just how different species are, how genes match across different people, and genetic similarities among samples in genomic studies.

Given the size of the human genome, which is about 3.5 billion base pairs, comparing a samples genome to a reference genome to find genetic similarities, or searching for gene placements within a reference genome can become quite a computationally intensive task especially during genome wide association studies (GWAS), which usually have thousands to tens of thousands of participants.

While speed is a concern, accuracy is more important in some usecases, as such, there are also different alignment methods, focusing on either global or local alignments, some allowing mismatches as well in the form of gaps or extensions or base mismatches.


## The Challenge of Modern Sequencing

In modern genomic studies and sequencing, we have the concept of short reads, which are gotten from a subjects sample and reassembled in line with a reference genome. This is due to the nature of modern sequence analyzers, which don't read entire genomes at once, but instead produce millions of DNA fragments. Short reads are usually 100-150bp, and more recently we are getting longer reads from 200-250bp.

In genomic studies, usually we would want to reassemble these short reads against the reference genome to get a representation of that subjects genome, and find out any genomic quirks they/their group might possess.


## Why Classic Algorithms Don't Scale

Before diving into modern approaches, let's understand why foundational algorithms hit performance walls.

### Needleman-Wunsch and Smith-Waterman

The **Needleman-Wunsch algorithm** performs optimal global alignment, while **Smith-Waterman** finds the best local alignment. Both use dynamic programming to fill a scoring matrix with matches, mismatches, and gaps (insertions/deletions).

These algorithms are elegant and accurate, with a time complexity of **O(M×N)**—where M is the reference genome length and N is the read length. The problem? When M is 3.5 billion bases and you have millions of reads, this quadratic complexity simply doesn't scale.

*For a 100bp read against the human genome: 3.5 billion × 100 = 350 billion operations per read.*

## Enter the Burrows-Wheeler Transform

Modern aligners like BWA (Burrows-Wheeler Aligner) take a radically different approach using the **Burrows-Wheeler Transform (BWT)** and **FM-Index**. Instead of comparing each read against the entire genome, they preprocess the genome into a searchable index.

### The BWA Pipeline

The alignment process follows four key stages:

1. **Suffix Array Construction** - Generate all suffixes of the reference genome and sort them lexicographically
2. **Burrows-Wheeler Transform** - Create a reversible permutation that clusters similar characters together
3. **FM-Index Construction** - Build a data structure for ultra-fast pattern matching
4. **Backward Search** - Search for reads in O(P) time, where P is the read length

The game-changer here is the FM-Index. For a reference genome of length M and a read of length P, it enables **O(P) lookup time**—completely independent of the genome size. A 100bp read takes 100 operations whether the genome is 1 million or 3.5 billion bases long.

### How the FM-Index Works

The FM-Index combines the BWT string with two key structures:

- **C-table**: Cumulative counts of characters in the alphabet
- **Occ-table**: Checkpointed occurrence counts throughout the BWT string

These enable backward search: starting from the last character of a read, we progressively narrow the search range within the BWT string until we've matched the entire read or determined it doesn't exist.

### The Suffix Array Bottleneck

Constructing the suffix array is expensive. A naive approach—generating all M suffixes and sorting them—has **O(M² log M)** complexity. For a 1GB genome, this is catastrophic.

The solution for this is **SA-IS (Suffix Array by Induced Sorting)**, a clever algorithm that achieves **linear O(M) construction time**. My implementation uses SA-IS to make the preprocessing step tractable.

## Implementation and Initial Results

I implemented the complete BWA pipeline with support for approximate matching (allowing mismatches). You can find the full code at [github.com/owolabioromidayo/bwa/](https://github.com/owolabioromidayo/bwa/).

### Stress Test: 1 Billion Bases

To establish a performance baseline, I ran a stress test on a 1GB reference genome with 10,000 reads of 100bp each:

```
Running TestStressApproximateMatch...
Genome loading time: 16.08 seconds
Genome length: 1,000,000,000 bases
Suffix Array construction time: 924.15 seconds (15.4 minutes)
BWT construction time: 46.68 seconds
FM-Index construction time: 723.64 seconds (12.0 minutes)
Pattern loading time: 0.03 seconds
Number of patterns: 10,000
Approximate matching time for 10,000 patterns (k=1): 47.41 seconds
Total matches found: 22,503
```

**Total preprocessing time: ~28 minutes**  
**Peak memory usage: ~10GB**

While the search itself is fast (47 seconds for 10,000 reads), the preprocessing is brutal. The 12-minute FM-Index construction time particularly stood out—this step should be much faster.

## Optimization Deep Dive


This eliminates the inner loop and uses a single memory block, dramatically improving cache performance. The FM-Index construction became a trivial fraction of the total time.

### Caching: Skip Preprocessing Entirely

Since the FM-Index only depends on the reference genome (which doesn't change), I implemented serialization to disk. Now the 28-minute preprocessing happens once, and subsequent runs load the index in seconds.

```cpp
// Save FM-Index to disk
fm_index.save("genome_index.bin");

// Load for instant startup
FMIndex fm_index = FMIndex::load("genome_index.bin");
```

This single optimization transformed the user experience: what was a 28-minute wait before every analysis became a one-time cost.



## Future optimizations



The 10GB memory usage for a 1GB genome needs work. Some ideas:



**FM-Index construction optimization** - The biggest bottleneck right now. Replace `std::map<char, std::vector<int>>` with `std::vector<std::array<int, ALPHABET_SIZE>>` and eliminate the nested loop:

```cpp
// Proposed fix
Occ_table.resize((bwt_string.length() / OCC_SAMPLE_RATE) + 1);
std::array current_counts{};

for (size_t i = 0; i < bwt_string.length(); ++i) {
    if (i % OCC_SAMPLE_RATE == 0) {
        Occ_table[i / OCC_SAMPLE_RATE] = current_counts;
    }
    current_counts[get_char_rank(bwt_string[i])]++;
}
```


**Genome storage** - Using std::string means one byte per base. DNA has 4 bases (A,C,G,T), which only needs 2 bits each. By packing 4 bases per byte, you save 75% memory. To handle special characters like 'N' and '$', 3-bit encoding is a good compromise, still saving 62.5%.

**Suffix array sampling** - The suffix array stores an integer position for every base. For 1 billion bases, that's 4GB. But you don't need every position. Store every 16th entry and reconstruct missing ones by walking backward through the FM-Index. This is a space-time tradeoff and should be configurable.

**Occ-table sampling** - The OCC_SAMPLE_RATE controls how often we checkpoint counts. I set it to 4096 for the stress test. Lower values = more memory but faster searches. Higher values = less memory but slower searches. Making this tunable gives users control.

**Parallelization** - The approximate matching loop is embarrassingly parallel. Each read is searched independently against the same FM-Index. Should be a near-linear speedup on multi-core CPUs.

**GPU acceleration** - If the FM-Index fits in GPU memory, searches could be much faster. Though the dynamic nature of approximate matching might not map well to GPU architecture.

**SAM output** - Need to generate output in standard SAM format for compatibility with other tools.

## Things I learned
Algorithm choice matters more than micro-optimizations. Going from O(MN) to O(P) search was the real win. No amount of cache tweaking would make Needleman-Wunsch competitive at this scale.

Profile before optimizing. I assumed suffix array construction was the main bottleneck. Turns out FM-Index construction was equally as bad.

The Occ sample rate thing. I didn't understand what the Occ sample rate was really doing until I implemented it. It's a checkpoint system  so we don't store counts at every position, we can just sample periodically and calculate intermediate values on the fly.

## Conclusion

There's still work to do, memory optimizations, parallelization, SAM output. But the core is solid and performs well enough that I learned what I wanted to learn.
This was a pretty fun project to learn about the weirdly performant data structures in bioinformatics.

For those interested in the implementation details, the complete source code with SA-IS suffix array construction, FM-Index implementation, and approximate matching is available at [github.com/owolabioromidayo/bwa/](https://github.com/owolabioromidayo/bwa/).

---


## References

- [Needleman-Wunsch Algorithm](https://en.wikipedia.org/wiki/Needleman%E2%80%93Wunsch_algorithm)
- [Smith-Waterman Algorithm](https://en.wikipedia.org/wiki/Smith%E2%80%93Waterman_algorithm)
- [Burrows-Wheeler Transform](https://en.wikipedia.org/wiki/Burrows%E2%80%93Wheeler_transform)
- [FM-Index](https://en.wikipedia.org/wiki/FM-index)
- [FM-Index Tutorial (YouTube)](https://www.youtube.com/watch?v=kvVGj5V65io)
- [Linear Suffix Array Construction by Induced Sorting (PDF)](https://www.researchgate.net/profile/Daricks-Wai-Hong-Chan/publication/221577802_Linear_Suffix_Array_Construction_by_Almost_Pure_Induced-Sorting/links/00b495318a21ba484f000000/Linear-Suffix-Array-Construction-by-Almost-Pure-Induced-Sorting.pdf)
- [Opportunistic Data Structures with Applications (PDF)](https://people.unipmn.it/manzini/papers/focs00draft.pdf)
- [BWA in Genomic Analysis (Carleton CS)](https://cs.carleton.edu/cs_comps/2324/sequenceAlignment/final-results/BWTFinalPaper23.pdf)
- [Stanford CS262: Computational Genomics](https://web.stanford.edu/class/cs262/archives/presentations/lecture4.pdf)

## Technical Specifications

**Implementation**: C++ with SA-IS suffix array construction  
**Features**: Approximate matching with configurable mismatches, serializable FM-Index  
**Memory**: ~10GB for 1GB genome (before optimizations)  
**Performance**: 47 seconds for 10,000 reads (100bp each, k=1 mismatches)