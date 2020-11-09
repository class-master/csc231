---
title: "Memory hierarchy and cache memories"
teaching: 0
exercises: 0
questions:
- "What else is there besides RAM?"
- "How different levels of the memory hierarchy can affect performance?"
objectives:
- "Know the different levels of the memory hierarchy."
- "Understand performance differences in accessing different memory levels."
keypoints:
- "First key point. Brief Answer to questions. (FIXME)"
---

> ## 1. Memory abstraction: writing and reading memory
>
> - Write:
>   - Transfer data from CPU to memory: `movq 8(%rsp), %rax`
>   - `Store` operation
> - Read:
>   - Trasnfer data from memory to CPU: `movq %rax, 8(%rbp)`
>   - `Load` operation
> - Physical representation of this abstraction:
> 
> <img src="../fig/05-memory/01.png" alt="CPU, memory, and bus" style="height:200px">
{: .slide}


> ## 2. Random-Access Memory (RAM)
>
> - Key features:
>   - RAM is traditionally packaged as a chip, or embedded as part of processor chip
>   - Basic storage unit is normally a cell (one bit per cell).
>   - Multiple RAM chips form a memory.
> - RAM comes in two varieties:
>   - SRAM (Static RAM): trasistors only
>   - DRAM (Dynamic RAM): transistor and capacitor
>   - Both are volatile: memory goes away without power. 
> 
>      | Transistor per bit | Access time | Need refresh  | Needs EDC  | Cost | Applications                  |
> -----| ------------------ | ----------- | ------------- | ---------- | ---- | ----------------------------- | 
> SRAM | 6 or 8             | 1x          | No            | Maybe      | 100x | Cache memories                |
> DRAM | 1                  | 10x         | Yes           | Yes        | 1x   | Main memories, frame buffers  |
> 
> *EDC: Error Detection and Correction*
> 
> - Trends: 
>   - SRAM scales with semiconductor technology
>     - Reaching its limits
>   - DRAM scaling limited by need for minimum capacitance
>     - Aspect ratio limits how deep can make capacitor
>     - Also reaching its limits
{: .slide}


> ## 3. Enhanced DRAMs
>
> - Operation of DRAM cell has not changed since its invention
>   - Commercialized by Intel in 1970. 
> - DRAM cores with better interface logic and faster I/O :
>   - Synchronous DRAM (SDRAM)
>     - Uses a conventional clock signal instead of asynchronous control
>   - Double data-rate synchronous DRAM (DDR SDRAM)
>     - Double edge clocking sends two bits per cycle per pin
>     - Different types distinguished by size of small prefetch buffer:
>       - DDR (2 bits), DDR2 (4 bits), DDR3 (8 bits), DDR4 (16 bits)
>     - By 2010, standard for most server and desktop systems
>   - Intel Core i7 supports DDR3 and DDR4 SDRAM
{: .slide}


> ## 4. The CPU-Memory gap
> 
> <img src="../fig/05-memory/02.png" alt="CPU memory gap" style="height:200px">
> 
> - The key to bridging this gap is a fundamental property of computer programs known as **locality**:
>   - Principle of Locality: Programs tend to use data and instructions with addresses near or equal 
>   to those they have used recently
>   - `Temporal locality`:  Recently referenced items are likely to be referenced again in the near future
>   - `Spatial locality`:  Items with nearby addresses tend to be referenced close together in time
> 
> ~~~
> sum = 0;
> for (i = 0; i < n; i++)
>   sum += a[i];
> return sum;
> ~~~
> {: .language-c}
> 
> - Data references
>   - Reference array elements in succession (stride-1 reference pattern): `spatial`
>   - Reference variable `sum` each iteration: `temporal`
> - Instruction references
>   - Reference instructions in sequence: `spatial`
>   - Cycle through loop repeatedly: `temporal`
{: .slide}


> ## 5. Qualitative estimates of locality
> 
> - Being able to look at code and get a qualitative sense of its locality is among the 
> key skills for a professional programmer.
> 
> > ## Exercise 1
> >
> > Does this function have good locality with respect to array `a`?
> >
> > ~~~
> > int sum_array_rows(int a[M][N]) {
> >   int i, j, sum = 0;
> >   for (i = 0; i < M; i++)
> >     for (j = 0; j < N; j++)
> >       sum += a[i][j];
> >   return sum;
> > }
> > ~~~
> > {: .language-c}
> >
> > *Hint: array layout is row-major order*
> >
> > > ## Answer     
> > > 
> > > Yes
> > > 
> > > <img src="../fig/05-memory/03.png" alt="stride-1 array access pattern" style="height:100px">
> > >
> > {: .solution}
> {: .challenge}
> 
> > ## Exercise 2
> >
> > Does this function have good locality with respect to array `a`?
> >
> > ~~~
> > int sum_array_rows(int a[M][N]) {
> >   int i, j, sum = 0;
> >   for (j = 0; j < N; j++)
> >     for (i = 0; i < M; i++)
> >       sum += a[i][j];
> >   return sum;
> > }
> > ~~~
> > {: .language-c}
> >
> > *Hint: array layout is row-major order*
> >
> > > ## Answer     
> > > 
> > > No
> > > 
> > > <img src="../fig/05-memory/03.png" alt="stride-1 array access pattern" style="height:100px">
> > >
> > {: .solution}
> {: .challenge}
{: .slide}


> ## 6. Hands on: performance measurement of locality
>
> - In your home directory, create a directory called `05-memory` and change into this 
> directory.
> - Create a file named `sum.c` with the following contents:
>
> <script src="https://gist.github.com/linhbngo/d1e9336a82632c528ea797210ed0f553.js?file=sum.c"></script>
>
> - Compile and run several times. 
> - Observe the performance difference. 
>
> ~~~
> $ gcc -Og -o sum sum.c
> $ ./sum
> $ ./sum
> $ ./sum
> $ ./sum
> ~~~
> {: .language-bash}
>
> <img src="../fig/05-memory/04.png" alt="differences in performance due to access pattern" style="height:400px">
>
{: .slide}


> ## 7. Memory hierarchies
> 
> - Some fundamental and enduring properties of hardware and software:
>   - Fast storage technologies cost more per byte, have less capacity, 
>   and require more power (heat!). 
>   - The gap between CPU and main memory speed is widening.
>   - Well-written programs tend to exhibit good locality.
> - These fundamental properties complement each other beautifully.
> - They suggest an approach for organizing memory and storage systems 
> known as a memory hierarchy.
>
> <img src="../fig/05-memory/05.png" alt="memory hierarchy" style="height:400px">
{:. slide}

> ## 8. Caching
> 
> - Cache: A smaller, faster storage device that acts as a staging area for a subset of the data in a larger, 
> slower device.
> - Fundamental idea of a memory hierarchy:
>   - For each `k`, the faster, smaller device at level `k` serves as a cache for the larger, 
>   slower device at level `k+1`.
> - Why do memory hierarchies work?
>   - Because of locality, programs tend to access the data at level `k` more often than they access the 
>   data at level `k+1`. 
>   - Thus, the storage at level k+1 can be slower, and thus larger and cheaper per bit.
> - Big Idea (Ideal):  The memory hierarchy creates a large poolof storage that costs as much as the cheap 
> storage near the bottom, but that serves data to programs at the rate of the fast storage near the top.
{:. slide}


> ## 8. Caching: general concepts
> 
> <img src="../fig/05-memory/06.png" alt="cache concepts" style="height:400px">
> <img src="../fig/05-memory/07.png" alt="cache hits" style="height:400px">
> <img src="../fig/05-memory/08.png" alt="cache misses" style="height:400px">
>
> - Cold (compulsory) miss
>   - Cold misses occur because the cache starts empty and this is the first reference to the block.
> - Capacity miss
>   - Occurs when the set of active cache blocks (working set) is larger than the cache.
> - Conflict miss
>   - Most caches limit blocks at level `k+1` to a small subset (sometimes a singleton) of the 
>   block positions at level `k`.
>     - E.g. Block i at level `k+1` must be placed in block (`i mod 4`) at level `k`.
>   - Conflict misses occur when the level `k` cache is large enough, but multiple data objects all map 
>   to the same level `k` block.
>     - E.g. Referencing blocks 0, 8, 0, 8, 0, 8, ... would miss every time.
>
> | Cache Type            | What is cached       | Where is it cached  | Latency (cycles) | Managed By       |
> | --------------------- | -------------------- | ------------------- | ---------------- | ---------------- | 
> | Register              | 4-6 byte words       | CPU core            | 0                | Compiler         |
> | TLB                   | Address translations | On-chip TLB         | 0                | Hardware MMU     |
> | L1 cache              | 64-byte blocks       | On-chip L1          | 4                | Hardware         |
> | L2 cache              | 64-byte blocks       | On-chip L2          | 10               | Hardware         |
> | Virtual memory        | 4-KB pages           | Main memory         | 100              | Hardware + OS    |
> | Buffer cache          | Part of files        | Main memory         | 100              | OS               |
> | Disk cache            | Disk sectors         | Disk controller     | 100,000          | Disk firmware    |
> | Network buffer cache  | Part of files        | Local disk          | 10,000,000       | NFS client       |
> | Browser cache         | Web pages            | Local disk          | 10,000,000       | Web browser      |
> | Web cache             | Web pages            | Remote server disks | 1,000,000,000    | Web proxy server |
{: .slide}





{% include links.md %}
