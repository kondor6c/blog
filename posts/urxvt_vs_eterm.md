---
title: urxvt_vs_eterm 
date: 2016-10-14
category: devops
tags:
- linux
slug: urxvt_vs_eterm
description: Brief look at performance with perf and a couple terminals
---

====== urxvt vs Eterm ======
So I was trying different terminals so that I can keep on terminal just for a screencast sessions. I want to keep screencasting clean and partitioned. Additionally I have admitted to myself (now everyone else) that I need to know more about profiling the system, I got [http://www.brendangregg.com/books.html](Breddan Gregg's book ) on system performance, so far its very through. One complaint is that as a Solaris guy and even regarding many of his Linux presentations and writings is that he really favors dtrace, for obvious reasons; its a fantastic tracing tool. However due to license restrictions it isn't available on Linux (except for [ http://www.oracle.com/technetwork/articles/servers-storage-dev/dtrace-on-linux-1956556.html](OEL )). So I have first been reading up and trying out tools like perf. All the while I kept reading that urxvt has a lower footprint than many other terminals; in order to get some of these metrics they were using metrics [ https://www.void.gr/kargig/blog/2008/06/22/the-quest-for-a-better-rxvt-unicode-on-gentoo/](like ) [ https://forums.gentoo.org/viewtopic-t-539368-view-previous.html?sid=d495c38e4482f4e9e59e1f969fc45f2e](RSS ) and [ https://bbs.archlinux.org/viewtopic.php?id=125217](htop ) output.... While other use [ https://bbs.archlinux.org/viewtopic.php?id=65634](install size) and [ http://www.calno.com/evilvte/](striped binary size ).... tisk tisk. Where has the massive use of htop come from? I understand it has colors, but it doesn't bring much more to the table (and you have to install it)!
Well I remembered using Eterm, and that it had [ http://arstechnica.com/features/2000/03/simd/](SMID optimizations ), I thought that I would give perf a try to figure this out. This output is far from through, and is extremely basic (I would like to do some more analyzing)

```
[kondor6c@horse ~]$ perf stat -d Eterm

 Performance counter stats for 'Eterm':

        149.269413      task-clock:u (msec)       #    0.064 CPUs utilized
                 0      context-switches:u        #    0.000 K/sec
                 0      cpu-migrations:u          #    0.000 K/sec
             6,141      page-faults:u             #    0.041 M/sec
       170,152,520      cycles:u                  #    1.140 GHz                      (54.58%)
       333,404,464      instructions:u            #    1.96  insn per cycle           (65.90%)
        74,065,473      branches:u                #  496.187 M/sec                    (64.70%)
           898,795      branch-misses:u           #    1.21% of all branches          (65.13%)
        93,099,936      L1-dcache-loads:u         #  623.704 M/sec                    (54.92%)
         4,197,791      L1-dcache-load-misses:u   #    4.51% of all L1-dcache hits    (24.98%)
           323,876      LLC-loads:u               #    2.170 M/sec                    (29.76%)
            31,341      LLC-load-misses:u         #    9.68% of all LL-cache hits     (42.03%)

       2.348637133 seconds time elapsed

[kondor6c@horse ~]$ perf stat -d urxvt
urxvt: no visual found for requested depth 256, using default visual.
urxvt: no visual found for requested depth 256, using default visual.

 Performance counter stats for 'urxvt':

        221.960133      task-clock:u (msec)       #    0.096 CPUs utilized          
                 0      context-switches:u        #    0.000 K/sec                  
                 0      cpu-migrations:u          #    0.000 K/sec                  
             7,553      page-faults:u             #    0.034 M/sec                  
       283,231,245      cycles:u                  #    1.276 GHz                      (53.93%)
       464,723,521      instructions:u            #    1.64  insn per cycle           (64.73%)
       108,938,037      branches:u                #  490.800 M/sec                    (63.29%)
         1,682,643      branch-misses:u           #    1.54% of all branches          (61.20%)
       133,799,730      L1-dcache-loads:u         #  602.810 M/sec                    (58.34%)
         4,321,438      L1-dcache-load-misses:u   #    3.23% of all L1-dcache hits    (26.90%)
           832,193      LLC-loads:u               #    3.749 M/sec                    (29.48%)
           122,159      LLC-load-misses:u         #   14.68% of all LL-cache hits     (41.89%)

       2.315625428 seconds time elapsed

[kondor6c@horse ~]$
```

I ran ps auxf as quick as I could, far from scientific. But the results are interesting. I did add a few color settings to .Xdefaults (solarized and tab support).

In short, try to think about other tools at our disposal, we are fortunate to not have a few tools that a vendor distributes, instead many great people have encountered problems and have written tools that we can use. I try to avoid top output unless I'm in super quick triage kind of situation, where I need to understand what is going on in a disaster. Additionally plan ahead because I don't believe that perf is installed by default on many distributions.

{{tag>urxvt Desktop}}



