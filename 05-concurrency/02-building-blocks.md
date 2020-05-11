# Building blocks of multithreaded application
* [Synchronized Collections](#synchronized-collections)
* [Iterators and Concurrentmodificationexception](#iterators-and-concurrentmodificationexception)
* [Hidden Iterators](#hidden-iterators)
* [Concurrent Collections](#concurrent-collections)
* [ConcurrentHashMap](#concurrenthashmap)
* [CopyOnWriteArrayList](#copyonwritearraylist)
* [Blocking Queues and the Producer-consumer Pattern](#blocking-queues-and-the-producer-consumer-pattern)
* [Latches](#latches)
* [FutureTask](#futuretask)
* [Semaphores](#semaphores)
* [Barriers](#barriers)
#### Synchronized Collections
The synchronized collection classes include **Vector** and **Hashtable**, part of the original JDK, as well as their cousins added in JDK 1.2, the synchronized wrapper classes created by the **Collections.synchronizedXxx** factory methods. These classes achieve thread safety by encapsulating their state and synchronizing every public method so that only one thread at a time can access the collection state.
#### Iterators and Concurrentmodificationexception
The iterators returned by the synchronized collections are not designed to deal with concurrent modification, and they are fail-fast - meaning that if they detect that the collection has changed since iteration began, they throw the unchecked **ConcurrentModificationException**. These fail-fast iterators are not designed to be foolproof - they are designed to catch concurrency errors on a "good-faith-effort" basis and thus act only as early-warning indicators for concurrency problems. They are implemented by associating a modification count with the collection: if the modification count changes during iteration, **hasNext** or **next** throws **ConcurrentModificationException**. However, this check is done without synchronization, so there is a risk of seeing a stale value of the modification count and therefore that the iterator does not realize a modification has been made. This was a deliberate design tradeoff to reduce the performance impact of the concurrent modification detection code.
#### Hidden Iterators
Iteration is also indirectly invoked by the collection's **hashCode** and **equals** methods, which may be called if the collection is used as an element or key of another collection. Similarly, the **containsAll**, **removeAll**, and **retainAll** methods, as well as the constructors that take collections are arguments, also iterate the collection. All of these indirect uses of iteration can cause **ConcurrentModificationException**.
#### Concurrent Collections
The concurrent collections, on the other hand, are designed for concurrent access from multiple threads. Java 5.0 adds **ConcurrentHashMap**, a replacement for synchronized hash-based **Map** implementations, and **CopyOnWriteArrayList**, a replacement for synchronized **List** implementations for cases where traversal is the dominant operation. The new **ConcurrentMap** interface adds support for common compound actions such as put-if-absent, replace, and conditional remove.

Just as **ConcurrentHashMap** is a concurrent replacement for a synchronized hash-based **Map**, Java 6 adds **ConcurrentSkipListMap** and **ConcurrentSkipListSet**, which are concurrent replacements for a synchronized **SortedMap** or **SortedSet** (such as **TreeMap** or **TreeSet** wrapped with **synchronizedMap**).
#### ConcurrentHashMap
**ConcurrentHashMap** is a hash-based **Map** like **HashMap**, but it uses an entirely different locking strategy that offers better concurrency and scalability. Instead of synchronizing every method on a common lock, restricting access to a single thread at a time, it uses a finer-grained locking mechanism called lock striping to allow a greater degree of shared access. Arbitrarily many reading threads can access the map concurrently, readers can access the map concurrently with writers, and a limited number of writers can modify the map concurrently. The result is far higher throughput under concurrent access, with little performance penalty for single-threaded access. **ConcurrentHashMap**, along with the other concurrent collections, further improve on the synchronized collection classes by providing iterators that do not throw **ConcurrentModificationException**, thus eliminating the need to lock the collection during iteration. The iterators returned by **ConcurrentHashMap** are weakly consistent instead of fail-fast. A weakly consistent iterator can tolerate concurrent modification, traverses elements as they existed when the iterator was constructed, and may (but is not guaranteed to) reflect modifications to the collection after the construction of the iterator.
#### CopyOnWriteArrayList
The copy-on-write collections derive their thread safety from the fact that as long as an effectively immutable object is properly published, no further synchronization is required when accessing it. They implement mutability by creating and republishing a new copy of the collection every time it is modified. Iterators for the copy-on-write collections retain a reference to the backing array that was current at the start of iteration, and since this will never change, they need to synchronize only briefly to ensure visibility of the array contents.
#### Blocking Queues and the Producer-consumer Pattern
**BlockingQueue** extends **Queue** to add blocking insertion and retrieval operations. If the queue is empty, a retrieval blocks until an element is available, and if the queue is full (for bounded queues) an insertion blocks until there is space available. Blocking queues are extremely useful in producer-consumer designs.

The class library contains several implementations of **BlockingQueue**. **LinkedBlockingQueue** and **ArrayBlockingQueue are FIFO queues, analogous to **LinkedList** and **ArrayList** but with better concurrent performance than a synchronized **List**. **PriorityBlockingQueue** is a priority-ordered queue, which is useful when you want to process elements in an order other than FIFO. Just like other sorted collections, PriorityBlockingQueue can compare elements according to their natural order (if they implement Comparable) or using a Comparator.
#### Latches
A latch is a synchronizer that can delay the progress of threads until it reaches its terminal state. A latch acts as a gate: until the latch reaches the terminal state the gate is closed and no thread can pass, and in the terminal state the gate opens, allowing all threads to pass. Once the latch reaches the terminal state, it cannot change state again, so it remains open forever.

**CountDownLatch** is a flexible latch implementation that can be used in any of these situations; it allows one or more threads to wait for a set of events to occur. The latch state consists of a counter initialized to a positive number, representing the number of events to wait for. The **countDow**n method decrements the counter, indicating that an event has occurred, and the **await** methods wait for the counter to reach zero, which happens when all the events have occurred. If the counter is nonzero on entry, **await** blocks until the counter reaches zero, the waiting thread is interrupted, or the wait times out.
```java
public class TestHarness {
  public long timeTasks(int nThreads, final Runnable task) throws InterruptedException {
    final CountDownLatch startGate = new CountDownLatch(1);
    final CountDownLatch endGate = new CountDownLatch(nThreads);

    for (int i = 0; i < nThreads; i++) {
      Thread t = new Thread() {
        public void run() {
          try {
            startGate.await();
              try {
                task.run();
              } finally {
                endGate.countDown();
              }
          } catch (InterruptedException ignored) { }
        }
      };
      t.start();
    }

    long start = System.nanoTime();
    startGate.countDown();
    endGate.await();
    long end = System.nanoTime();
    return end-start;
  }
}
```
#### FutureTask
**FutureTask** also acts like a latch. (**FutureTask** implements **Future**, which describes an abstract result-bearing computation.) A computation represented by a **FutureTask** is implemented with a **Callable**, the result-bearing equivalent of **Runnable**, and can be in one of three states: waiting to run, running, or completed. Completion subsumes all the ways a computation can complete, including normal completion, cancellation, and exception. Once a **FutureTask** enters the completed state, it stays in that state forever. The behavior of **Future.get** depends on the state of the task. If it is completed, get returns the result immediately, and otherwise blocks until the task transitions to the completed state and then returns the result or throws an exception. **FutureTask** conveys the result from the thread executing the computation to the thread(s) retrieving the result; the specification of **FutureTask** guarantees that this transfer constitutes a safe publication of the result.
#### Semaphores
Counting semaphores are used to control the number of activities that can access a certain resource or perform a given action at the same time. Counting semaphores can be used to implement resource pools or to impose a bound on a collection. A **Semaphore** manages a set of virtual permits; the initial number of permits is passed to the **Semaphore** constructor. Activities can acquire permits (as long as some remain) and release permits when they are done with them. If no permit is available, acquire blocks until one is (or until interrupted or the operation times out). The release method returns a permit to the semaphore. A degenerate case of a counting semaphore is a binary semaphore, a **Semaphore** with an initial count of one. A binary semaphore can be used as a mutex with non-reentrant locking semantics; whoever holds the sole permit holds the mutex.
#### Barriers
**Barriers** are similar to latches in that they block a group of threads until some event has occurred. The key difference is that with a barrier, all the threads must come together at a barrier point at the same time in order to proceed. **Latches** are for waiting for events; barriers are for waiting for other threads.

**CyclicBarrier** allows a fixed number of parties to rendezvous repeatedly at a barrier point and is useful in parallel iterative algorithms that break down a problem into a fixed number of independent subproblems. Threads call **await** when they reach the barrier point, and **await** blocks until all the threads have reached the barrier point. If all threads meet at the barrier point, the barrier has been successfully passed, in which case all threads are released and the barrier is reset so it can be used again. If a call to **await** times out or a thread blocked in await is interrupted, then the barrier is considered broken and all outstanding calls to **await** terminate with **BrokenBarrierException**. If the barrier is successfully passed, **await** returns a unique arrival index for each thread, which can be used to "elect" a leader that takes some special action in the next iteration. **CyclicBarrier** also lets you pass a barrier action to the constructor; this is a **Runnable** that is executed (in one of the subtask threads) when the barrier is successfully passed but before the blocked threads are released.
