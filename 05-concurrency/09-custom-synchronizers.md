# Building Custom Synchronizers
* [Condition Queues](#condition-queues)
* [Explicit Condition Objects](#explicit-condition-objects)
* [AbstractQueuedSynchronizer](#abstractqueuedsynchronizer)
#### Condition Queues
A condition queue gets its name because it gives a group of threads - called the wait set - a way to wait for a specific condition to become true. Unlike typical queues in which the elements are data items, the elements of a condition queue are the threads waiting for the condition. Just as each Java object can act as a lock, each object can also act as a condition queue, and the **wait**, **notify**, and **notifyAll** methods in **Object** constitute the API for intrinsic condition queues. An object's intrinsic lock and its intrinsic condition queue are related: in order to call any of the condition queue methods on object X, you must hold the lock on X. This is because the mechanism for waiting for state-based conditions is necessarily tightly bound to the mechanism for preserving state consistency: you cannot wait for a condition unless you can examine the state, and you cannot release another thread from a condition wait unless you can modify the state. **Object.wait** atomically releases the lock and asks the OS to suspend the current thread, allowing other threads to acquire the lock and therefore modify the object state. Upon waking, it reacquires the lock before returning. Intuitively, calling wait means "I want to go to sleep, but wake me when something interesting happens", and calling the notification methods means "something interesting happened".
```java
@ThreadSafe
public class BoundedBuffer<V> extends BaseBoundedBuffer<V> {
// CONDITION PREDICATE: not-full (!isFull())
// CONDITION PREDICATE: not-empty (!isEmpty())
  public BoundedBuffer(int size) { super(size); }
  // BLOCKS-UNTIL: not-full
  public synchronized void put(V v) throws InterruptedException {
    while (isFull())
      wait();
    doPut(v);
    notifyAll();
  }

  // BLOCKS-UNTIL: not-empty
  public synchronized V take() throws InterruptedException {
    while (isEmpty())
      wait();
    V v = doTake();
    notifyAll();
    return v;
  }
}
```
The key to using condition queues correctly is identifying the condition predicates that the object may wait for. It is the condition predicate that causes much of the confusion surrounding wait and notify, because it has no instantiation in the API and nothing in either the language specification or the JVM implementation ensures its correct use.

There is an important three-way relationship in a condition wait involving locking, the wait method, and a condition predicate. The condition predicate involves state variables, and the state variables are guarded by a lock, so before testing the condition predicate, we must hold that lock. The lock object and the condition queue object (the object on which wait and notify are invoked) must also be the same object.

The wait method releases the lock, blocks the current thread, and waits until the specified timeout expires, the thread is interrupted, or the thread is awakened by a notification. After the thread wakes up, wait reacquires the lock before returning. A thread waking up from wait gets no special priority in reacquiring the lock; it contends for the lock just like any other thread attempting to enter a synchronized block.

When control re-enters the code calling wait, it has reacquired the lock associated with the condition queue. Is the condition predicate now true? Maybe. It might have been true at the time the notifying thread called **notifyAll**, but could have become false again by the time you reacquire the lock. Other threads may have acquired the lock and changed the object's state between when your thread was awakened and when wait reacquired the lock. Or maybe it hasn't been true at all since you called wait. You don't know why another thread called notify or notifyAll; maybe it was because another condition predicate associated with the same condition queue became true. Multiple condition predicates per condition queue are quite common.

Notification is not "sticky"if thread A notifies on a condition queue and thread B subsequently waits on that same condition queue, B does not immediately wake up another notification is required to wake B. Missed signals are the result of coding errors, such as failing to test the condition predicate before calling wait.

Whenever you wait on a condition, make sure that someone will perform a notification whenever the condition predicate becomes true.

There are two notification methods in the condition queue API **notify** and **notifyAll**. To call either, you must hold the lock associated with the condition queue object. Calling **notify** causes the JVM to select one thread waiting on that condition queue to wake up; calling **notifyAll** wakes up all the threads waiting on that condition queue. Because you must hold the lock on the condition queue object when calling **notify** or **notifyAll**, and waiting threads cannot return from wait without reacquiring the lock, the notifying thread should release the lock quickly to ensure that the waiting threads are unblocked as soon as possible. Because multiple threads could be waiting on the same condition queue for different condition predicates, using **notify** instead of **notifyAll** can be dangerous, primarily because single notification is prone to a problem akin to missed signals.

Suppose thread A waits on a condition queue for predicate PA, while thread B waits on the same condition queue for predicate PB. Now, suppose PB becomes true and thread C performs a single notify: the JVM will wake up one thread of its own choosing. If A is chosen, it will wake up, see that PA is not yet true, and go back to waiting. Meanwhile, B, which could now make progress, does not wake up. This is not exactly a missed signalit's more of a "hijacked signal"but the problem is the same: a thread is waiting for a signal that has (or should have) already occurred.

Single **notify** can be used instead of **notifyAll** only when both of the following conditions hold: 
* Uniform waiters. Only one condition predicate is associated with the condition queue, and each thread executes the 
same logic upon returning from wait;
* One-in, one-out. A notification on the condition variable enables at most one thread to proceed;

#### Explicit Condition Objects
Intrinsic condition queues have several drawbacks. Each intrinsic lock can have only one associated condition queue, which means that in classes like BoundedBuffer multiple threads might wait on the same condition queue for different condition predicates, and the most common pattern for locking involves exposing the condition queue object. Both of these factors make it impossible to enforce the uniform waiter requirement for using notifyAll. If you want to write a concurrent object with multiple condition predicates, or you want to exercise more control over the visibility of the condition queue, the explicit Lock and Condition classes offer a more flexible alternative to intrinsic locks and condition queues. 

A Condition is associated with a single Lock, just as a condition queue is associated with a single intrinsic lock; to create a Condition, call Lock.newCondition on the associated lock. And just as Lock offers a richer feature set than intrinsic locking, Condition offers a richer feature set than intrinsic condition queues: multiple wait sets per lock, interruptible and uninterruptible condition waits, deadline-based waiting, and a choice of fair or nonfair queueing.

**Hazard warning:** The equivalents of **wait**, **notify**, and **notifyAll** for **Condition** objects are **await**, **signal**, and **signalAll**. However, **Condition** extends **Object**, which means that it also has **wait** and **notify** methods. Be sure to use the proper versions - await and signal - instead!

By separating the two condition predicates into separate wait sets, Condition makes it easier to meet the requirements for single notification. Using the more efficient signal instead of signalAll reduces the number of context switches and lock acquisitions triggered by each buffer operation.

#### AbstractQueuedSynchronizer 
Most developers will probably never use AQS directly; the standard set of synchronizers covers a fairly wide range of situations. But seeing how the standard synchronizers are implemented can help clarify how they work. The basic operations that an AQS-based synchronizer performs are some variants of acquire and release. Acquisition is the state-dependent operation and can always block. With a lock or semaphore, the meaning of acquire is straightforward - acquire the lock or a permit - and the caller may have to wait until the synchronizer is in a state where that can happen. With CountDownLatch, acquire means "wait until the latch has reached its terminal state", and with FutureTask, it means "wait until the task has completed". Release is not a blocking operation; a release may allow threads blocked in acquire to proceed. For a class to be state-dependent, it must have some state. AQS takes on the task of managing some of the state for the synchronizer class: it manages a single integer of state information that can be manipulated through the protected getState, setState, and compareAndSetState methods. This can be used to represent arbitrary state; for example, ReentrantLock uses it to represent the count of times the owning thread has acquired the lock, Semaphore uses it to represent the number of permits remaining, and FutureTask uses it to represent the state of the task (not yet started, running, completed, cancelled). Synchronizers can also manage additional state variables themselves; for example, ReentrantLock keeps track of the current lock owner so it can distinguish between reentrant and contended lock-acquisition requests.