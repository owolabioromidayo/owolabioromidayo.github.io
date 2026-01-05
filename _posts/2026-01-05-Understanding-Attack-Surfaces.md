---
layout: post
title: Understanding Attack Surfaces
author: Dayo
category: Main 
---


Before taking the OMSCS cybersecurity course, I thought I had a reasonable understanding of security. We used HTTPS, sanitized inputs, signed JWTs, rotated secrets, ran vulnerability scans. Security felt like something you layered onto a system once the “real” engineering was done.

That framing was wrong.

What the course actually taught me was not a collection of exploits, but how to *see* attack surfaces—how to look at a system and ask, very concretely: where can control, data, or assumptions leak? Once you start asking that question consistently, security stops being abstract and starts being structural.

An attack surface is not just an exposed endpoint or a missing auth check. It’s any place where user input, configuration, timing, or trust crosses a boundary without being fully constrained. Most of the course was, in one way or another, about learning to identify those boundaries.


Packet tracing was the first place this clicked. Network traffic looks overwhelming until you realize that most of it is irrelevant. Cloud infrastructure chatters constantly, but real attack surfaces sit quietly inside normal-looking flows. Reconstructing TCP and HTTP streams, filtering aggressively, and ignoring background noise taught me that leakage rarely announces itself. If credentials or flags show up on the wire, they usually do so in perfectly valid packets. The attack surface isn’t "the network", it’s the assumptions engineers make about what nobody will bother to inspect.

Binary exploitation made attack surfaces feel physical. Memory stopped being an internal implementation detail and became exposed terrain. Stack frames, return addresses and memory layout became points of leverage. Stepping through programs in GDB, overflowing buffers, redirecting execution, and chaining ROP gadgets made me realize that if you can influence memory layout or control flow, you own the program. Whether or not the source code is available is largely irrelevant. The attack surface is defined by what the binary allows, not by what the author intended.

API security brought the concept closer to modern application design. Tokens, endpoints, and permissions form an attack surface that’s often wider than teams realize. JWTs, in particular, are a frequent source of misplaced trust. When authorization data lives in a token, when secrets are weak, or when claims are assumed to be immutable, the attack surface quietly expands. Iterating over user IDs, replaying requests, modifying token payloads doesn't require cleverness. It only requires the system to trust more than it verifies.

Web security reinforced the idea that the browser itself is part of the attack surface. JavaScript executes with real authority. Cookies carry implicit permissions. Once XSS is possible, everything the user can do becomes reachable by an attacker. CSRF, CORS misconfigurations, client-side check are surfaces created by assuming the client will behave. Writing exploit scripts that simply act like users makes it obvious how thin those assumptions are.

Log4Shell was a lesson in hidden surfaces. Logging was never thought of as an execution vector, yet configuration, string interpolation, and delayed evaluation turned it into one. The important realization wasn’t the exploit mechanics, but the pattern that any system that interprets data later creates an attack surface now. Data at rest is not safe if it will eventually be processed in a more powerful context.

Even database security, which initially feels well-understood, revealed less obvious surfaces. SQL injection is direct and noisy. Inference attacks are not. You can leak sensitive information without ever querying it explicitly, simply by observing aggregates, filters, or response behavior. The attack surface is the query interface, as well as the statistical properties of the data you expose.

By the end of the course, security engineering stopped feeling like preventing exploits and started feeling like shrinking the space of possible misuse. Reducing entry points. Removing ambient authority. Enforcing strict boundaries. Assuming adversarial behavior by default. Accepting that software stacks evolve faster than any static defense.

Understanding attack surfaces changes how you write code. You become more suspicious of defaults, more careful with trust, and more aware that every abstraction leaks somewhere. I still feel there’s a lot more depth to grind through, be it harder CTFs, staying current with CVEs, reverse engineering work, re-implementing and testing old exploits. But as an early step in my OMSCS journey, this course was worth it and has changed how I view systems. 