---
title: "Recent updates to PMwCAS"
date: 2019-12-02T18:00:21-08:00
draft: true 
---

Update 12-03: Add group persist data

Update 12-02: Add more performance data

I've been working on PMwCAS for a few days, here are the recent updates, todo list, as well as my motivations.

For those who are not familiar with PMwCAS:

>PMwCAS is a library that allows atomically changing multiple 8-byte words on non-volatile memory in a lock-free manner. It allows developers to easily build lock-free data structures for non-volatile memory and requires no custom recovery logic from the application.

For more details, please check out Microsoft's GitHub [repo](https://github.com/microsoft/pmwcas), and [my fork](https://github.com/XiangpengHao/pmwcas).

I like the library for two reasons: 1) it provides useful building primitives for crash-consistent and multi-thread applications, 2) it is correct, it guarantees read-committed. (I'm not kidding, but most persistent memory research papers and data structures are just WRONG in terms of crash-consistency)

Some recent updates to the PMwCAS library:

- Fixed a non-linearity bug in multi-word cas algorithm. (Detected by another research group in Canada)

- Refactored the whole building system, it now has modern and efficient building process, it now also builds on most x86 machines. 

- Added a CI to automatic track the building and testing, several bugs are also fixed along with this process.

- Improved the performance by up to 100%.


### On improving the performance of PMwCAS

There're two main changes that impact the performance:

- Disallowed working threads to help each other. For any thread encountering a in-progress operations, it should keep retrying rather than helping the other thread.

- Removed the unnecessary address flushes after installing the descriptors. (Let me know if I'm not correct!) During recovery, the library can identify the in-progress descriptors by looking for **dirty undecided status**.
- Add more aggressive paddings when running on persistent memory.

- Group flushes (`clwb`)


Benchmark details:

- Eight threads running for 30 seconds, other parameters are kept default.
- Compiled by **clang++**, with all optimization enabled.
- NUMA bind, of course.


|                      | update count/s | success count/s |
|----------------------|--------------|---------------|
| original             | 711371    | 512794        |
| remove thread help   | 873892    | 609362     |
| remove extra persist | 1131266   | 762112     |
| add more padding     | 1450235   | 912708     |
| group persist        | 1650137   | 1011900    |

![](/img/pmwcas.png)

I also set up a [continuos benchmark](https://xiangpenghao.github.io/pmwcas/) to track the performance change by commit history. 

(One of my dreams is that every research project should have a CI, to track the { build | test | benchmark },
and automatically plot the performance change over time.
It not only helps the academia to deliver honest and reproducible results but also help the researchers to locate the problems.)


### Motivation

I'm working on improving PMwCAS not just to make my life busier but want to use PMwCAS to **build things that matter**. 

I started thinking about persistent memory about one year ago and witnessed lots of encouraging and frustrating moments in this area.
There are definitely not enough research efforts in making robust and efficient crash-consistent applications, 
and all existing frameworks are either working on inappropriate abstractions or making wrong assumptions. 

Thus I started the [VeryPM](https://github.com/XiangpengHao/VeryPM) project to build the whole persistent memory software (and eventually hardware) infrastructure from scratch.
(VeryPM is currently a part-time research project, I would be extremely happy if I have the opportunities to realize it during my Ph.D. life.)

The PMwCAS will be one of the essential building blocks in the VeryPM; it thus needs to be correct and efficient.


