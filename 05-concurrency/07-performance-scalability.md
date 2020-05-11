# Performance and Scalability
* [Performance Versus Scalability](#performance-versus-scalability)
* [Evaluating Performance Tradeoffs](#evaluating-performance-tradeoffs)
* [Amdahl's Law](#amdahls-law)
* [Context Switching](#context-switching)
* [Memory Synchronization](#memory-synchronization)
* [Blocking](#blocking)
* [Reducing Lock Contention](#reducing-lock-contention)
* [Narrowing Lock Scope](#narrowing-lock-scope-get-in-get-out)
* [Reducing Lock Granularity](#reducing-lock-granularity)
* [Lock Striping](#lock-striping)
* [Alternatives to Exclusive Locks](#alternatives-to-exclusive-locks)
#### Performance Versus Scalability 
Application performance can be measured in a number of ways, such as service time, latency, throughput, efficiency, scalability, or capacity. Some of these (service time, latency) are measures of "how fast" a given unit of work can be processed or acknowledged; others (capacity, throughput) are measures of "how much" work can be performed with a given quantity of computing resources.

Scalability describes the ability to improve throughput or capacity when additional computing resources (such as additional CPUs, memory, storage, or I/O bandwidth) are added.

#### Evaluating Performance Tradeoffs
Avoid premature optimization. First make it right, then make it fast - if it is not already fast enough.

Most performance decisions involve multiple variables and are highly situational. Before deciding that one approach is "faster" than another, ask yourself some questions: 
* What do you mean by "faster"? 
* Under what conditions will this approach actually be faster? Under light or heavy load? With large or small data sets? Can you support your answer with measurements? 
* How often are these conditions likely to arise in your situation? Can you support your answer with measurements? 
* Is this code likely to be used in other situations where the conditions may be different? 
* What hidden costs, such as increased development or maintenance risk, are you trading for this improved performance? Is this a good tradeoff?

#### Amdahl's Law
The maximum speedup converges to 1/F, meaning that a program in which fifty percent of the processing must be executed serially can be sped up only by a factor of two, regardless of how many processors are available, and a program in which ten percent must be executed serially can be sped up by at most a factor of ten. Amdahl's law also quantifies the efficiency cost of serialization. With ten processors, a program with 10% serialization can achieve at most a speedup of 5.3 (at 53% utilization), and with 100 processors it can achieve at most a speedup of 9.2 (at 9% utilization). It takes a lot of inefficiently utilized CPUs to never get to that factor of ten.

The processing time of a single task includes not only the time to execute the task **Runnable**, but also the time to dequeue the task from the shared work queue. If the work queue is a **LinkedBlockingQueue**, the dequeue operation may block less than with a synchronized **LinkedList** because **LinkedBlockingQueue** uses a more scalable algorithm, but accessing any shared data structure fundamentally introduces an element of serialization into a program. This example also ignores another common source of serialization: result handling. All useful computations produce some sort of result or side effectif not, they can be eliminated as dead code. Since **Runnable** provides for no explicit result handling, these tasks must have some sort of side effect, say writing their results to a log file or putting them in a data structure. Log files and result containers are usually shared by multiple worker threads and therefore are also a source of serialization. If instead each thread maintains its own data structure for results that are merged after all the tasks are performed, then the final merge is a source of serialization.

#### Context Switching
If there are more runnable threads than CPUs, eventually the OS will preempt one thread so that another can use the CPU. This causes a context switch, which requires saving the execution context of the currently running thread and restoring the execution context of the newly scheduled thread.

Context switches are not free; thread scheduling requires manipulating shared data structures in the OS and JVM. The OS and JVM use the same CPUs your program does; more CPU time spent in JVM and OS code means less is available for your program. But OS and JVM activity is not the only cost of context switches. When a new thread is switched in, the data it needs is unlikely to be in the local processor cache, so a context switch causes a flurry of cache misses, and thus threads run a little more slowly when they are first scheduled. This is one of the reasons that schedulers give each runnable thread a certain minimum time quantum even when many other threads are waiting: it amortizes the cost of the context switch and its consequences over more uninterrupted execution time, improving overall throughput (at some cost to responsiveness).

The vmstat command on Unix systems and the perfmon tool on Windows systems report the number of context switches and the percentage of time spent in the kernel. High kernel usage (over 10%) often indicates heavy scheduling activity, which may be caused by blocking due to I/O or lock contention.

#### Memory Synchronization 
The performance cost of synchronization comes from several sources. The visibility guarantees provided by synchronized and volatile may entail using special instructions called memory barriers that can flush or invalidate caches, flush hardware write buffers, and stall execution pipelines. Memory barriers may also have indirect performance consequences because they inhibit other compiler optimizations; most operations cannot be reordered with memory barriers.

Modern JVMs can reduce the cost of incidental synchronization by optimizing away locking that can be proven never to contend. If a lock object is accessible only to the current thread, the JVM is permitted to optimize away a lock acquisition because there is no way another thread could synchronize on the same lock.

More sophisticated JVMs can use escape analysis to identify when a local object reference is never published to the heap and is therefore thread-local.

Don't worry excessively about the cost of uncontended synchronization. The basic mechanism is already quite fast, and JVMs can perform additional optimizations that further reduce or eliminate the cost. Instead, focus optimization efforts on areas where lock contention actually occurs.

Synchronization by one thread can also affect the performance of other threads. Synchronization creates traffic on the shared memory bus; this bus has a limited bandwidth and is shared across all processors. If threads must compete for synchronization bandwidth, all threads using synchronization will suffer.

#### Blocking 
Uncontended synchronization can be handled entirely within the JVM; contended synchronization may require OS activity, which adds to the cost. When locking is contended, the losing thread(s) must block. The JVM can implement blocking either via spin-waiting (repeatedly trying to acquire the lock until it succeeds) or by suspending the blocked thread through the operating system. Which is more efficient depends on the relationship between context switch overhead and the time until the lock becomes available; spin-waiting is preferable for short waits and suspension is preferable for long waits. Some JVMs choose between the two adaptively based on profiling data of past wait times, but most just suspend threads waiting for a lock. Suspending a thread because it could not get a lock, or because it blocked on a condition wait or blocking I/O operation, entails two additional context switches and all the attendant OS and cache activity: the blocked thread is switched out before its quantum has expired, and is then switched back in later after the lock or other resource becomes available. (Blocking due to lock contention also has a cost for the thread holding the lock: when it releases the lock, it must then ask the OS to resume the blocked thread.)

#### Reducing Lock Contention 
We've seen that serialization hurts scalability and that context switches hurt performance. Contended locking causes both, so reducing lock contention can improve both performance and scalability.

The principal threat to scalability in concurrent applications is the exclusive resource lock.

Two factors influence the likelihood of contention for a lock: how often that lock is requested and how long it is held once acquired. If the product of these factors is sufficiently small, then most attempts to acquire the lock will be uncontended, and lock contention will not pose a significant scalability impediment. If, however, the lock is in sufficiently high demand, threads will block waiting for it; in the extreme case, processors will sit idle even though there is plenty of work to do.

There are three ways to reduce lock contention: 
* Reduce the duration for which locks are held; 
* Reduce the frequency with which locks are requested;
* Replace exclusive locks with coordination mechanisms that permit greater concurrency;

#### Narrowing Lock Scope ("Get in, Get Out") 
An effective way to reduce the likelihood of contention is to hold locks as briefly as possible. This can be done by moving code that doesn't require the lock out of synchronized blocks, especially for expensive operations and potentially blocking operations such as I/O.

While shrinking synchronized blocks can improve scalability, a synchronized block can be too small operations that need to be atomic (such updating multiple variables that participate in an invariant) must be contained in a single synchronized block. And because the cost of synchronization is nonzero, breaking one synchronized block into multiple synchronized blocks (correctness permitting) at some point becomes counterproductive in terms of performance. The ideal balance is of course platform-dependent, but in practice it makes sense to worry about the size of a synchronized block only when you can move "substantial" computation or blocking operations out of it.

#### Reducing Lock Granularity 
The other way to reduce the fraction of time that a lock is held (and therefore the likelihood that it will be contended) is to have threads ask for it less often. This can be accomplished by lock splitting and lock striping, which involve using separate locks to guard multiple independent state variables previously guarded by a single lock. These techniques reduce the granularity at which locking occurs, potentially allowing greater scalabilitybut using more locks also increases the risk of deadlock.

#### Lock Striping
Lock splitting can sometimes be extended to partition locking on a variablesized set of independent objects, in which case it is called lock striping. For example, the implementation of ConcurrentHashMap uses an array of 16 locks, each of which guards 1/16 of the hash buckets; bucket N is guarded by lock N mod 16. Assuming the hash function provides reasonable spreading characteristics and keys are accessed uniformly, this should reduce the demand for any given lock by approximately a factor of 16. It is this technique that enables ConcurrentHashMap to support up to 16 concurrent writers.

One of the downsides of lock striping is that locking the collection for exclusive access is more difficult and costly than with a single lock. Usually an operation can be performed by acquiring at most one lock, but occasionally you need to lock the entire collection, as when ConcurrentHashMap needs to expand the map and rehash the values into a larger set of buckets. This is typically done by acquiring all of the locks in the stripe set.

#### Alternatives to Exclusive Locks 
A third technique for mitigating the effect of lock contention is to forego the use of exclusive locks in favor of a more concurrency-friendly means of managing shared state. These include using the concurrent collections, read-write locks, immutable objects and atomic variables.