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

---

#### Performance Gains via Multi-Threading

- The performance gains can be many folds depending on the availability of multiple CPUs and the nature of the problem being solved. However, there will always be problems that don't yield well to a multi-threaded approach and may very well be solved efficiently using a single thread.

---

#### Problems with Threads

- Usually very hard to find bugs: some that may only rear head in production environments
- Higher cost of code maintenance: since the code inherently becomes harder to reason about
- Increased utilization of system resources: Creation of each thread consumes additional memory, CPU cycles for book-keeping and waste of time in context switches.
- Programs may experience slowdown: as coordination amongst threads comes at a price. Acquiring and releasing locks adds to program execution time. Threads fighting over acquiring locks cause lock contention.

---

### Program vs Process vs Thread

#### Program

<p>A program is a set of instructions and associated data that resides on the disk and is loaded by the operating system to perform some task.</p>
<p>Eg: A program is a set of instructions and associated data that resides on the disk and is loaded by the operating system to perform some task.</p>
<p>In order to run a program, the operating system's kernel is first asked to create a new process, which is an environment in which a program executes.</p>
<br/>
- <b>Kernel</b>: The kernel is a computer program at the core of a computer's operating system and generally has complete control over everything in the system.

---

#### Process

<p>A process is a program in execution.</p>
<p>A process is an execution environment that consists of instructions, user-data, and system-data segments, as well as lots of other resources such as CPU, memory, address-space, disk and network I/O acquired at runtime.</p>
<p>A program can have several copies of it running at the same time but a process necessarily belongs to only one program.</p>

---

#### Thread

<p>Thread is the smallest unit of execution in a process.</p>
<p>Usually, there would be some state associated with the process that is shared among all the threads and in turn each thread would have some state private to itself.</p>
<p>The globally shared state amongst the threads of a process is visible and accessible to all the threads, and special attention needs to be paid when any thread tries to read or write to this global shared state. There are several constructs offered by various programming languages to guard and discipline the access to this global state, which we will go into further detail</p>

---

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

---

### Concurrency vs Parallelism

#### Serial execution

<p>When programs are serially executed, they are scheduled one at a time on the CPU. Once a task gets completed, the next one gets a chance to run. Each task is run from the beginning to the end without interruption. The analogy for serial execution is a circus juggler who can only juggle one ball at a time. Definitely not very entertaining!</p>

---

#### Concurrency

<p>A concurrent program is one that can be decomposed into constituent parts and each part can be executed out of order or in partial order without affecting the final outcome.</p>
<p>A system capable of running several distinct programs or more than one independent unit of the same program in overlapping time intervals is called a concurrent system.</p>
<p>The execution of two programs or units of the same program may not happen simultaneously.</p>

- A concurrent system can have two programs in progress at the same time where progress doesn't imply execution.
- <b>In concurrent systems, the goal is to maximize throughput and minimize latency. Concurrent systems achieve lower latency and higher throughput when programs running on the system require frequent network or disk I/O.</b>
<p>Eg: The classic example of a concurrent system is that of an operating system running on a single core machine. Such an operating system is concurrent but not parallel. It can only process one task at any given point in time but all the tasks being managed by the operating system appear to make progress because the operating system is designed for concurrency. Each task gets a slice of the CPU time to execute and move forward.</p>
<p>Going back to our circus analogy, a concurrent juggler is one who can juggle several balls at the same time. However, at any one point in time, he can only have a single ball in his hand while the rest are in flight. Each ball gets a time slice during which it lands in the juggler's hand and then is thrown back up. A concurrent system is in a similar sense juggling several processes at the same time.</p>

---

#### Parallelism

<p>A parallel system is one which necessarily has the ability to execute multiple programs at the same time.</p>
<p>Usually, this capability is aided by hardware in the form of multicore processors on individual machines or as computing clusters where several machines are hooked up to solve independent pieces of a problem simultaneously.</p>

- Remember an individual problem has to be concurrent in nature, that is portions of it can be worked on independently without affecting the final outcome before it can be executed in parallel.
- <b>In parallel systems the emphasis is on increasing throughput and optimizing usage of hardware resources. The goal is to extract out as much computation speedup as possible.</b>

<p>Example problems include matrix multiplication, 3D rendering, data analysis, and particle simulation.</p>
<p>Revisiting our juggler analogy, a parallel system would map to at least two or more jugglers juggling one or more balls. In the case of an operating system, if it runs on a machine with say four CPUs then the operating system can execute four tasks at the same time, making execution parallel. Either a single (large) problem can be executed in parallel or distinct programs can be executed in parallel on a system supporting parallel execution.</p>

---

#### Concurrency vs Parallelism

- From the above discussion it should be apparent that a concurrent system need not be parallel, whereas a parallel system is indeed concurrent. Additionally, a system can be both concurrent and parallel e.g. a multitasking operating system running on a multicore machine
- Concurrency is about dealing with lots of things at once.
- Parallelism is about doing lots of things at once.
  <br/>

- Eg: We end the lesson with an analogy, frequently quoted in online literature, of customers waiting in two queues to buy coffee. Single-processor concurrency is akin to alternatively serving customers from the two queues but with a single coffee machine, while parallelism is similar to serving each customer queue with a dedicated coffee machine.

---

### Cooperative Multitasking vs Preemptive Multitasking

A system can achieve concurrency by employing one of the following multitasking models:

---

#### Preemptive Multitasking

- In preemptive multitasking, the operating system preempts a program to allow another waiting task to run on the CPU. Programs or threads can't decide how long for or when they can use the CPU. The operating system’s scheduler decides which thread or program gets to use the CPU next and for how much time.
- Furthermore, scheduling of programs or threads on the CPU isn’t predictable. A thread or program once taken off of the CPU by the scheduler can't determine when it will get on the CPU next. As a consequence, if a malicious program initiates an infinite loop, it only hurts itself without affecting other programs or threads. Lastly, the programmer isn't burdened to decide when to give up control back to the CPU in code.
- Mainly used in all PC OS

---

#### Cooperative Multitasking

- Cooperative Multitasking involves well-behaved programs to voluntarily give up control back to the scheduler so that another program can run. A program or thread may give up control after a period of time has expired or if it becomes idle or logically blocked. The operating system’s scheduler has no say in how long a program or thread runs for.
- A malicious program can bring the entire system to a halt by busy waiting or running an infinite loop and not giving up control. The process scheduler for an operating system implementing cooperative multitasking is called a cooperative scheduler. As the name implies, the participating programs or threads are required to cooperate to make the scheduling scheme work.

---

### Synchronous vs Asynchronous

#### Synchronous

<p>Synchronous execution refers to line-by-line execution of code.</p>

- Synchronous execution blocks at each method call before proceeding to the next line of code. A program executes in the same sequence as the code in the source code file. Synchronous execution is synonymous to serial execution.

---

#### Asynchronous

<p>Asynchronous (or async) execution refers to execution that doesn't block when invoking subroutines.</p>

- By wiki: Asynchronous programming is a means of parallel programming in which a unit of work runs separately from the main application thread and notifies the calling thread of its completion, failure or progress.
- An asynchronous program doesn’t wait for a task to complete and can move on to the next task.
- In non-threaded environments, asynchronous programming provides an alternative to threads in order to achieve concurrency and fall under the cooperative multitasking model.

---

### I/O Bound vs CPU Bound

We write programs to solve problems. Programs utilize various resources of the computer systems on which they run. For instance a program running on your machine will broadly require:

- CPU Time

- Memory

- Networking Resources

- Disk Storage

<p>Depending on what a program does, it can require heavier use of one or more resources. For instance, a program that loads gigabytes of data from storage into main memory would hog the main memory of the machine it runs on. Another can be writing several gigabytes to permanent storage, requiring abnormally high disk i/o.</p>

---

#### CPU Bound

<p>Programs which are compute-intensive i.e. program execution requires very high utilization of the CPU (close to 100%) are called CPU bound programs. Such programs primarily depend on improving CPU speed to decrease program completion time. This could include programs such as data crunching, image processing, matrix multiplication etc.</p>

---

#### I/O Bound

<p>I/O bound programs are the opposite of CPU bound programs. Such programs spend most of their time waiting for input or output operations to complete while the CPU sits idle.</p>

- I/O operations can consist of operations that write or read from main memory or network interfaces.
- Because the CPU and main memory are physically separate a data bus exists between the two to transfer bits to and fro. Similarly, data needs to be moved between network interfaces and CPU/memory. Even though the physical distances are tiny, the time taken to move the data across is big enough for several thousand CPU cycles to go waste. This is why I/O bound programs would show relatively lower CPU utilization than CPU bound programs.

---

#### Caveats

Both types of programs can benefit from concurrent architectures.

- If a program is CPU bound we can increase the number of processors and structure our program to spawn multiple threads that individually run on a dedicated or shared CPU.
- For I/O bound programs, it makes sense to have a thread give up CPU control if it is waiting for an I/O operation to complete so that another thread can get scheduled on the CPU and utilize CPU cycles.

---

### Throughput vs Latency

#### Throughput

Throughput is defined as the rate of doing work or how much work gets done per unit of time. If you are an Instagram user, you could define throughput as the number of images your phone or browser downloads per unit of time.

---

#### Latency

Latency is defined as the time required to complete a task or produce a result. Latency is also referred to as response time. The time it takes for a web browser to download Instagram images from the internet is the latency for downloading the images.

---

#### Throughput vs Latency

In the context of concurrency, throughput can be thought of as time taken to execute a program or computation.

- For instance, imagine a program that is given hundreds of files containing integers and asked to sum up all the numbers. Since addition is commutative each file can be worked on in parallel. In a single-threaded environment, each file will be sequentially processed but in a concurrent system, several threads can work in parallel on distinct files. Of course, there will be some overhead to manage the state including already processed files. However, such a program will complete the task much faster than a single thread. The performance difference will become more and more apparent as the number of input files increases.
- The throughput in this example can be defined as the number of files processed by the program in a minute.
- Latency can be defined as the total time taken to completely process all the files.
- As you observe in a multithreaded implementation throughput will go up and latency will go down. More work gets done in less amount of time.
- In general, the two have an inverse relationship.

---

### Critical Sections & Race Conditions

<p>This section exhibits how incorrect synchronization in a critical section can lead to race conditions and buggy code.</p>

#### Critical Section

- Critical section is any piece of code that has the possibility of being executed concurrently by more than one thread of the application and exposes any shared data or resources used by the application for access.

---

#### Race Condition

- Race conditions happen when threads run through critical sections without thread synchronization.
- In a race condition, threads access shared resources or program variables that might be worked on by other threads at the same time causing the application data to be inconsistent.
- Image goes hereeee

---

### Deadlocks, Liveness & Reentrant Locks

- Logical follies committed in multithreaded code, while trying to avoid race conditions and guarding critical sections, can lead to a host of subtle and hard to find bugs and side-effects. Some of these incorrect usage patterns have their names and are discussed below.

---

#### Deadlock

- Deadlocks occur when two or more threads aren't able to make any progress because the resource required by the first thread is held by the second and the resource required by the second thread is held by the first.

- Example:

```java
void increment(){

  acquire MUTEX_A
  acquire MUTEX_B
    // do work here
  release MUTEX_B
  release MUTEX_A

}


void decrement(){

  acquire MUTEX_B
  acquire MUTEX_A
    // do work here
  release MUTEX_A
  release MUTEX_B

}
```

The above code can potentially result in a deadlock. Note that deadlock may not always happen, but for certain execution sequences, deadlock can occur. Consider the below execution sequence that ends up in a deadlock:

- T1 enters function increment

- T1 acquires MUTEX_A

- T1 gets context switched by the operating system

- T2 enters function decrement

- T2 acquires MUTEX_B

both threads are blocked now

---

#### Liveness

- Ability of a program or an application to execute in a timely manner is called liveness. If a program experiences a deadlock then it's not exhibiting liveness.

---

#### Live-Lock

- A live-lock occurs when two threads continuously react in response to the actions by the other thread without making any real progress.
- The best analogy is to think of two persons trying to cross each other in a hallway. John moves to the left to let Arun pass, and Arun moves to his right to let John pass. Both block each other now. John sees he's blocking Arun again and moves to his right and Arun moves to his left seeing he's blocking John. They never cross each other and keep blocking each other. This scenario is an example of a livelock.
- A process seems to be running and not deadlocked but in reality, isn't making any progress.

---

#### Starvation

- Other than a deadlock, an application thread can also experience starvation, when it never gets CPU time or access to shared resources. Other greedy threads continuously hog shared system resources not letting the starving thread make any progress.

---

#### Reentrant Lock

- Re-entrant locks allow for re-locking or re-entering of a synchronization lock.

---

### Mutex vs Semaphore

- The concept of and the difference between a mutex and a semaphore will draw befuddled expressions on most developers' faces. We discuss the differences between the two most fundamental concurrency constructs offered by almost all language frameworks. Difference between a mutex and a semaphore makes a pet interview question for senior engineering positions!

#### Mutex

- Mutex as the name hints implies mutual exclusion. A mutex is used to guard shared data such as a linked-list, an array or any primitive type.
- A mutex allows only a single thread to access a resource or critical section.
- Once a thread acquires a mutex, all other threads attempting to acquire the same mutex are blocked until the first thread releases the mutex.
- imageeeee

---

#### Semaphore

- Semaphore, on the other hand, is used for limiting access to a collection of resources.
- Think of semaphore as having a limited number of permits to give out.
- If a semaphore has given out all the permits it has, then any new thread that comes along requesting for a permit will be blocked, till an earlier thread with a permit returns it to the semaphore.
- A typical example would be a pool of database connections that can be handed out to requesting threads. Say there are ten available connections but 50 requesting threads. In such a scenario, a semaphore can only give out ten permits or connections at any given point in time.

- A semaphore with a single permit is called a binary semaphore and is often thought of as an equivalent of a mutex, which isn't completely correct.
- Semaphores can also be used for signaling among threads. This is an important distinction as it allows threads to cooperatively work towards completing a task. A mutex, on the other hand, is strictly limited to serializing access to shared state among competing threads.

---

#### When a Semaphore Masquerades as a Mutex?

<p>A semaphore can potentially act as a mutex if the permits it can give out is set to 1.</p>

- However, the most important difference between the two is that in case of a mutex the same thread must call acquire and subsequent release on the mutex whereas in case of a binary sempahore, different threads can call acquire and release on the semaphore.
- This leads us to the concept of ownership. <b>A mutex is owned by the thread acquiring it till the point the owning-thread releases it</b>, whereas for a semaphore there's no notion of ownership.

---

#### Semaphore for signaling

- Another distinction between a semaphore and a mutex is that semaphores can be used for signaling amongst threads.
- For example in case of the classical producer/consumer problem the producer thread can signal the consumer thread by incrementing the semaphore count to indicate to the consumer thread to consume the freshly produced item. A mutex in contrast only guards access to shared data among competing threads by forcing threads to serialize their access to critical sections and shared data-structures.
- 2 imagesss

---

#### Summary

- Mutex implies mutual exclusion and is used to serialize access to critical sections whereas semaphore can potentially be used as a mutex but it can also be used for cooperation and signaling amongst threads. Semaphore also solves the issue of missed signals.

- Mutex is owned by a thread, whereas a semaphore has no concept of ownership.

- Mutex if locked, must necessarily be unlocked by the same thread. A semaphore can be acted upon by different threads. This is true even if the semaphore has a permit of one

- Think of semaphore analogous to a car rental service such as Hertz. Each outlet has a certain number of cars, it can rent out to customers. It can rent several cars to several customers at the same time but if all the cars are rented out then any new customers need to be put on a waitlist till one of the rented cars is returned. In contrast, think of a mutex like a lone runway on a remote airport. Only a single jet can land or take-off from the runway at a given point in time. No other jet can use the runway simultaneously with the first aircraft.

---

## Multithreading in Java

<p>With the abstract concepts discussed, we'll now turn to the concurrency constructs offered by Java</p>

### Thread safety and Synchronized

#### Thread Safe

- A class and its public APIs are labelled as thread safe if multiple threads can consume the exposed APIs without causing race conditions or state corruption for the class.
- Note that composition of two or more thread-safe classes doesn't guarantee the resulting type to be thread-safe.

---

#### Synchronized

- Java’s most fundamental construct for thread synchronization is the `synchronized` keyword. It can be used to restrict access to critical sections one thread at a time.
- Each object in Java has an entity associated with it called the "monitor lock" or just `monitor`. Think of it as an exclusive lock. Once a thread gets hold of the monitor of an object, it has exclusive access to all the methods marked as synchronized.
- No other thread will be allowed to invoke a method on the object that is marked as synchronized and will block, till the first thread releases the monitor which is equivalent of the first thread exiting the synchronized method.

<p>Note Carefully:</p>

- For static methods, the monitor will be the class object, which is distinct from the monitor of each instance of the same class.
- If an uncaught exception occurs in a synchronized method, the monitor is still released.
- Furthermore, synchronized blocks can be re-entered.

<p>You may think of "synchronized" as the mutex portion of a monitor.</p>

```java
class Employee {

    // shared variable
    private String name;

    // method is synchronize on 'this' object
    public synchronized void setName(String name) {
        this.name = name;
    }

    // also synchronized on the same object
    public synchronized void resetName() {

        this.name = "";
    }

    // equivalent of adding synchronized in method
    // definition
    public String getName() {
        synchronized (this) {
            return this.name;
        }
    }
}
```

<p>As an example look at the employee class above. All the three methods are synchronized on the "this" object. If we created an object and three different threads attempted to execute each method of the object, only one will get access, and the other two will block. If we synchronized on a different object other than the this object, which is only possible for the `getName` method given the way we have written the code, then the 'critical sections' of the program become protected by two different locks. In that scenario, since `setName` and `resetName` would have been synchronized on the `this` object only one of the two methods could be executed concurrently. However `getName` would be synchronized independently of the other two methods and can be executed alongside one of them. The change would look like as follows:</p>

```java
class Employee {

    // shared variable
    private String name;
    private Object lock = new Object();

    // method is synchronize on 'this' object
    public synchronized void setName(String name) {
        this.name = name;
    }

    // also synchronized on the same object
    public synchronized void resetName() {

        this.name = "";
    }

    // equivalent of adding synchronized in method
    // definition
    public String getName() {
        // Using a different object to synchronize on
        synchronized (lock) {
            return this.name;
        }
    }
}
```

- All the sections of code that you guard with synchronized blocks on the same object can have at most one thread executing inside of them at any given point in time. These sections of code may belong to different methods, classes or be spread across the code base.
- <b>Note:</b> with the use of the `synchronized` keyword, Java forces you to implicitly acquire and release the monitor-lock for the object within the same method! One can't explicitly acquire and release the monitor in different methods. This has an important ramification, the same thread will acquire and release the monitor! In contrast, if we used semaphore, we could acquire/release them in different methods or by different threads.
  <br/>

Marking all the methods of a class synchronized in order to make it thread-safe may reduce throughput.

- As a naive example, consider a class with two completely independent properties accessed by getter methods. Both the getters synchronize on the same object, and while one is being invoked, the other would be blocked because of synchronization on the same object.
- The solution is to lock at a finer granularity, possibly use two different locks for each property so that both can be accessed in parallel.

---

### Wait & Notify

#### wait()

- The `wait` method is exposed on each java object. Each Java object can act as a condition variable.
- When a thread executes the `wait` method, it releases the monitor for the object and is placed in the wait queue.
- Note that the thread must be inside a `synchronized` block of code that synchronizes on the `same object` as the one on which `wait() `is being called, or in other words, the `thread must hold the monitor of the object on which it'll call wait`. If not so, an `illegalMonitor exception` is raised!

---

#### notify()

- Like the wait method, `notify()` can only be called by the thread which owns the monitor for the object on which `notify()` is being called else an illegal monitor exception is thrown.
- The notify method, will awaken one of the threads in the associated wait queue, i.e., waiting on the thread's monitor.
- However, this thread will not be scheduled for execution immediately and will compete with other active threads that are trying to synchronize on the same object. The thread which executed notify will also need to give up the object's monitor, before any one of the competing threads can acquire the monitor and proceed forward.

---

#### notifyAll()

- This method is the same as the `notify()` one except that it wakes up all the threads that are waiting on the object's monitor.

---

### Interrupting Threads

#### Interrupted Exception

- You'll often come across this exception being thrown from functions. When a thread wait()-s or sleep()-s then one way for it to give up waiting/sleeping is to be interrupted. If a thread is interrupted while waiting/sleeping, it'll wake up and immediately throw Interrupted exception.
- The thread class exposes the `interrupt()` method which can be used to interrupt a thread that is blocked in a `sleep()` or `wait()` call.
- Note that invoking the interrupt method only sets a flag that is polled periodically by sleep or wait to know the current thread has been interrupted and an interrupted exception should be thrown.

```java
class Demonstration {

    public static void main(String args[]) throws InterruptedException {
        InterruptExample.example();
    }
}

class InterruptExample {

    static public void example() throws InterruptedException {

        final Thread sleepyThread = new Thread(new Runnable() {

            public void run() {
                try {
                    System.out.println("I am too sleepy... Let me sleep for an hour.");
                    Thread.sleep(1000 * 60 * 60);
                } catch (InterruptedException ie) {
                    System.out.println("The interrupt flag is cleard : " + Thread.interrupted() + " " + Thread.currentThread().isInterrupted());
                    Thread.currentThread().interrupt();
                    System.out.println("Oh someone woke me up ! ");
                    System.out.println("The interrupt flag is set now : " + Thread.currentThread().isInterrupted() + " " + Thread.interrupted());

                }
            }
        });

        sleepyThread.start();

        System.out.println("About to wake up the sleepy thread ...");
        sleepyThread.interrupt();
        System.out.println("Woke up sleepy thread ...");

        sleepyThread.join();
    }
}
```

```
Output:
About to wake up the sleepy thread ...
Woke up sleepy thread ...
I am too sleepy... Let me sleep for an hour.
The interrupt flag is cleard : false false
Oh someone woke me up !
The interrupt flag is set now : true true
```

Take a minute to go through the output of the above program. Observe the following:

- Once the interrupted exception is thrown, the interrupt status/flag is cleared as the output of line-19 shows.

- On line-20 we again interrupt the thread and no exception is thrown. This is to emphasize that merely calling the interrupt method isn't responsible for throwing the interrupted exception. Rather the implementation should periodically check for the interrupt status and take appropriate action.

- On line 22 we print the interrupt status for the thread, which is set to true because of line 20.

- Note that there are two methods to check for the interrupt status of a thread. One is the static method Thread.`interrupted()` and the other is `Thread.currentThread().isInterrupted()`. The important difference between the two is that the static method would return the interrupt status and also clear it at the same time. On line 22 we deliberately call the object method first followed by the static method. If we reverse the ordering of the two method calls on line 22, the output for the line would be true and false, instead of true and true.

---

### Volatile

- The volatile concept is specific to Java.
- If a variable is declared `volatile` then whenever a thread writes or reads to the volatile variable, the read and write always happen in the main memory.
- As a further guarantee, all the variables that are visible to the writing thread also get written-out to the main memory alongside the volatile variable. Similarly, all the variables visible to the reading thread alongside the volatile variable will have the latest values visible to the reading thread.

---

#### When is `volatile` thread safe?

- Volatile comes into play because of multiples levels of memory in hardware architecture required for performance enhancements. If there's a single thread that writes to the volatile variable and other threads only read the volatile variable then just using volatile is enough, however, if there's a possibility of multiple threads writing to the volatile variable then "synchronized" would be required to ensure atomic writes to the variable.

---
