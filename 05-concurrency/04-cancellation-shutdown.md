# Cancellation and Shutdown
* [Task Cancellation](#task-cancellation)
* [Interruption](#interruption)
* [Shutdown Hooks](#shutdown-hooks)
* [Daemon Threads](#daemon-threads)
* [Finalizers](#finalizers)
#### Task Cancellation
There is no safe way to preemptively stop a thread in Java, and therefore no safe way to preemptively stop a task. There are only cooperative mechanisms, by which the task and the code requesting cancellation follow an agreed-upon protocol. One such cooperative mechanism is setting a "cancellation requested" flag that the task checks periodically; if it finds the flag set, the task terminates early.

A task that wants to be cancellable must have a cancellation policy that specifies the "how", "when", and "what" of cancellation - how other code can request cancellation, when the task checks whether cancellation has been requested, and what actions the task takes in response to a cancellation request.

Calling interrupt does not necessarily stop the target thread from doing what it is doing; it merely delivers the message that interruption has been requested.

#### Interruption
Interruption is usually the most sensible way to implement cancellation.

Thread interruption is a cooperative mechanism for a thread to signal another thread that it should, at its convenience and if it feels like it, stop what it is doing and do something else.

A good way to think about interruption is that it does not actually interrupt a running thread; it just requests that the thread interrupt itself at the next convenient opportunity. (These opportunities are called cancellation points.) Some methods, such as **wait**, **sleep**, and **join**, take such requests seriously, throwing an exception when they receive an interrupt request or encounter an already set interrupt status upon entry.

Each thread has a boolean **interrupted** status; interrupting a thread sets its interrupted status to true. Thread contains methods for interrupting a thread and querying the interrupted status of a thread.

The **interrupt** method interrupts the target thread, and **isInterrupted** returns the interrupted status of the target thread. The poorly named static **interrupted** method clears the interrupted status of the current thread and returns its previous value; this is the only way to clear the interrupted status.

Because each thread has its own interruption policy, you should not interrupt a thread unless you know what interruption means to that thread.

Only code that implements a thread's interruption policy may swallow an interruption request. General-purpose task and library code should never swallow interruption requests.

Provide lifecycle methods whenever a thread-owning service has a lifetime longer than that of the method that created it.

When a thread exits due to an uncaught exception, the JVM reports this event to an application-provided **UncaughtExceptionHandler**; if no handler exists, the default behavior is to print the stack trace to **System.err**.

There are two practical strategies for handling InterruptedException:
* Propagate the exception (possibly after some task-specific cleanup), making your method an interruptible blocking method, too; 
* Restore the interruption status so that code higher up on the call stack can deal with it.

**ExecutorService.submit** returns a **Future** describing the task. Future has a **cancel method** that takes a boolean argument, **mayInterruptIfRunning**, and returns a value indicating whether the cancellation attempt was successful. (This tells you only whether it was able to deliver the interruption, not whether the task detected and acted on it.) When **mayInterruptIfRunning** is true and the task is currently running in some thread, then that thread is interrupted. Setting this argument to false means "don't run this task if it hasn't started yet", and should be used for tasks that are not designed to handle interruption. 

The task execution threads created by the standard **Executor** implementations implement an interruption policy that lets tasks be cancelled using interruption, so it is safe to set **mayInterruptIfRunning** when cancelling tasks through their **Futures** when they are running in a standard **Executor**. You should not interrupt a pool thread directly when attempting to cancel a task, because you won't know what task is running when the interrupt request is delivered - do this only through the task's **Future**.
```java
public static void timedRun(Runnable r, long timeout, TimeUnit unit) throws InterruptedException {
  Future<?> task = taskExec.submit(r);
  try {
    task.get(timeout, unit);
  } catch (TimeoutException e) {
    // task will be cancelled below
  } catch (ExecutionException e) {
    // exception thrown in task; rethrow
    throw launderThrowable(e.getCause());
  } finally {
    // Harmless if task already completed
    task.cancel(true); // interrupt if running
  }
}
```
#### Shutdown Hooks
In an orderly shutdown, the JVM first starts all registered shutdown hooks. Shutdown hooks are unstarted threads that are registered with **Runtime.addShutdownHook**. The JVM makes no guarantees on the order in which shutdown hooks are started. If any application threads (daemon or nondaemon) are still running at shutdown time, they continue to run concurrently with the shutdown process. When all shutdown hooks have completed, the JVM may choose to run finalizers if runFinalizersOnExit is true, and then halts. The JVM makes no attempt to stop or interrupt any application threads that are still running at shutdown time; they are abruptly terminated when the JVM eventually halts. If the shutdown hooks or finalizers don't complete, then the orderly shutdown process "hangs" and the JVM must be shut down abruptly. In an abrupt shutdown, the JVM is not required to do anything other than halt the JVM; shutdown hooks will not run.
```java
public void start() {
  Runtime.getRuntime().addShutdownHook(new Thread() {
    public void run() {
      try { LogService.this.stop(); }
      catch (InterruptedException ignored) {}
    }
  });
}
```
#### Daemon Threads
Sometimes you want to create a thread that performs some helper function but you don't want the existence of this thread to prevent the JVM from shutting down. This is what daemon threads are for. Threads are divided into two types: normal threads and daemon threads. When the JVM starts up, all the threads it creates (such as garbage collector and other housekeeping threads) are daemon threads, except the main thread. When a new thread is created, it inherits the daemon status of the thread that created it, so by default any threads created by the main thread are also normal threads. Normal threads and daemon threads differ only in what happens when they exit. When a thread exits, the JVM performs an inventory of running threads, and if the only threads that are left are daemon threads, it initiates an orderly shutdown. When the JVM halts, any remaining daemon threads are abandoned - finally blocks are not executed, stacks are not unwound - the JVM just exits.

#### Finalizers
The garbage collector does a good job of reclaiming memory resources when they are no longer needed, but some resources, such as file or socket handles, must be explicitly returned to the operating system when no longer needed. To assist in this, the garbage collector treats objects that have a nontrivial finalize method specially: after they are reclaimed by the collector, finalize is called so that persistent resources can be released. Since finalizers can run in a thread managed by the JVM, any state accessed by a finalizer will be accessed by more than one thread and therefore must be accessed with synchronization. Finalizers offer no guarantees on when or even if they run, and they impose a significant performance cost on objects with nontrivial finalizers. They are also extremely difficult to write correctly.[9] In most cases, the combination of finally blocks and explicit close methods does a better job of resource management than finalizers; the sole exception is when you need to manage objects that hold resources acquired by native methods. For these reasons and others, work hard to avoid writing or using classes with finalizers (other than the platform library classes)
