# Notes
The sleep() method is called on a Thread object and holds a lock. The wait() method is called on a lock object in synchronized block and releases the lock.

***

In some cases, all waiting threads can take useful action once the wait finishes. An example would be a set of threads waiting for a certain task to finish; once the task has finished, all waiting threads can continue with their business. In such a case you would use notifyAll() to wake up all waiting threads at the same time. Another case, for example mutually exclusive locking, only one of the waiting threads can do something useful after being notified (in this case acquire the lock). In such a case, you would rather use notify(). Properly implemented, you could use notifyAll() in this situation as well, but you would unnecessarily wake threads that can't do anything anyway.

***

ConcurrentHashMap uses multiple buckets to store data (only needed bucked is locked and not the whole object). This avoids read locks and greatly improves performance over a HashTable. Both are thread safe, but there are obvious performance wins with ConcurrentHashMap. When you read from a ConcurrentHashMap using get(), there are no locks, contrary to the HashTable for which all operations are simply synchronized. In Collections.synchronizedMap(map) each method is synchronized using an object level lock. So the get and put methods on synchMap acquire a lock.

***

AtomicInteger uses CAS mechanism. In computer science, compare-and-swap (CAS) is an atomic instruction used in multithreading to achieve synchronization. It compares the contents of a memory location with a given value and, only if they are the same, modifies the contents of that memory location to a new given value.

***

The Java volatile keyword is used to mark a Java variable as "being stored in main memory". More precisely that means, that every read of a volatile variable will be read from the computer's main memory, and not from the CPU cache, and that every write to a volatile variable will be written to main memory, and not just to the CPU cache.

***

Synchronized blocks in Java are reentrant. This means, that if a Java thread enters a synchronized block of code, and thereby take the lock on the monitor object the block is synchronized on, the thread can enter other Java code blocks synchronized on the same monitor object.

***

Benefits offered by ReentrantLock over synchronized in Java:
* Ability to lock interruptibly.
* Ability to timeout while waiting for lock.
* Power to create fair lock.
* API to get list of waiting thread for lock.
* Flexibility to try for lock without blocking.

Disadvantages of ReentrantLock in Java:
Major drawback of using ReentrantLock in Java is wrapping method body inside try-finally block, which makes code unreadable and hides business logic. Another disadvantage is that, now programmer is responsible for acquiring and releasing lock, which is a power but also opens gate for new subtle bugs, when programmer forget to release the lock in finally block.