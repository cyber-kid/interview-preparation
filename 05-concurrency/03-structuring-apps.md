# Structuring Concurrent Applications
* [Disadvantages of Unbounded Thread Creation](#disadvantages-of-unbounded-thread-creation)
* [Execution Policies](#execution-policies)
* [Thread Pools](#thread-pools)
* [Executor Lifecycle](#executor-lifecycle)
#### Disadvantages of Unbounded Thread Creation
For production use, however, the thread-per-task approach has some practical drawbacks, especially when a large number of threads may be created:
* **Thread lifecycle overhead** Thread creation and teardown are not free. The actual overhead varies across platforms, but thread creation takes time, introducing latency into request processing, and requires some processing activity by the
* **JVM and OS** If requests are frequent and lightweight, as in most server applications, creating a new thread for each request can consume significant computing resources.
* **Resource consumption** Active threads consume system resources, especially memory. When there are more runnable threads than available processors, threads sit idle. Having many idle threads can tie up a lot of memory, putting pressure on the garbage collector, and having many threads competing for the CPUs can impose other performance costs as well. If you have enough threads to keep all the CPUs busy, creating more threads won't help and may even hurt.
* **Stability** There is a limit on how many threads can be created. The limit varies by platform and is affected by factors including JVM invocation parameters, the requested stack size in the Thread constructor, and limits on threads placed by the underlying operating system. When you hit this limit, the most likely result is an OutOfMemoryError. trying to recover from such an error is very risky; it is far easier to structure your program to avoid hitting this limit.

The primary abstraction for task execution in the Java class libraries is not **Thread**, but **Executor**.
#### Execution Policies
The value of decoupling submission from execution is that it lets you easily specify, and subsequently change without great difficulty, the execution policy for a given class of tasks. An execution policy specifies the "what, where, when, and how" of task execution, including:
* In what thread will tasks be executed?
* In what order should tasks be executed (FIFO, LIFO, priority order)?
* How many tasks may execute concurrently?
* How many tasks may be queued pending execution?
* If a task has to be rejected because the system is overloaded, which task should be selected as the victim, and how should the application be notified?
* What actions should be taken before or after executing a task?
#### Thread Pools
A thread pool, as its name suggests, manages a homogeneous pool of worker threads. A thread pool is tightly bound to a work queue holding tasks waiting to be executed. Worker threads have a simple life: request the next task from the work queue, execute it, and go back to waiting for another task.

Executing tasks in pool threads has a number of advantages over the thread-per-task approach. Reusing an existing thread instead of creating a new one amortizes thread creation and teardown costs over multiple requests. As an added bonus, since the worker thread often already exists at the time the request arrives, the latency associated with thread creation does not delay task execution, thus improving responsiveness. By properly tuning the size of the thread pool, you can have enough threads to keep the processors busy while not having so many that your application runs out of memory or thrashes due to competition among threads for resources. The class library provides a flexible thread pool implementation along with some useful predefined configurations. You can create a thread pool by calling one of the static factory methods in Executors:
* **newFixedThreadPool** A fixed-size thread pool creates threads as tasks are submitted, up to the maximum pool size, and then attempts to keep the pool size constant (adding new threads if a thread dies due to an unexpected Exception).
* **newCachedThreadPool** A cached thread pool has more flexibility to reap idle threads when the current size of the pool exceeds the demand for processing, and to add new threads when demand increases, but places no bounds on the size of the pool.
* **newSingleThreadExecutor** A single-threaded executor creates a single worker thread to process tasks, replacing it if it dies unexpectedly. Tasks are guaranteed to be processed sequentially according to the order imposed by the task queue (FIFO, LIFO, priority order).
* **newScheduledThreadPool** A fixed-size thread pool that supports delayed and periodic task execution, similar to Timer.
#### Executor Lifecycle
An **Executor** implementation is likely to create threads for processing tasks. But the JVM can't exit until all the (non-daemon) threads have terminated, so failing to shut down an **Executor** could prevent the JVM from exiting. Because an **Executor** processes tasks asynchronously, at any given time the state of previously submitted tasks is not immediately obvious. Some may have completed, some may be currently running, and others may be queued awaiting execution. In shutting down an application, there is a spectrum from graceful shutdown (finish what you've started but don't accept any new work) to abrupt shutdown (turn off the power to the machine room), and various points in between.

To address the issue of execution service lifecycle, the **ExecutorService** interface extends **Executor**, adding a number of methods for lifecycle management (as well as some convenience methods for task submission).

The lifecycle implied by **ExecutorService** has three states - **running**, **shutting down**, and **terminated**. **ExecutorServices** are initially created in the running state. The **shutdown method** initiates a graceful shutdown: no new tasks are accepted but previously submitted tasks are allowed to complete - including those that have not yet begun execution. The **shutdownNow method** initiates an abrupt shutdown: it attempts to cancel outstanding tasks and does not start any tasks that are queued but not begun. Tasks submitted to an ExecutorService after it has been shut down are handled by the rejected execution handler, which might silently discard the task or might cause execute to throw the unchecked **RejectedExecutionException**. Once all tasks have completed, the **ExecutorService** transitions to the terminated state. You can wait for an **ExecutorService** to reach the terminated state with **awaitTermination**, or poll for whether it has yet terminated with **isTerminated**. It is common to follow shutdown immediately by **awaitTermination**, creating the effect of synchronously shutting down the **ExecutorService**.

The behavior of **get** varies depending on the task state (not yet started, running, completed). It returns immediately or throws an Exception if the task has already completed, but if not it blocks until the task completes. If the task completes by throwing an exception, **get** rethrows it wrapped in an **ExecutionException**; if it was cancelled, **get** throws **CancellationException**. If **get** throws **ExecutionException**, the underlying exception can be retrieved with getCause.

There are several ways to create a Future to describe a task. The **submit methods** in **ExecutorService** all return a **Future**, so that you can submit a **Runnable** or a **Callable** to an executor and get back a **Future** that can be used to retrieve the result or cancel the task. You can also explicitly instantiate a **FutureTask** for a given **Runnable** or **Callable**. (Because **FutureTask implements Runnable**, it can be submitted to an **Executor** for execution or executed directly by calling its **run method**.)

The real performance payoff of dividing a program's workload into tasks comes when there are a large number of independent, homogeneous tasks that can be processed concurrently.

**CompletionService** combines the functionality of an **Executor** and a **BlockingQueue**. You can submit **Callable** tasks to it for execution and use the queue-like methods take and poll to retrieve completed results, packaged as **Futures**, as they become available. **ExecutorCompletionService** implements **CompletionService**, delegating the computation to an **Executor**.
```java
public class Renderer {
    private final ExecutorService executor;
    Renderer(ExecutorService executor) { this.executor = executor; }
  
    void renderPage(CharSequence source) {
        final List<ImageInfo> info = scanForImageInfo(source);
        CompletionService<ImageData> completionService = new ExecutorCompletionService<ImageData>(executor);

        for (final ImageInfo imageInfo : info)
            completionService.submit(new Callable<ImageData>() {
                public ImageData call() {return imageInfo.downloadImage();}
            });
    
        renderText(source);

        try {
            for (int t = 0, n = info.size(); t < n; t++) {
                Future<ImageData> f = completionService.take();
                ImageData imageData = f.get();
                renderImage(imageData);
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        } catch (ExecutionException e) {
            throw launderThrowable(e.getCause());
        }
    }
}
```
Multiple **ExecutorCompletionServices** can share a single **Executor**, so it is perfectly sensible to create an
**ExecutorCompletionService** that is private to a particular computation while sharing a common **Executor**. When used
in this way, a **CompletionService** acts as a handle for a batch of computations in much the same way that a **Future**
acts as a handle for a single computation. By remembering how many tasks were submitted to the **CompletionService**
and counting how many completed results are retrieved, you can know when all the results for a given batch have been
retrieved, even if you use a shared **Executor**.