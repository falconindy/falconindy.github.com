---
title: "Con Kolivas is a Mad Man"
layout: article
categories: articles
---

I've recently gotten back into the insane (and mindless) hobby of compiling kernels for fun and purposelessness of:

* How fast I can compile another kernel (currently 6 minutes)
* How fast I can boot (slightly under 20 seconds)
* How fast I can find primes in the first 1,000,000 numbers (there's 78,498 of them)
* Masochistic pleasure

Up until recently, this involved a lot of trial and error, going back and forth with utilities like lsmod and hwdetect in order to strip out unneeded kernel modules and build in various crucial ones. I've been running without an init image for over a month now. Recently, I rediscovered the anaesthetist by day, kernel hacker by night Con Kolivas. I first heard about Con a few months back when he released "BFS":http://ck.kolivas.org/patches/bfs/bfs-faq.txt, a.k.a. the Brain Fuck Scheduler. The name alone inspires some amount of fear -- would it set my house on fire if I made one too many typos? BFS is a linear performance, single runqueue scheduler that's touted as being outrageously efficient for any workload, while remaining stable. Due to its lacking "scalability":http://xkcd.com/619/, BFS would never be included in the mainline kernel. It will, however, continue to exist and be developed as part of the -ck patchset.

There's some amount of drama surrounding Con and his involvement (and departure) with kernel development. In a "2007 interview":http://apcmag.com/interview_with_con_kolivas_part_2_his_effort_to_improve_linux_performance_on_the_desktop.htm, he notes that he initially got into kernel hacking as a hobby. Starting in 2003, he wrote some low impact code, learned from it, and then dug deeper into scheduling and desktop response. Eventually, major developers (Linus included) got wind of his work and asked him to clean it up for mainline submission. He proceeded to churn out some other projects in the realm of desktop performance: the Staircase Scheduler, Plugsched (a hotplug CPU scheduler to integrate the Staircase Scheduler), and the Staircase Deadline Scheduler . Fast forward, and Con is starting to feel like constant requests for bug fixes and resolution for corner cases is more of a job and less of an intellectual hobby. Throw in some more corner cases, a few extremely vocal users, and a medical condition (related to programming) that landed Con immobile for 6 weeks, and he decides to throw in the towel. In addition to the frustration, he also felt that mainline developers weren't concerned enough with desktop interactivity. Maybe it's just a "coincidence", but around the same time Con makes his departure, mainline adopts a linear performance scheduler called CFS (Completely Fair Scheduler), written by Ingo Molnar.

2 years later, in August of 2009, Con makes a reappearance with BFS and the revival of the -ck patchset. About a week ago, I finally had the courage to try not only BFS, but the entire patchset. In addition to BFS, also offers a 10000Hz timer frequency and a few other tweaks to beef up desktop response in the preemption sector. Consider him the ultimate advocate of the desktop user. The best part, of course, is that the patches work beautifully. Multi-tasking is faster, cleaner, and even while compiling in the background (yes, yet another kernel) I can run whatever else I want without a hitch. My "aging" Core2Duo doesn't even blink.

Welcome back, oh prolific poet of processing precision.

12/24 update: There's some "noise":http://doom10.org/index.php?topic=78.0 on the Linux Kernel Mailing List about a comparison of CFS and BFS. "Con chimes in":http://lkml.org/lkml/2009/12/18/490 a few days later.
