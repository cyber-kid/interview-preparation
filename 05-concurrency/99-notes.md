# Notes
The **sleep()** method is called on a **Thread** object and holds a lock. The **wait()** method is called on a lock object in synchronized block and releases the lock.

***

In some cases, all waiting threads can take useful action once the wait finishes. An example would be a set of threads waiting for a certain task to finish; once the task has finished, all waiting threads can continue with their business. In such a case you would use **notifyAll()** to wake up all waiting threads at the same time. Another case, for example mutually exclusive locking, only one of the waiting threads can do something useful after being notified (in this case acquire the lock). In such a case, you would rather use **notify()**. Properly implemented, you could use **notifyAll()** in this situation as well, but you would unnecessarily wake threads that can't do anything anyway.

***

**ConcurrentHashMap** uses multiple buckets to store data (only needed bucked is locked and not the whole object). This avoids read locks and greatly improves performance over a **HashTable**. Both are thread safe, but there are obvious performance wins with **ConcurrentHashMap**. When you read from a **ConcurrentHashMap** using **get()**, there are no locks, contrary to the **HashTable** for which all operations are simply synchronized. In **Collections.synchronizedMap(map)** each method is synchronized using an object level lock. So the **get** and **put** methods on **synchMap** acquire a lock.

***

The Java **volatile** keyword is used to mark a Java variable as "being stored in main memory". More precisely that means, that every read of a **volatile** variable will be read from the computer's main memory, and not from the CPU cache, and that every write to a **volatile** variable will be written to main memory, and not just to the CPU cache.

***

Synchronized blocks in Java are reentrant. This means, that if a Java thread enters a synchronized block of code, and thereby take the lock on the monitor object the block is synchronized on, the thread can enter other Java code blocks synchronized on the same monitor object.

***

Benefits offered by **ReentrantLock** over synchronized in Java:
* Ability to lock interruptibly.
* Ability to timeout while waiting for lock.
* Power to create fair lock.
* API to get list of waiting thread for lock.
* Flexibility to try for lock without blocking.

Disadvantages of **ReentrantLock** in Java:
Major drawback of using **ReentrantLock** in Java is wrapping method body inside try-finally block, which makes code unreadable and hides business logic. Another disadvantage is that, now programmer is responsible for acquiring and releasing lock, which is a power but also opens gate for new subtle bugs, when programmer forget to release the lock in finally block.

***

**Compare and Swap** CAS has three operands - a memory location V on which to operate, the expected old value A, and the new value B. CAS atomically updates V to the new value B, but only if the value in V matches the expected old value A; otherwise it does nothing. In either case, it returns the value currently in V. (The variant called compare-and-set instead returns whether the operation succeeded.) CAS means "I think V should have the value A; if it does, put B there, otherwise don't change it but tell me I was wrong." CAS is an optimistic technique - it proceeds with the update in the hope of success, and can detect failure if another thread has updated the variable since it was last examined.

***

**Non-blocking Algorithms** Lock-based algorithms are at risk for a number of liveness failures. If a thread holding a lock is delayed due to blocking I/O, page fault, or other delay, it is possible that no thread will make progress. An algorithm is called non-blocking if failure or suspension of any thread cannot cause failure or suspension of another thread; an algorithm is called lock-free if, at each step, some thread can make progress. Algorithms that use CAS exclusively for coordination between threads can, if constructed correctly, be both non-blocking and lock-free. An uncontended CAS always succeeds, and if multiple threads contend for a CAS, one always wins and therefore makes progress. Non-blocking algorithms are also immune to deadlock or priority inversion (though they can exhibit starvation or livelock because they can involve repeated retries).
