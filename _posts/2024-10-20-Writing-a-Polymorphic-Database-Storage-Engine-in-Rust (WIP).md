---
layout: post
title: Writing a Polymorphic Database Engine in Rust (WIP)
author: Dayo
category: Main 
---



### Introduction

I did a batch at the Recurse Center earlier this year. My main goal going into the batch was to work on some performance-based software engineering. Knowing no Rust, and wanting to deepn my knowledge in database systems, I spontaneously arrived on this as a project at day one. 
I have long postponed writing some coherent rendition of all I learned through this period, and I have decided to just bite the bullet and write about my progress anyways, months after basically abandoning the project.



### Motivation

I have already laid bare most of my motivation for embarking on this project. 
Good excuse to learn Rust,learn about database systems and query languages, learn about performance optimization, opportunity to do something interesting and stupid at the same time, could lead to the opportunity of learning about distributed systems, learn about performnace
optimization.
I also wanted to learn more about systems programming, and working on the database storage engine really helped build my capacity in that area. 
I always felt my SQL was weak since I hardly ever wrote any, using ORMs instead, and most resources I consumed on DB optimizations didn't stick either, so I saw this as the perfect growth opportunity for both.
I felt partly the same for the contents of DDIA, andd while my toy project hasn't yet reached the stage of caring about concepts such as ACID, or MVCC, or even simple transactions or replication setup yet, I hope I get to that point eventually.



### Dont fly in blind ( some database prerequisites)

While writing this, it became clear to me that some concepts would not be easily obvious without some background. This shoul contain the core things required to breeze through the rest of this article. I have Andy Pavlos Database Systems an Advanced database systems to thank for
99% of all I have learnt about database internals, and the screenshots I am about to share are from these courses, I shall provide the links.

[Just put some pavlo images]


[Intro to Database Systems - Andy Pavlo](https://www.youtube.com/playlist?list=PLSE8ODhjZXjbj8BMuIrRcacnQh20hmY9g)

#### Representation of a Database Query (the DB syntax tree)






### Language Design

Given I had a very limited timeline to work on this project, I wanted to get it going with as little hiccups as possible. As I had already followed the first half of the great Crafting Interpreters book in C++, a recursive descent parser felt like the fastest and most flexible way to get
my nebulous ideas solidified and further develop them. I hadn’t settled for a query language style at this point, but since I knew I was going to have to at least merge features from NoSQL and SQL, and given that I didn't see the ordering of SQL statements as logical, or its parsing mechanism as flexible, the RDP was the best way for me. I was able to get it going in about a week, and it was extremely easy to adapt new features as I developed my scope.

I wrote and developed a language grammar while implementing the parser. This is what I came up with:

```
program      => *declaration EOF

declaration  => varDecl | statement

varDecl  => "let" IDENTIFIER ( "=" data_expr) ";"
statement    => data_expr  ";" 

data_expr  => data_call | data_call  (  ‘JOIN’ | ‘LJOIN’’) data_call ‘ON’ expr 
data_call =>  variable | data_source(“.”method(arguments?)) 



method => filter  | groupby | orderby | limit | offset | max | min | sum | count | select | 
//arg restrictions
offset -> number ; limit -> number
orderby ATTRIBUTE
groupby ATTRIBUTE
filter  lambda_expr
select (ATTRIBUTE)*
select_distinct (ATTRIBUTE)*



ATTRIBUTE =>  IDENTIFIER(‘.’IDENTIFIER)
lambda_expr => ‘|ATTRIBUTE|’ expr

expr => logic_or

logic_or       → logic_and ( "or" logic_and )* 
logic_and      → equality ( "and" equality )* 
equality       → comparison ( ( "!=" | "==" ) comparison )* 
comparison     → term ( ( ">" | ">=" | "<" | "<=" ) term )* 
term           → factor ( ( "-" | "+" ) factor )* 
factor         → unary ( ( "/" | "*" ) unary )* 
unary          → ( "!" | "-" ) unary | primary


primary        → "true" | "false" | "null" |
               | NUMBER | STRING | IDENTIFIER |  ATTRIBUTE ;
```

Most things would look familiar for any language implementation, such as the build-up of primary expressions, and the definition of a program and a statement. The main changes exist in the definitions of data expressions and data calls, an the methos which they accept.
Data expressisons are basically the glue of a query syntax tree, allowing us to perform join operations on databases that have had predicates and other operations applied to them, bringing them into one stream for further processing.
Data calls, are what they are named, expressions that call some table from some database and wrap methods around it until it is in a state ready for the data expression.
This very nicely ties into the inefficient interpreter implementation which I currently have 😀.


In my effort to make scripting tenable and replace my verbose tests, I did quickly hack together some functionality for schema definition which I did not add to the grammar. I might discuss more on this later


Here is an example program using this language

```
        // Setup db and tables
        dbs.create_db('test_db');
        dbs.create_table('test_db' ,'test_table', 'DOCUMENT', 'ROW');

        dbs.create_table('test_db' ,'test_rtable', 'RELATIONAL', 'ROW', '{
            'name': 'string(50)',
            'balance': ['numeric', true],
            'pob': 'string',
            'active': 'boolean'
        }');

        //insert values
        dbs.insert('test_db', 'test_table', '{ 
                'name': 'John Doe',
                'age': 30.0,
                'city': 'New York',
                'address': {
                    'street': '123 Main St',
                    'zip': '10001'
                },
                'phone_numbers': [
                    '123-456-7890',
                    '987-654-3210'
                ]
        }');

        dbs.insert('test_db', 'test_table', '{
            'name': 'Jane Smith',
            'age': 25.0,
            'city': 'London',
            'address': {
                'street': '456 High St',
                'zip': 'SW1A 1AA'
            },
            'phone_numbers': [
                '020-1234-5678'
            ],
            'employment': {
                'company': 'Acme Inc.',
                'position': 'Software Engineer',
                'start_date': {
                'year': 2022.0,
                'month': 1.0
                }
            }
            }');

            dbs.insert('test_db', 'test_rtable', '{
                'name': 'Jane Smith',
                'balance': '2502034304.2332',
                'pob': 'London',
                'active': true
            }');

            dbs.insert('test_db', 'test_rtable', '{
                'name': 'John Doe',
                'balance': '450.2332',
                'pob': 'New York',
                'active': false
            }');
       
        //actual dataflow
        let x = dbs.test_db.test_table.offset(0);  
        let y = dbs.test_db.test_rtable.offset(0);  
        let z  = x LJOIN y ON name=name;
        z.limit(10);


```
As you can see from this code example, though not exhaustive, writing a query maps very nicely to constructing that syntax tree in your head, you can imagine the data flowing from the data calls to the expressions where they are joined, and then out again. This is what I always wanted,
and I believe there are some query languages like this worth checking out. I will list some here [TODO]


[TODO]
why I chose something dataflow-y and flexible
why not SQL?



### Language Implementation

The implementation of the language itself was not a tedious task, learning how to deal with rust was the main issue, given I did not go through the rust book or any other prerequisties. I did learn to fall in love with rust-analyzer, but it turns out I diddnt even unlock its full potnetial 
at this time. I was also very grateful for the package support provided and the ease of writing tests.


tokenization -> boring, already knowning

parsing -> wasnt anything too special either, just braindead following the grammar as given


Wnated to implement a typechecker but seemed much easier to just run through at interpreting stage and err out, subject to change tho
AST also didnt prove to be useful, just added complexity to unwrap / strip off. 


a lot of things are just structs for each of the different statements in the grammar, some are recursive, so just used box types.




## Interpreter Design

My goal was to transition from the final parsing stage into some intermediate representation that would map well with the syntax tree, so I could run some performance based rearrangement optimizations before passing down to a query executor. I was able to do this, but it turned out to be that this 
IR was only serving to make my approach more inflexible, and I scrapped it. I had already create custom recordditerator and predicate structs which kept track of constraints for datacall expressions, and I settled on some dynamic closure based interpreter form which made it so I could pass down these
predicates to the lowest data fetching operations, that it was so I didnt need the AST anymore.

Now, the use of a closure based interpreter, while kind of weird an cool, is highly inefficient, but I liked how stupid an fun it was that I had to leave it. Instead, for future purposes, I plan on making a sepearate compiler, so as not to harm my "art". how it works is very simple, and all the coe is 
contained in [LINK] . The main iea behind it is that in the dataflow language, the last operations depend on the pervious operations before them, an also ddepend on variables. variables can easily be substituted, and can be either ddata call or data expressions. So what we end up having is a nice query
tree anyways. So, what if, starting from the root / last statement, we recursively unwrapped the variables into their lowest forms, and eventually ended up with some sort of live being just pushing data up the tree on some .iter argument for it to be processed by its parents up to the roots.
ANd that is what I did! It was really nice to see it working, but rust and I had some serious isagreemenets I did not have the tact to solve, leaving me using Box::leak expressions for my closures, as I do not want to imagine the lifetime hell that awaits me.



AST

One of the main goals of the AST implementation is optimization. 
It has been a while since i looked through the code, but in its current state, it doesn't seem to have what I had in mind. 

I will quickly highlight some of the potential optimizations with the help of Prof Pavlo

(Predicate pushdown)
-> my take on this is to have a recorditerator attached to each dstacall and datarxpr with a predicate operator that can be algebraically combined with its parents. So all it would take is an initial pass for the root nodes to have the predicates optimally set.

(Tree refactoring) 
I haven't explored this fully / at all, but the goal is to rearrange options that filter out more variables to be performed first i.e. performing select operations before joins. 


using leaky boxes, why the other approach 


### Storage engine

The storage engine consists of a bunch of boring standard stuff, just simple record structures, supported datatype for relational and document databases, definitions of the 4 diff table types supported (RelationalRows, DocumentRows, RelationalColumn, DocumentColumn) alongside serialization / deserialization support for their page blocks (custom for Relational DBs, but using bson for Document DBs).

The core of the storage engine is just a multi-file pager, a page cache, and using b+ tree indexes. 


### The boring stuff?

Client side. Short of it is I have a basic implementation for a TCP server and client that allows me to send scripts to the DB server to execute. The pager has rwlocks in place, but I do not believe I have other concurrency handlers in place. 

Future plans

Benchmark inserting and updating 100k rows of data. 

Rewrite the closure-based interpreter into a compiler. 

Rewrite / fix my B+ tree implementation.

Insert-many operations.

Missing basic parts i.e. columnar relational storage support. 

Coalescing and optimizing operations that draw from both columnar and relational databases. Smoothing out the asymmetrical nature of how they are fetched i.e. columnar databases have many multiples of data stored in a page, since they store only one item per row. Data fetching from the interpreter is currently done in page multiples, so the columnar data would have to slow down for row access. 

Working on a Write-Ahead Log as a precursor for ACID properties (Atomicity, Consistency, Idempotency, Durability ).

Working on connection-pooling and multi-version concurrency control. 

Stop using Box::leak for my closure-based interpreter, might be completely fixed when I rewrite the whole thing anyways.
