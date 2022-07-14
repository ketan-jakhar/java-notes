# JAVA NOTES

## Multi Threading

### Introduction

#### What good is concurrency?

- Threads like most computer science concepts aren't physical objects. The closest tangible manifestation of threads can be seen in a debugger.

![Screenshot (54)](https://user-images.githubusercontent.com/70067813/178968261-7f532bcd-c525-42f2-bc70-aa3196f25eba.png)

- This image shows suspended threads in a debugger

Threads can give the illusion of multitasking even though at any given point in time the CPU is executing only one thread.
Each thread gets a slice of time on the CPU and then gets switched out either because it initiates a task which requires waiting and not utilizing the CPU or it completes its time slot on the CPU.

#### Benefits of Threads
- Higher throughput(Throughput is a measure of how many units of information a system can process in a given amount of time), though in some pathetic scenarios it is possible to have the overhead of context switching among threads steal away any throughput gains and result in worse performance than a single-threaded scenario. However such cases are unlikely and an exception, rather than the norm.

- Responsive applications that give the illusion of multi-tasking.

- Efficient utilization of resources. Note that thread creation is light-weight in comparison to spawning a brand new process. Web servers that use threads instead of creating new processes when fielding web requests, consume far fewer resources.
<br/>
All other benefits of multi-threading are extensions of or indirect benefits of the above.

#### Performance Gains via Multi-Threading
- The performance gains can be many folds depending on the availability of multiple CPUs and the nature of the problem being solved. However, there will always be problems that don't yield well to a multi-threaded approach and may very well be solved efficiently using a single thread.

#### Problems with Threads
- Usually very hard to find bugs: some that may only rear head in production environments
- Higher cost of code maintenance: since the code inherently becomes harder to reason about
- Increased utilization of system resources: Creation of each thread consumes additional memory, CPU cycles for book-keeping and waste of time in context switches.
- Programs may experience slowdown: as coordination amongst threads comes at a price. Acquiring and releasing locks adds to program execution time. Threads fighting over acquiring locks cause lock contention.

### Program vs Process vs Thread

#### Program
<p>A program is a set of instructions and associated data that resides on the disk and is loaded by the operating system to perform some task.</p>
<p>Eg: A program is a set of instructions and associated data that resides on the disk and is loaded by the operating system to perform some task.</p>
<p>In order to run a program, the operating system's kernel is first asked to create a new process, which is an environment in which a program executes.</p>
<br/>
- <b>Kernel</b>: The kernel is a computer program at the core of a computer's operating system and generally has complete control over everything in the system.

#### Process
<p>A process is a program in execution.</p>
<p>A process is an execution environment that consists of instructions, user-data, and system-data segments, as well as lots of other resources such as CPU, memory, address-space, disk and network I/O acquired at runtime.</p>
<p>A program can have several copies of it running at the same time but a process necessarily belongs to only one program.</p>

#### Thread
<p>Thread is the smallest unit of execution in a process.</p>
<p>Usually, there would be some state associated with the process that is shared among all the threads and in turn each thread would have some state private to itself.</p>
<p>The globally shared state amongst the threads of a process is visible and accessible to all the threads, and special attention needs to be paid when any thread tries to read or write to this global shared state. There are several constructs offered by various programming languages to guard and discipline the access to this global state, which we will go into further detail</p>

#### Caveats
<p>There's also the concept of "multiprocessing" systems, where multiple processes get scheduled on more than one CPU. Usually, this requires hardware support where a single system comes with multiple cores or the execution takes place in a cluster of machines. Processes don't share any resources amongst themselves whereas threads of a process can share the resources allocated to that particular process, including memory address space. However, languages do provide facilities to enable inter-process communication.</p>

![Screenshot (56)](https://user-images.githubusercontent.com/70067813/178974025-25ede935-3213-42d0-82f2-d49788edee4a.png)

---

<p>Below is an example highlighting how multi-threading necessitates caution when accessing shared data amongst threads. Incorrect synchronization between threads can lead to wildly varying program outputs depending on in which order threads get executed.</p>

```java
int counter = 0; // 1
// 2
void incrementCounter() { // 3
   counter++; // 4
} // 5
```

<p>
The increment on line 4 is likely to be decompiled into the following steps on a computer:

- Read the value of the variable counter from the register where it is stored
- Add one to the value just read
- Store the newly computed value back to the register
</p>

<p>
Lets call one thread as T1 and the other as T2. Say the counter value is equal to 7.

- T1 is currently scheduled on the CPU and enters the function. It performs step A i.e. reads the value of the variable from the register, which is 7.

- The operating system decides to context switch T1 and bring in T2.

- T2 gets scheduled and luckily gets to complete all the three steps A, B and C before getting switched out for T1. It reads the value 7, adds one to it and stores 8 back.

- T1 comes back and since its state was saved by the operating system, it still has the stale value of 7 that it read before being context switched. It doesn't know that behind its back the value of the variable has been updated. It unfortunately thinks the value is still 7, adds one to it and overwrites the existing 8 with its own computed 8. If the threads executed serially the final value would have been 9.
</p>

### Concurrency vs Parallelism

#### Serial execution
<p>When programs are serially executed, they are scheduled one at a time on the CPU. Once a task gets completed, the next one gets a chance to run. Each task is run from the beginning to the end without interruption. The analogy for serial execution is a circus juggler who can only juggle one ball at a time. Definitely not very entertaining!</p>

#### Concurrency

<p>A concurrent program is one that can be decomposed into constituent parts and each part can be executed out of order or in partial order without affecting the final outcome.</p>
<p>A system capable of running several distinct programs or more than one independent unit of the same program in overlapping time intervals is called a concurrent system.</p>
<p>The execution of two programs or units of the same program may not happen simultaneously.</p>

- A concurrent system can have two programs in progress at the same time where progress doesn't imply execution.
- In concurrent systems, the goal is to maximize throughput and minimize latency.
