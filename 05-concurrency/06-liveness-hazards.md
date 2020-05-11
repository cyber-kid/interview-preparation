# Avoiding Liveness Hazards
* [Deadlock](#deadlock)
* [Lock-ordering Deadlocks](#lock-ordering-deadlocks)
* [Dynamic Lock Order Deadlocks](#dynamic-lock-order-deadlocks)
* [Open Calls](#open-calls)
* [Avoiding and Diagnosing Deadlocks](#avoiding-and-diagnosing-deadlocks)
* [Deadlock Analysis with Thread Dump](#deadlock-analysis-with-thread-dumps)
* [Other Liveness Hazards](#other-liveness-hazards)
#### Deadlock
When a thread holds a lock forever, other threads attempting to acquire that lock will block forever waiting. When thread A holds lock L and tries to acquire lock M, but at the same time thread B holds M and tries to acquire L, both threads will wait forever. This situation is the simplest case of deadlock (or deadly embrace), where multiple threads wait forever due to a cyclic locking dependency.

#### Lock-ordering Deadlocks
The deadlock came about because the two threads attempted to acquire the same locks in a different order. If they asked for the locks in the same order, there would be no cyclic locking dependency and therefore no deadlock. If you can guarantee that every thread that needs locks L and M at the same time always acquires L and M in the same order, there will be no deadlock.

A program will be free of lock-ordering deadlocks if all threads acquire the locks they need in a fixed global order.

#### Dynamic Lock Order Deadlocks
Since the order of arguments is out of our control, to fix the problem we must induce an ordering on the locks and acquire them according to the induced ordering consistently throughout the application.

One way to induce an ordering on objects is to use **System.identityHashCode**, which returns the value that would be returned by **Object.hashCode**.

In the rare case that two objects have the same hash code, we must use an arbitrary means of ordering the lock acquisitions, and this reintroduces the possibility of deadlock. To prevent inconsistent lock ordering in this case, a third "tie breaking" lock is used.

If Account has a unique, immutable, comparable key such as an account number, inducing a lock ordering is even easier: order objects by their key, thus eliminating the need for the tie-breaking lock.
```java
private static final Object tieLock = new Object();

public void transferMoney(final Account fromAcct, final Account toAcct, final DollarAmount amount) throws InsufficientFundsException {
  class Helper {
    public void transfer() throws InsufficientFundsException {
      if (fromAcct.getBalance().compareTo(amount) < 0)
        throw new InsufficientFundsException();
      else {
        fromAcct.debit(amount);
        toAcct.credit(amount);
      }
    }
  }

  int fromHash = System.identityHashCode(fromAcct);
  int toHash = System.identityHashCode(toAcct);

  if (fromHash < toHash) {
    synchronized (fromAcct) {
      synchronized (toAcct) {
        new Helper().transfer();
      }
    }
  } else if (fromHash > toHash) {
    synchronized (toAcct) {
      synchronized (fromAcct) {
        new Helper().transfer();
      }
    }
  } else {
    synchronized (tieLock) {
      synchronized (fromAcct) {
        synchronized (toAcct) {
          new Helper().transfer();
        }
      }
    }
  }
}
```

#### Open Calls
But because you don't know what is happening on the other side of the call, calling an alien method with a lock held is difficult to analyze and therefore risky. Calling a method with no locks held is called an open call, and classes that rely on open calls are more well-behaved and composable than classes that make calls with locks held.

This involves shrinking the synchronized blocks to guard only operations that involve shared state. Very often, the cause of problems is the use of synchronized methods instead of smaller synchronized blocks for reasons of compact syntax or simplicity rather than because the entire method must be guarded by a lock.

Restructuring a synchronized block to allow open calls can sometimes have undesirable consequences, since it takes an operation that was atomic and makes it not atomic. In many cases, the loss of atomicity is perfectly acceptable.

#### Avoiding and Diagnosing Deadlocks
A program that never acquires more than one lock at a time cannot experience lock-ordering deadlock. Of course, this is not always practical, but if you can get away with it, it's a lot less work. If you must acquire multiple locks, lock ordering must be a part of your design: try to minimize the number of potential locking interactions, and follow and document a lock-ordering protocol for locks that may be acquired together.

In programs that use fine-grained locking, audit your code for deadlock freedom using a two-part strategy: first, identify where multiple locks could be acquired (try to make this a small set), and then perform a global analysis of all such instances to ensure that lock ordering is consistent across your entire program. Using open calls wherever possible simplifies this analysis substantially. With no non-open calls, finding instances where multiple locks are acquired is fairly easy, either by code review or by automated bytecode or source code analysis.

Another technique for detecting and recovering from deadlocks is to use the timed **tryLock** feature of the explicit **Lock** classes instead of intrinsic locking. Where intrinsic locks wait forever if they cannot acquire the lock, explicit locks let you specify a timeout after which **tryLock** returns failure. By using a timeout that is much longer than you expect acquiring the lock to take, you can regain control when something unexpected happens.

#### Deadlock Analysis with Thread Dumps 
While preventing deadlocks is mostly your problem, the JVM can help identify them when they do happen using thread dumps. A thread dump includes a stack trace for each running thread, similar to the stack trace that accompanies an exception. Thread dumps also include locking information, such as which locks are held by each thread, in which stack frame they were acquired, and which lock a blocked thread is waiting to acquire. Before generating a thread dump, the JVM searches the is-waiting-for graph for cycles to find deadlocks. If it finds one, it includes deadlock information identifying which locks and threads are involved, and where in the program the offending lock acquisitions are.

#### Other Liveness Hazards
Avoid the temptation to use thread priorities, since they increase platform dependence and can cause liveness problems. Most concurrent applications can use the default priority for all threads.

Retrying with random waits and back-offs can be equally effective for avoiding livelock in concurrent applications.
