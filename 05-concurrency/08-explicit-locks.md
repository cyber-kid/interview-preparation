# Explicit Locks
* [Lock and ReentrantLock](#lock-and-reentrantlock)
* [Polled and Timed Lock Acquisition](#polled-and-timed-lock-acquisition)
* [Fairness](#fairness)
* [Read-write Locks](#read-write-locks)
#### Lock and ReentrantLock
The **Lock** interface, defines a number of abstract locking operations. Unlike intrinsic locking, **Lock** offers a choice of unconditional, polled, timed, and interruptible lock acquisition, and all lock and unlock operations are explicit. Lock implementations must provide the same memory-visibility semantics as intrinsic locks, but can differ in their locking semantics, scheduling algorithms, ordering guarantees, and performance characteristics.

**ReentrantLock** implements **Lock**, providing the same mutual exclusion and memory-visibility guarantees as synchronized. Acquiring a **ReentrantLock** has the same memory semantics as entering a synchronized block, and releasing a **ReentrantLock** has the same memory semantics as exiting a synchronized block. And, like **synchronized**, **ReentrantLock** offers reentrant locking semantics. **ReentrantLock** supports all of the lock-acquisition modes defined by Lock, providing more flexibility for dealing with lock unavailability than does synchronized.
```java
Lock lock = new ReentrantLock();
...
lock.lock();
try {
// update object state
// catch exceptions and restore invariants if necessary
} finally {
lock.unlock();
}
```

#### Polled and Timed Lock Acquisition 
The timed and polled lock-acquisition modes provided by **TryLock** allow more sophisticated error recovery than unconditional acquisition. With intrinsic locks, a deadlock is fatal - the only way to recover is to restart the application, and the only defense is to construct your program so that inconsistent lock ordering is impossible. Timed and polled locking offer another option: probabilistic deadlock avoidance. Using timed or polled lock acquisition (TryLock) lets you regain control if you cannot acquire all the required locks, release the ones you did acquire, and try again (or at least log the failure and do something else).

Timed locks are also useful in implementing activities that manage a time budget. When an activity with a time budget calls a blocking method, it can supply a timeout corresponding to the remaining time in the budget. This lets activities terminate early if they cannot deliver a result within the desired time. With intrinsic locks, there is no way to cancel a lock acquisition once it is started, so intrinsic locks put the ability to implement time-budgeted activities at risk.

#### Fairness 
The **ReentrantLock** constructor offers a choice of two fairness options: create a nonfair lock (the default) or a fair lock. Threads acquire a fair lock in the order in which they requested it, whereas a nonfair lock permits barging: threads requesting a lock can jump ahead of the queue of waiting threads if the lock happens to be available when it is requested. (Semaphore also offers the choice of fair or nonfair acquisition ordering.) Nonfair **ReentrantLocks** do not go out of their way to promote bargingthey simply don't prevent a thread from barging if it shows up at the right time. With a fair lock, a newly requesting thread is queued if the lock is held by another thread or if threads are queued waiting for the lock; with a nonfair lock, the thread is queued only if the lock is currently held.

When it comes to locking, though, fairness has a significant performance cost because of the overhead of suspending and resuming threads. In practice, a statistical fairness guarantee - promising that a blocked thread will eventually acquire the lock - is often good enough, and is far less expensive to deliver. Some algorithms rely on fair queuing to ensure their correctness, but these are unusual. In most cases, the performance benefits of non-fair locks outweigh the benefits of fair queuing.

One reason barging locks perform so much better than fair locks under heavy contention is that there can be a significant delay between when a suspended thread is resumed and when it actually runs. Let's say thread A holds a lock and thread B asks for that lock. Since the lock is busy, B is suspended. When A releases the lock, B is resumed so it can try again. In the meantime, though, if thread C requests the lock, there is a good chance that C can acquire the lock, use it, and release it before B even finishes waking up. In this case, everyone wins: B gets the lock no later than it otherwise would have, C gets it much earlier, and throughput is improved.

#### Read-write Locks
**ReadWriteLock**, exposes two Lock objects one for reading and one for writing. To read data guarded by a **ReadWriteLock** you must first acquire the read lock, and to modify data guarded by a **ReadWriteLock** you must first acquire the write lock. While there may appear to be two separate locks, the read lock and write lock are simply different views of an integrated read-write lock object.Read-write locks can improve concurrency when locks are typically held for a moderately long time and most operations do not modify the guarded resources.