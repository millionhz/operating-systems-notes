# Concurrency

- [Threads and Concurrency](https://applied-programming.github.io/Operating-Systems-Notes/3-Threads-and-Concurrency/)

## Multi-Threaded Program

- Program with more than one point of execution → Multiple Program Counters
- Threaded programs share the same memory space.

![Untitled](./img/Untitled%2070.png)

![Untitled](./img/Untitled%2071.png)

### Benefits of Threads

- Parallelization
    * Process data in parallel.
    * If a computer program or system is parallelized, it breaks a problem down into smaller pieces to be independently solved simultaneously by discrete computing resources.
- Specialization
    * Threads will get the data they need from the cache. (As other threads performing some similar workload might have put it there)
- Time for context switch in threads is less, since memory is shared, hence mapping is not required between virtual and physical memory.

### Are threads useful on a single CPU?

- Yes, If the time for switching to Thread2 and back is less than the time, Thread1 stays idle (in an IO)

### Are threads useful when #threads > #CPUs?

- Yes, because cost of thread switching is less than the cost of process switching, as we threads share address space and thus no need to remap address space.

## Multithreading Models

- User Threads: Supported above the kernel and are managed without any kernel support. (dummy threads) (cheaper to make)
- Kernel Threads: Supported and managed directly by the Operating System. (true threads) (expensive to make)

### Many to One

- Many user level threads mapped to one kernel thread.
- Pros:
    * Portable
    * Not effected by OS limits on kernel threads and other policies.
- Cons:
    * True parallelism is not possible as one kernel thread will process one user thread at a time. (even on multicore system)
    * If a user thread makes a blocking call, the whole process blocks.

### One to One

- Every user thread is mapped to a unique kernel thread.
- Pros:
    * Does not block the process if one thread makes a blocking call.
    * Parallelism is possible.
- Cons:
    * Spawning a kernel thread for every user thread is expensive.
    * Dependence on the OS for all operations
    * Affected by OS limitation on kernel threads and policies.

### Many to Many

- Many user threads are mapped to **smaller** or **equal number** of kernel threads.
- Best of both worlds

## Multithreading Patterns

- How to structure multithreaded applications?

### Boss-Worker

- A Single Boss assigns work.
- Many workers perform the work assigned by the boss.
- Throughput of the system is dependent on the boss.
- Boss and workers communicate by
    * directly signaling individual workers
    * using a producer-consumer queue
- Pros:
    * Simplicity
- Cons:
    * thread pool needs to be managed.
    * process-consumer queue needs to be managed
    * Boss doesnt have notion of locality; a worker does better if its assigned similar tasks (cache), but for that boss will need to know the state of the worker, which is not supported by boss-worker model.

#### Variants

- Workers can be specialized for certain tasks.
- Boss will now need to know what task to assign to what worker.
- Pros:
    * Better locality
- Cons:
    * Load Balancing

### Pipeline

- Divide the main task into subtasks.
- Assign each worker a specific subtask.
- Throughput of the system depends on the subtask that takes the longest (throughput can be managed by increasing or decreasing workers per subtasks)
- One worker will depend on the other.
- Pros:
    * Specialization
    * Locality
- Cons:
    * Balancing
    * Synchronization overheads

### Layered

- CHECK SLIDES

## Locks

- Keep code atomic.
- Keep critical areas safe.
- CHECK SLIDES + Book

## Semaphores

- PENDING

## Multiprocessors

- Computer architects have had a difficult time making a single CPU much faster without using (way) too much power
- Multiple processes can be scheduled on different CPUs
- Multithreaded applications can spread work across multiple CPUs and thus run faster

### Caches vs Memory

![Untitled](./img/Untitled%2094.png)

- The difference between the two is mainly around the use of hardware caches and how data is shared between the two.

## Caches and Multiple CPUs

- In multi CPU architectures, caches can be shared.
- If there are three levels of caches, L0 and L1 will be dedicated to the CPU but L2 will be shared.

### Cache Coherence

- Program running on one CPU can read the cache and write the cache the data, but as the CPU buffers the writes to main memory, if the program gets moved to another CPU with a different cache. It may not be able to access the updated data that is still sitting in the cache of the old CPU waiting to be written.
- Similarly if a program is running on the 2 CPUs with a certain variable if cached in the both CPUs, then if a CPU writes and mutates the value of the variable, the cache of the old CPU will not be updated and will cause issues.
- Generally, its is difficult to ensure that the data in caches is coherent across CPUs.

### Hardware - Cache Coherence

#### Write Invalidate (write-through)

- If one CPUs tries to modify a variable in the shared memory, hardware controller will invalidate the caches of the other CPUs.
- Subsequent reads will be issue to the Main memory.
- This **only solves the problem if the caches being used are write-through** (i.e. values are updated in both the main memory and the cache)
- + Lower Bandwidth - if you write to one cache, the only thing that happens is invalidation of other caches.

#### Write Update (write-back)

- If a cache is modified, all other caches are also modified with the updated value.
- - Large amount of traffic: needs to update all pages.
- + The updated variable is immediately available.

### Atomic Instructions

- Atomic instruction have to deal with concurrent execution in case of multiple CPUs.
- Atomic instructions are handed by a memory controller
    * which maintains the order and synchronization of the instructions.
    * no 2 atomic instruction will get issued to the main memory directly.
- - Takes Longer
- - Generated coherence traffic

## NUMA Aware Scheduling

- In a multiprocessor architecture, there are multiple CPUs with multiple memory chips.
- The CPU closest to the memory containing the data will perform the fastest.
- When scheduling, ensure that the cpu closest to the memory with the data is scheduled to run.
- Non Uniform Memory Access NUMA aware scheduling - algorithms that take in account the location of the memory with respect to the CPU.

## Cache Affinity & Scheduling

- Schedule threads onto the same CPU as much as possible to take advantage of hot caches.

## *Hyperthreading*

- Multithreading on a single CPU
- Keep an array of registers with data of processes (only on of them is active)
- Context Switching is faster
- Challenge: What kind of threads should be co-scheduled:
    * Mix of I/O and CPU bound processes
- Hyperthreading also reduces memory access latency by employing hot caches.

## Scheduling Algorithms

### Single Queue Multiprocessor - SQMS

- Put all jobs that need to be scheduled into a single queue.

#### Advantage

- Simplicity

#### Disadvantage

- Locks on a single queue slows down execution
- Cache affinity is violated. Jobs end up bouncing around from CPU to CPU
- Some threads can be bound to specific CPUs and the rest of the threads can be allowed to bounce around.

![Untitled](./img/Untitled%2095.png)

### Multi-Queue Multiprocessor Scheduling - MQMS

- Multiple scheduling queues, one per processor/core
- When a job enters the system, it is placed on exactly one scheduling queue random, or picking one with fewer jobs than others
    * The job stays bound to the queue throughout its lifecycle

#### Advantages

- Cache Affinity

#### Disadvantages

![Untitled](./img/Untitled%2096.png)

#### Dealing with Load Imbalance

![Untitled](./img/Untitled%2097.png)

![Untitled](./img/Untitled%2098.png)

#### **Work stealing**

- A (source) queue that is low on jobs will occasionally peek at another (target) queue
- If the target queue is fuller than the source queue, the source will “steal” one or more jobs from the target to help balance load
- If you look around at other queues too often, you will suffer from high overhead and have trouble scaling
- If you don’t look at other queues very often, you are in danger of suffering from severe load imbalances
- Finding the right threshold → a voodoo parameter
