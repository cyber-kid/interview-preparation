# Concurrency
* [Definitions](#definitions)
* [Introducing Runnable](#introducing-runnable)
* [Creating a Thread](#creating-a-thread)
* [Creating Threads with the ExecutorService](#creating-threads-with-the-executorservice)
* [Introducing the Single-Thread Executor](#introducing-the-single-thread-executor)
* [Shutting Down a Thread Executor](#shutting-down-a-thread-executor)
* [Submitting Tasks](#submitting-tasks)
* [Submitting Task Collections](#submitting-task-collections)
* [Waiting for Results](#waiting-for-results)
* [Introducing Callable](#introducing-callable)
* [Waiting for All Tasks to Finish](#waiting-for-all-tasks-to-finish)
* [Scheduling Tasks](#scheduling-tasks)
* [Increasing Concurrency with Pools](#increasing-concurrency-with-pools)
* [Synchronizing Data Access](#synchronizing-data-access)
* [Protecting Data with Atomic Classes](#protecting-data-with-atomic-classes)
* [Improving Access with Synchronized Blocks](#improving-access-with-synchronized-blocks)
* [Synchronizing Methods](#synchronizing-methods)
* [Introducing Concurrent Collections](#introducing-concurrent-collections)
* [Understanding Memory Consistency Errors](#understanding-memory-consistency-errors)
* [Working with Concurrent Classes](#working-with-concurrent-classes)
* [Understanding Blocking Queues](#understanding-blocking-queues)
* [Understanding SkipList Collections](#understanding-skiplist-collections)
* [Understanding CopyOnWrite Collections](#understanding-copyonwrite-collections)
* [Obtaining Synchronized Collections](#obtaining-synchronized-collections)
* [Working with Parallel Streams](#working-with-parallel-streams)
* [Understanding Independent Operations](#understanding-independent-operations)
* [Creating a CyclicBarrier](#creating-a-cyclicbarrier)
* [Applying the Fork/Join Framework](#applying-the-forkjoin-framework)
* [Working with a RecursiveTask](#working-with-a-recursivetask)
* [Identifying Fork/Join Issues](#identifying-forkjoin-issues)
#### Definitions
* **Thread** is the smallest unit of execution that can be scheduled by the operating system.
* A **process** is a group of associated threads that execute in the same, shared environment.
* **Single-threaded process** is one that contains exactly one thread,
* **Multi-threaded process** is one that contains one or more threads.
* By **shared environment**, we mean that the threads in the same process share the same memory space and can communicate directly with one another.
* **Task** is a single unit of work performed by a thread. A thread can complete multiple independent tasks but only one task at a time.
* A **system thread** is created by the JVM and runs in the background of the application. For example, the garbage-collection thread is a system thread that is created by the JVM and runs in the background, helping to free memory that is no longer in use.
* A **user-defined thread** is one created by the application developer to accomplish a specific task. main() method is a user defined thread.
* **Concurrency** is an ability to execute multiple threads and processes at the same time.
* **Thread scheduler** is used by operating systems to determine which threads should be currently executing.
* **Round-robin schedule** is a thread scheduler in which each available thread receives an equal number of CPU cycles with which to execute, with threads visited in a circular order.
* A **context switch** is the process of storing a thread’s current state and later restoring the state of the thread to continue execution. This happens when allotted time is complete but the thread has not finished processing.
* A **thread priority** is a numeric value associated with a thread that is taken into consideration by the thread scheduler when determining which threads should currently be executing.
* **Polling** is the process of intermittently checking data at some fixed interval.
* A **thread pool** is a group of pre-instantiated reusable threads that are available to perform a set of arbitrary tasks.
* **Atomic** is the property of an operation to be carried out as a single unit of execution without any interference by another thread.
* A **memory consistency error** occurs when two threads have inconsistent views of what should be the same data.
* **Scaling** is the property that, as we add more resources such as CPUs, the results gradually improve.
* **Liveness** is the ability of an application to be able to execute in a timely manner.
* **Deadlock** occurs when two or more threads are blocked forever, each waiting on the other.
* **Starvation** occurs when a single thread is perpetually denied access to a shared resource or lock.
* **Livelock** occurs when two or more threads are conceptually blocked forever, although they are each still active and trying to complete their task.
* A **race condition** is an undesirable result that occurs when two tasks, which should be completed sequentially, are completed at the same time.
#### Introducing Runnable
**java.lang.Runnable** is a functional interface that takes no arguments and returns no data. The following is the definition of the Runnable interface:
```java
@FunctionalInterface public interface Runnable {
   void run();
}
```
The **Runnable** interface is commonly used to define the work a thread will execute, separate from the main application thread. Even though Runnable was made a functional interface in Java 8, the interface Runnable has existed since the very first version of Java. It was, and still is, commonly used to define a thread task by creating a class that implements the Runnable interface. It is also useful if you need to pass information to your Runnable object to be used by the **run()** method.
#### Creating a Thread
The simplest way to execute a thread is by using the **java.lang.Thread** class. Executing a task with **Thread** is a two-step process. First you define the **Thread** with the corresponding task to be done. Then you start the task by using the **Thread.start()** method. Java does not provide any guarantees about the order in which a thread will be processed once it is started. It may be executed immediately or delayed for a significant amount of time.

Defining the task, or work, that a Thread instance will execute can be done two ways in Java:
* Provide a **Runnable** object or lambda expression to the **Thread** constructor.
* Create a class that extends **Thread** and override the **run()** method.

Example #1:
```java
public class PrintData implements Runnable {
   public void run() {
      for(int i=0; i<3; i++) System.out.println("Printing record: "+i);
   }
   public static void main(String[] args) {
      (new Thread(new PrintData())).start();
   }
}
```
Example #2:
```java
public class ReadInventoryThread extends Thread {
   public void run() {
      System.out.println("Printing zoo inventory"); }
   public static void main(String[] args) {
      (new ReadInventoryThread()).start();
   }
}
```
The following are some reasons to prefer one method over the other in Java:
* If you need to define your own Thread rules upon which multiple tasks will rely, such as a priority Thread, extending Thread may be preferable.
* Since Java doesn't support multiple inheritance, extending Thread does not allow you to extend any other class, whereas implementing Runnable lets you extend another class.
* Implementing Runnable is often a better object-oriented design practice since it separates the task being performed from the Thread object performing it.
* Implementing Runnable allows the class to be used by numerous Concurrency API classes.
  

Anytime you create a **Thread** instance, make sure that you remember to start the task with the **Thread.start()** method. When a **Thread** or **Runnable** is created but no **start()** method is called the code will compile but none will actually execute a task on a separate processing thread. Instead, the thread that made the call will be used to execute the task, causing the thread to wait until each **run()** method is complete before moving on to the next line.
#### Creating Threads with the ExecutorService
With the announcement of the Concurrency API, Java introduced the **ExecutorService**, which creates and manages threads for you. You first obtain an instance of an **ExecutorService** interface, and then you send the service tasks to be processed. The framework includes numerous useful features, such as thread pooling and scheduling, which would be cumbersome for you to implement in every project. Therefore, it is recommended that you use this framework anytime you need to create and execute a separate task, even if you need only a single thread.

#### Introducing the Single-Thread Executor
The Concurrency API includes the **Executors** factory class that can be used to create instances of the **ExecutorService** object. Use the **newSingleThreadExecutor()** method to obtain an **ExecutorService** instance and the **execute()** method to perform asynchronous tasks.

Example:
```java
import java.util.concurrent.*;
public class ZooInfo {
   public static void main(String[] args) {
      ExecutorService service = null;
      try {
         service = Executors.newSingleThreadExecutor();
         System.out.println("begin");
         service.execute(() -> System.out.println("Printing zoo inventory"));
         service.execute(() -> {for(int i=0; i<3; i++)
            System.out.println("Printing record: "+i);}
         );
         service.execute(() -> System.out.println("Printing zoo inventory"));
         System.out.println("end");
      } finally {
         if(service != null) service.shutdown();
      }
   }
}
```
This example uses only one thread, which means that the threads will order their results. With a single-thread executor, results are guaranteed to be executed in the order in which they are added to the executor service.
#### Shutting Down a Thread Executor
Once you have finished using a thread executor, it is important that you call the **shutdown()** method. A thread executor creates a non-daemon thread on the first task that is executed, so failing to call **shutdown()** will result in your application never terminating. The shutdown process for a thread executor involves first rejecting any new tasks submitted to the thread executor while continuing to execute any previously submitted tasks. During this time, calling **isShutdown()** will return true, while **isTerminated()** will return false. If a new task is submitted to the thread executor while it is shutting down, a **RejectedExecutionException** will be thrown. Once all active tasks have been completed, **isShutdown()** and **isTerminated()** will both return true.

You should be aware that **shutdown()** does not actually stop any tasks that have already been submitted to the thread executor. What if you want to cancel all running and upcoming tasks? The **ExecutorService** provides a method called **shutdownNow()**, which attempts to stop all running tasks and discards any that have not been started yet. Note that **shutdownNow()** attempts to stop all running tasks. Lastly, **shutdownNow()** returns a **List<Runnable>** of tasks that were submitted to the thread executor but that were never started.

Unfortunately, the **ExecutorService** interface does not implement **AutoCloseable**, so you cannot use a try-with-resources statement.
#### Submitting Tasks
You can submit tasks to an **ExecutorService** instance multiple ways. The first method is **execute()**, is inherited from the **Executor** interface, which the **ExecutorService** interface extends. The **execute()** method takes a **Runnable** lambda expression or instance and completes the task asynchronously. Because the return type of the method is void, it does not tell us anything about the result of the task. It is considered a “fire-and-forget” method. Fortunately, the writers of the Java added **submit()** methods to the **ExecutorService** interface, which, like **execute()**, can be used to complete tasks asynchronously. Unlike **execute()**, though, **submit()** returns a **Future** object that can be used to determine if the task is complete. It can also be used to return a generic result object after the task has been completed.

Methods to execute tasks:
* **void execute(Runnable command)** Executes a Runnable task at some point in the future.
* **Future<?> submit(Runnable task)** Executes a Runnable task at some point in the future and returns a Future representing the task.
* **< T > Future< T > submit(Callable< T > task)** Executes a Callable task at some point in the future and returns a Future representing the pending results of the task.
* **< T > List<Future< T >> invokeAll(Collection<? extends Callable< T >> tasks) throws InterruptedException** Executes the given tasks, synchronously returning the results of all tasks as a Collection of Future objects, in the same order they were in the original collection.
* **< T > T invokeAny(Collection<? extends Callable< T >> tasks) throws InterruptedException, ExecutionException** Executes the given tasks, synchronously returning the result of one of finished tasks, cancelling any unfinished tasks.
#### Submitting Task Collections
**invokeAll()** and **invokeAny()** methods take a **Collection** object containing a list of tasks to execute. Both of these methods also execute synchronously. By synchronous, we mean that unlike the other methods used to submit tasks to a thread executor, these methods will wait until the results are available before returning control to the enclosing program. The **invokeAll()** method executes all tasks in a provided collection and returns a **List** of ordered **Future** objects, with one **Future** object corresponding to each submitted task, in the order they were in the original collection. Even though **Future.isDone()** returns true for each element in the returned List, a task could have completed normally or thrown an exception. The **invokeAny()** method executes a collection of tasks and returns the result of one of the tasks that successfully completes execution, cancelling all unfinished tasks. While the first task to finish is often returned, this behavior is not guaranteed, as any completed task can be returned by this method. Finally, the **invokeAll()** method will wait indefinitely until all tasks are complete, while the **invokeAny()** method will wait indefinitely until at least one task completes. The **ExecutorService** interface also includes overloaded versions of **invokeAll()** and **invokeAny()** that take a timeout value and **TimeUnit** parameter.
#### Waiting for Results
How do we know when a task submitted to an **ExecutorService** is complete? As mentioned in the last section, the **submit()** method returns a **java.util.concurrent.Future< V >** object, that can be used to determine this result:
```java
Future<?> future = service.submit(() -> System.out.println("Hello Zoo"));
```
The Future class includes methods that are useful in determining the state of a task:
* **boolean isDone()** Returns true if the task was completed, threw an exception, or was cancelled.
* **boolean isCancelled()** Returns true if the task was cancelled before it completely normally.
* **boolean cancel()** Attempts to cancel execution of the task.
* **V get()** Retrieves the result of a task, waiting endlessly if it is not yet available.
* **V get(long timeout,TimeUnit unit)** Retrieves the result of a task, waiting the specified amount of time. If the result is not ready by the time the timeout is reached, a checked TimeoutException will be thrown.

What is the return value of this task? As **Future< V >** is a generic class, the type V is determined by the return type of the **Runnable** method. Since the return type of **Runnable.run()** is void, the **get()** method always returns null.
#### Introducing Callable
When the Concurrency API was released in Java 5, the new **java.util.concurrent.Callable** interface was added, which is similar to **Runnable** except that its **call()** method returns a value and can throw a checked exception. As you may remember from the definition of **Runnable**, the **run()** method returns void and cannot throw any checked exceptions. Along with Runnable, Callable was also made a functional interface in Java 8. The following is the definition of the Callable interface:
```java
@FunctionalInterface public interface Callable<V> {
   V call() throws Exception;
}
```
The **Callable** interface was introduced as an alternative to the **Runnable** interface, since it allows more details to be retrieved easily from the task after it is completed. The **ExecutorService** includes an overloaded version of the **submit()** method that takes a **Callable** object and returns a generic **Future< T >** object.
#### Waiting for All Tasks to Finish
First, we shut down the thread executor using the **shutdown()** method. Next, we use the **awaitTermination(long timeout, TimeUnit unit)** method available for all thread executors. The method waits the specified time to complete all tasks, returning sooner if all tasks finish or an **InterruptedException** is detected.

#### Scheduling Tasks
Oftentimes in Java, we need to schedule a task to happen at some future time. We might even need to schedule the task to happen repeatedly, at some set interval. The **ScheduledExecutorService**, which is a subinterface of **ExecutorService**, can be used for just such a task. Like **ExecutorService**, we obtain an instance of **ScheduledExecutorService** using a factory method in the **Executors** class, as shown in the following snippet:
```java
ScheduledExecutorService service = Executors.newSingleThreadScheduledExecutor();
```
Note that we could implicitly cast an instance of **ScheduledExecutorService** to **ExecutorService**, although doing so would remove access to the scheduled methods that we want to use.

Methods of **ScheduledExecutorService** class:
* **schedule(Callable< V > callable, long delay, TimeUnit unit)**  Creates and executes a Callable task after the given delay.
* **schedule(Runnable command, long delay, TimeUnit unit)** Creates and executes a Runnable task after the given delay.
* **scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnit unit)** Creates and executes a Runnable task after the given initial delay, creating a new task every period value that passes.
* **scheduleAtFixedDelay(Runnable command, long initialDelay, long delay, TimeUnit unit)** Creates and executes a Runnable task after the given initial delay and subsequently with the given delay between the termination of one execution and the commencement of the next.

The first two **schedule()** methods take a Callable or Runnable, respectively, perform the task after some delay, and return a **ScheduledFuture< V >** instance. **ScheduledFuture< V >** is identical to the **Future< V >** class, except that it includes a **getDelay()** method that returns the delay set when the process was created.

Notice that neither of the methods, **scheduleAtFixedDelay() and scheduleAtFixedRate()**, take a **Callable** object as an input parameter. Since these tasks are scheduled to run infinitely, as long as the **ScheduledExecutorService** is still alive, they would generate an endless series of Future objects.
#### Increasing Concurrency with Pools
We now present three additional factory methods in the Executors class that act on a pool of threads, rather than on a single thread.
* **newSingleThreadExecutor()** (return type is ExecutorService) Creates a single-threaded executor that uses a single worker thread operating off an unbounded queue. Results are processed sequentially in the order in which they are submitted.
* **newSingleThreadScheduledExecutor()** (return type is ScheduledExecutorService) Creates a single-threaded executor that can schedule commands to run after a given delay or to execute periodically.
* **newCachedThreadPool()** (return type is ExecutorService) Creates a thread pool that creates new threads as needed, but will reuse previously constructed threads when they are available.
* **newFixedThreadPool(int nThreads)** (return type is ExecutorService) Creates a thread pool that reuses a fixed number of threads operating off a shared unbounded queue.
* **newScheduledThreadPool(int nThreads)** (return type is ScheduledExecutorService) Creates a thread pool that can schedule commands to run after a given delay or to execute periodically.

The difference between a single-thread and a pooled-thread executor is what happens when a task is already running. While a single-thread executor will wait for an available thread to become available before running the next task, a pooled-thread executor can execute the next task concurrently. If the pool runs out of available threads, the task will be queued by the thread executor and wait to be completed.

The **newCachedThreadPool()** method will create a thread pool of unbounded size, allocating a new thread anytime one is required or all existing threads are busy. This is commonly used for pools that require executing many short-lived asynchronous tasks. For long-lived processes, usage of this executor is strongly discouraged, as it could grow to encompass a large number of threads over the application life cycle.

The **newFixedThreadPool()** takes a number of threads and allocates them all upon creation. As long as our number of tasks is less than our number of threads, all tasks will be executed concurrently. If at any point the number of tasks exceeds the number of threads in the pool, they will wait in similar manner as you saw with a single-thread executor. In fact, calling **newFixedThreadPool()** with a value of 1 is equivalent to calling **newSingleThreadExecutor()**.
#### Synchronizing Data Access
Recall that thread safety is the property of an object that guarantees safe execution by multiple threads at the same time. Now that we have multiple threads capable of accessing the same objects in memory, we have to make sure to organize our access to this data such that we don’t end up with invalid or unexpected results.
#### Protecting Data with Atomic Classes
With the release of the Concurrency API, Java added a new **java.util.concurrent.atomic** package to help coordinate access to primitive values and object references. As with most classes in the Concurrency API, these classes are added solely for convenience. Since accessing primitives and references in Java is common in shared environments, the Concurrency API includes numerous useful classes that are conceptually the same as our primitive classes but that support atomic operations.

Atomic classes:
* **AtomicBoolean** A boolean value that may be updated atomically
* **AtomicInteger** An int value that may be updated atomically
* **AtomicIntegerArray** An int array in which elements may be updated atomically
* **AtomicLong** A long value that may be updated atomically
* **AtomicLongArray** A long array in which elements may be updated atomically
* **AtomicReference** A generic object reference that may be updated atomically
* **AtomicReferenceArray** An array of generic object references in which elements may be updated atomically

Common atomic methods:
* **get()** Retrieve the current value
* **set()** Set the given value, equivalent to the assignment = operator
* **getAndSet()** Atomically sets the new value and returns the old value
* **incrementAndGet()** For numeric classes, atomic pre-increment operation equivalent to ++value
* **getAndIncrement()** For numeric classes, atomic post-increment operation equivalent to value++
* **decrementAndGet()** For numeric classes, atomic pre-decrement operation equivalent to --value
* **getAndDecrement()** For numeric classes, atomic post-decrement operation equivalent to value--
* **compareAndSet(expectedValue, newValue)** For numeric classes, it first checks if the current value is same as the expected value and if so, updates to the new value.
* **addAndGet(int i)** For numeric classes, atomically adds the given value to the current value and returns the new value.

Using the atomic classes ensures that the data is consistent between workers and that no values are lost due to concurrent modifications.
#### Improving Access with Synchronized Blocks
The most common technique is to use a monitor, also called a lock, to synchronize access. A monitor is a structure that supports mutual exclusion or the property that at most one thread is executing a particular segment of code at a given time. In Java, any **Object** can be used as a monitor, along with the **synchronized** keyword, as shown in the following example:
```java
SheepManager manager = new SheepManager();
synchronized(manager) {
   // Work to be completed by one thread at a time
}
```
This example is referred to as a synchronized block. Each thread that arrives will first check if any threads are in the block. In this manner, a thread “acquires the lock” for the monitor. If the lock is available, a single thread will enter the block, acquiring the lock and preventing all other threads from entering. While the first thread is executing the block, all threads that arrive will attempt to acquire the same lock and wait for first thread to finish. Once a thread finishes executing the block, it will release the lock, allowing one of the waiting threads to proceed.

We could have synchronized on any object, so long as it was the same object. For example, the following code snippet would have also worked:
```java
private final Object lock = new Object();
private void incrementAndReport() {
   synchronized(lock) {
      System.out.print((++sheepCount)+" ");
   }
}
```
Although we didn’t need to make the lock variable final, doing so ensures that it is not reassigned after threads start using it.
#### Synchronizing Methods
In the previous example, we established our monitor using **synchronized(this)** around the body of the method. Java actually provides a more convenient compiler enhancement for doing so. We can add the **synchronized** modifier to any instance method to synchronize automatically on the object itself.

Example:
```java
private synchronized void incrementAndReport() {
   System.out.print((++sheepCount)+" ");
}
```
We can also add the **synchronized** modifier to **static** methods. What object is used as the monitor when we synchronize on a static method? The **Class** object.
#### Introducing Concurrent Collections
Using the concurrent collections is extremely convenient in practice. It also prevents us from introducing mistakes in own custom implementation, such as if we forgot to synchronize one of the accessor methods. In fact, the concurrent collections often include performance enhancements that avoid unnecessary synchronization. Accessing collections from across multiple threads is so common that the writers of Java thought it would be a good idea to have alternate versions of many of the regular collections classes just for multi-threaded access.

Example:
```java
private Map<String,Object> foodData = new ConcurrentHashMap<String,Object>();
```
#### Understanding Memory Consistency Errors
The purpose of the concurrent collection classes is to solve common memory consistency errors. Conceptually, we want writes on one thread to be available to another thread if it accesses the concurrent collection after the write has occurred.

When two threads try to modify the same non-concurrent collection, the JVM may throw a **ConcurrentModificationException** at runtime. In fact, it can happen with a single thread. Take a look at the following code snippet:
```java
Map<String, Object> foodData = new HashMap<String, Object>();
foodData.put("penguin", 1);
foodData.put("flamingo", 2);
for(String key: foodData.keySet())
   foodData.remove(key);
```
This snippet will throw a **ConcurrentModificationException** at runtime, since the iterator **keyset()** is not properly updated after the first element is removed. Changing the first line to use a **ConcurrentHashMap** will prevent the code from throwing an exception at runtime.
#### Working with Concurrent Classes
You should use a concurrent collection class anytime that you are going to have multiple threads modify a collections object outside a synchronized block or method, even if you don’t expect a concurrency problem. On the other hand, if all of the threads are accessing an established immutable or read-only collection, a concurrent collection class is not required.

In the same way that we instantiate an **ArrayList** object but pass around a **List** reference, it is considered a good practice to instantiate a concurrent collection but pass it around using a non-concurrent interface whenever possible. This has some similarities with the factory pattern, as the users of these objects may not be aware of the underlying implementation. In some cases, the callers may need to know that it is a concurrent collection so that a concurrent interface or class is appropriate, but for the majority of circumstances, that distinction is not necessary.

Examples:
* **ConcurrentHashMap** implements **ConcurrentMap**
* **ConcurrentLinkedDeque** implements **Deque**
* **ConcurrentLinkedQueue** implements **Queue**
* **ConcurrentSkipListMap** implements **ConcurrentMap,SortedMap,NavigableMap**
* **ConcurrentSkipListSet** implements **SortedSet,NavigableSet**
* **CopyOnWriteArrayList** implements **List**
* **CopyOnWriteArraySet** implements **Set**
* **LinkedBlockingDeque** implements **BlockingQueue,BlockingDeque**
* **LinkedBlockingQueue** implements **BlockingQueue**
#### Understanding Blocking Queues
As you may have noticed, there are two queue classes that implement blocking interfaces: **LinkedBlockingQueue** and **LinkedBlockingDeque**. The **BlockingQueue** is just like a regular **Queue**, except that it includes methods that will wait a specific amount of time to complete an operation.

Since **BlockingQueue** inherits all of the methods from **Queue**, we skip the inherited methods and present the new waiting methods:
* **offer(E e, long timeout, TimeUnit unit)** Adds item to the queue waiting the specified time, returning false if time elapses before space is available
* **poll(long timeout, TimeUnit unit)** Retrieves and removes an item from the queue, waiting the specified time, returning null if the time elapses before the item is available

Since **LinkedBlockingQueue** implements both **Queue** and **BlockingQueue**, we can use methods available to both, such as those that don’t take any wait arguments. The methods can each throw a checked **InterruptedException**, as they can be interrupted before they finish waiting for a result; therefore they must be properly caught.

The **LinkedBlockingDeque** class maintains a doubly linked list between elements and implements a **BlockingDeque** interface. The **BlockingDeque** interface extends **Deque** much in the same way that **BlockingQueue** extends **Queue**, providing numerous waiting methods:
* **offerFirst(E e, long timeout, TimeUnit unit)** Adds an item to the front of the queue, waiting a specified time, returning false if time elapses before space is available
* **offerLast(E e, long timeout, TimeUnit unit)** Adds an item to the tail of the queue, waiting a specified time, returning false if time elapses before space is available
* **pollFirst(long timeout, TimeUnit unit)** Retrieves and removes an item from the front of the queue, waiting the specified time, returning null if the time elapses before the item is available
* **pollLast(long timeout, TimeUnit unit)** Retrieves and removes an item from the tail of the queue, waiting the specified time, returning null if the time elapses before the item is available
#### Understanding SkipList Collections
The **SkipList** classes, **ConcurrentSkipListSet** and **ConcurrentSkipListMap**, are concurrent versions of their sorted counterparts, **TreeSet** and **TreeMap**, respectively. They maintain their elements or keys in the natural ordering of their elements.
#### Understanding CopyOnWrite Collections
**CopyOnWriteArrayList** and **CopyOnWriteArraySet**, behave a little differently than the other concurrent examples that you have seen. These classes copy all of their elements to a new underlying structure anytime an element is added, modified, or removed from the collection. By a modified element, we mean that the reference in the collection is changed. Modifying the actual contents of the collection will not cause a new structure to be allocated.

Although the data is copied to a new underlying structure, our reference to the object does not change. This is particularly useful in multi-threaded environments that need to iterate the collection. Any iterator established prior to a modification will not see the changes, but instead it will iterate over the original elements prior to the modification.

Let’s take a look at how this works with an example:
```java
List<Integer> list = new CopyOnWriteArrayList<>(Arrays.asList(4,3,52));
for(Integer item: list) {
   System.out.print(item+" ");
   list.add(9);
}
System.out.println(); System.out.println("Size: "+list.size());
```
When executed as part of a program, this code snippet outputs the following: 4 3 52 Size: 6

Despite adding elements to the array while iterating over it, only those elements in the collection at the time the **for()** loop was created were accessed. Alternatively, if we had used a regular **ArrayList** object, a **ConcurrentModificationException** would have been thrown at runtime. With either class, though, we avoid entering an infinite loop in which elements are constantly added to the array as we iterate over them.
#### Obtaining Synchronized Collections
Besides the concurrent collection classes that we have covered, the Concurrency API also includes methods for obtaining synchronized versions of existing non-concurrent collection objects. These methods, defined in the **Collections** class, contain synchronized methods that operate on the inputted collection and return a reference that is the same type as the underlying collection.

Examples:
* **synchronizedCollection(Collection< T > c)**
* **synchronizedList(List< T > list)**
* **synchronizedMap(Map<K,V> m)**
* **synchronizedNavigableMap(NavigableMap<K,V> m)**
* **synchronizedNavigableSet(NavigableSet< T > s)**
* **synchronizedSet(Set< T > s)**
* **synchronizedSortedMap(SortedMap<K,V> m)**
* **synchronizedSortedSet(SortedSet< T > s)**

When should you use these methods? If you know at the time of creation that your object requires synchronization, then you should use one of the concurrent collection classes. On the other hand, if you are given an existing collection that is not a concurrent class and need to access it among multiple threads, you can wrap it using the methods listed above.

While the methods listed above synchronize access to the data elements, such as the **get()** and **set()** methods, they do not synchronize access on any **iterators** that you may create from the synchronized collection. Therefore, it is imperative that you use a synchronization block if you need to iterate over any of the returned collections.

Unlike the concurrent collections, the synchronized collections also throw an exception if they are modified within an iterator by a single thread. For example, take a look at the following modification of our earlier example:
#### Working with Parallel Streams
A serial stream is a stream in which the results are ordered, with only one entry being processed at a time. A parallel stream is a stream that is capable of processing results concurrently, using multiple threads.

There are two ways to create parallel stream: 

**parallel()** The first way to create a parallel stream is from an existing stream. You just call parallel() on an existing stream to convert it to one that supports multi-threaded processing, as shown in the following code:
```java
Stream<Integer> stream = Arrays.asList(1,2,3,4,5,6).stream();
Stream<Integer> parallelStream = stream.parallel();
```
Be aware that **parallel()** is an intermediate operation that operates on the original stream.

**parallelStream()** The second way to create a parallel stream is from a Java collection class. The Collection interface includes a method **parallelStream()** that can be called on any collection and returns a parallel stream.
```java
Stream<Integer> parallelStream2 = Arrays.asList(1,2,3,4,5,6).parallelStream();
```
The **Stream** interface includes a method **isParallel()** that can be used to test if the instance of a stream supports parallel processing. Some operations on streams preserve the parallel attribute, while others do not. For example, the **Stream.concat(Stream s1, Stream s2)** is parallel if either s1 or s2 is parallel. On the other hand, **flatMap()** creates a new stream that is not parallel by default, regardless of whether the underlying elements were parallel.
#### Understanding Independent Operations
Parallel streams can improve performance because they rely on the property that many stream operations can be executed independently. By independent operations, we mean that the results of an operation on one element of a stream do not require or impact the results of another element of the stream.

**Processing Parallel Reductions** Besides possibly improving performance and modifying the order of operations, using parallel streams can impact how you write your application. Reduction operations on parallel streams are referred to as parallel reductions. The results for parallel reductions can be different from what you expect when working with serial streams.

**Creating Unordered Streams** All of the streams are considered ordered by default. It is possible to create an unordered stream from an ordered stream, similar to how you create a parallel stream from a serial stream:
```java
Arrays.asList(1,2,3,4,5,6).stream().unordered();
```
This method does not actually reorder the elements; it just tells the JVM that if an order-based stream operation is applied, the order can be ignored. For example, calling **skip(5)** on an unordered stream will skip any 5 elements, not the first 5 required on an ordered stream. For serial streams, using an unordered version has no effect, but on parallel streams, the results can greatly improve performance:
```java
Arrays.asList(1,2,3,4,5,6).stream().unordered().parallel();
```
**Combining Results with reduce()** With parallel streams, though, we now have to be concerned about order. What if the elements of a string are combined in the wrong order? The Streams API prevents this problem, while still allowing streams to be processed in parallel, as long as the arguments to the **reduce()** operation adhere to certain principles.

Requirements for **reduce()** arguments:
* The identity must be defined such that for all elements in the stream u, combiner.apply(identity, u) is equal to u.
* The accumulator operator op must be associative and stateless such that (a op b) op c is equal to a op (b op c).
* The combiner operator must also be associative and stateless and compatible with the identity, such that for all u and t combiner.apply(u,accumulator.apply(identity,t)) is equal to accumulator.apply(u,t).

If you follow these principles when building your **reduce()** arguments, then the operations can be performed using a parallel stream and the results will be ordered as they would be with a serial stream. Note that these principles still apply to the identity and accumulator when using the one- or two-argument version of reduce() on parallel streams.

**Combining Results with collect()** Like reduce(), the Streams API includes a three-argument version of **collect()** that takes accumulator and combiner operators, along with a supplier operator instead of an identity. Also like reduce(), the accumulator and combiner operations must be associative and stateless, with the combiner operation compatible with the accumulator operator, as previously discussed. In this manner, the three-argument version of collect() can be performed as a parallel reduction, as shown in the following example:
```java
Stream<String> stream = Stream.of("w", "o", "l", "f").parallel();
SortedSet<String> set = stream.collect(ConcurrentSkipListSet::new, Set::add, Set::addAll);
System.out.println(set); // [f, l, o, w]
```
Recall that elements in a **ConcurrentSkipListSet** are sorted according to their natural ordering. You should use a concurrent collection to combine the results, ensuring that the results of concurrent threads do not cause a **ConcurrentModificationException**.

**Using the One-Argument collect() Method**

Requirements for Parallel Reduction with **collect()**:
* The stream is parallel.
* The parameter of the collect operation has the **Collector.Characteristics.CONCURRENT** characteristic.
* Either the stream is unordered, or the collector has the **characteristic Collector.Characteristics.UNORDERED**.

Any class that implements the **Collector** interface includes a **characteristics()** method that returns a set of available attributes for the collector. While **Collectors.toSet()** does have the **UNORDERED** characteristic, it does not have the **CONCURRENT** characteristic; The **Collectors** class includes two sets of methods for retrieving collectors that are both **UNORDERED and CONCURRENT**, **Collectors.toConcurrentMap()** and **Collectors.groupingByConcurrent()**, and therefore it is capable of performing parallel reductions efficiently. Like their non-concurrent counterparts, there are overloaded versions that take additional arguments.
#### Creating a CyclicBarrier
The **CyclicBarrier** takes in its constructors a **limit value**, indicating the number of threads to wait for. As each thread finishes, it calls the **await()** method on the cyclic barrier. Once the specified number of threads have each called **await()**, the barrier is released and all threads can continue.

The following is a reimplementation of our LionPenManager class that uses CyclicBarrier objects to coordinate access:
```java
import java.util.concurrent.*;

public class LionPenManager {
   private void removeAnimals() { System.out.println("Removing animals"); }
   private void cleanPen() { System.out.println("Cleaning the pen"); }
   private void addAnimals() { System.out.println("Adding animals"); }

   public void performTask(CyclicBarrier c1, CyclicBarrier c2) {
      try {
         removeAnimals();
         c1.await();
         cleanPen();
         c2.await();
         addAnimals();
      } catch (InterruptedException | BrokenBarrierException e) {
         // Handle checked exceptions here }
      }
      public static void main(String[] args) {
          ExecutorService service = null;
          try {
             service = Executors.newFixedThreadPool(4);
             LionPenManager manager = new LionPenManager();
             CyclicBarrier c1 = new CyclicBarrier(4);
             CyclicBarrier c2 = new CyclicBarrier(4,  () -> System.out.println("*** Pen Cleaned!"));
             for(int i=0; i<4; i++) service.submit(() -> manager.performTask(c1,c2));
          } finally {
             if(service != null) service.shutdown();
          }
      }
}
```
In this example, we have updated the performTask() to use CyclicBarrier objects. Like synchronizing on the same object, coordinating a task with a CyclicBarrier requires the object to be static or passed to the thread performing the task. We also add a try/catch block in the performTask() method, as the await() method throws multiple checked exceptions.

The following is sample output based on this revised implementation of our LionPenManager class:
```
Removing animals
Removing animals
Removing animals
Removing animals
Cleaning the pen
Cleaning the pen
Cleaning the pen
Cleaning the pen
*** Pen Cleaned!
Adding animals
Adding animals
Adding animals
Adding animals
```
In this example, we used two different constructors for our CyclicBarrier objects, the latter of which called a Runnable method upon completion.
#### Applying the Fork/Join Framework
In most of the examples in this chapter, we knew at the start of the process exactly how many threads and tasks we needed to perform. Sometimes, we aren’t so lucky. It may be that we have five threads, but we have no idea how many tasks need to be performed. When a task gets too complicated, we can split the task into multiple other tasks using the fork/join framework.

Applying the fork/join framework requires us to perform three steps:
1. Create a ForkJoinTask.
2. Create the ForkJoinPool.
3. Start the ForkJoinTask.

The first step is the most complex, as it requires defining the recursive process. Fortunately, the second and third steps are easy and can each be completed with a single line of code. You can implement the fork/join solution by extending one of two classes, **RecursiveAction** and **RecursiveTask**, both of which implement the **ForkJoinTask** interface.

The first class, **RecursiveAction**, is an abstract class that requires us to implement the **compute()** method, which returns void, to perform the bulk of the work. The second class, **RecursiveTask**, is an abstract generic class that requires us to implement the **compute()** method, which returns the generic type, to perform the bulk of the work.

Example:
```java
import java.util.*;
import java.util.concurrent.*;

public class WeighAnimalAction extends RecursiveAction {
   private int start;
   private int end;
   private Double[] weights;

   public WeighAnimalAction(Double[] weights, int start, int end) {
      this.start = start;
      this.end = end;
      this.weights = weights;
   }

   protected void compute() {
      if(end-start <= 3) {
          for(int i=start; i<end; i++) {
             weights[i] = (double)new Random().nextInt(100);
             System.out.println("Animal Weighed: "+i);
          }
      } else {
          int middle = start+((end-start)/2);
          System.out.println("[start="+start+",middle="+middle+",end="+end+"]");
          invokeAll(new WeighAnimalAction(weights,start,middle),  new WeighAnimalAction(weights,middle,end));
      }
   }
    public static void main(String[] args) {
        Double[] weights = new Double[10];
        ForkJoinTask<?> task = new WeighAnimalAction(weights,0,weights.length);
        ForkJoinPool pool = new ForkJoinPool();
        pool.invoke(task);

         // Print results
        System.out.println(); System.out.print("Weights: "); Arrays.asList(weights).stream().forEach(d -> System.out.print(d.intValue()+" "));
     }
}
```
We start off by defining the task and the arguments on which the task will operate, such as start, end, and weights. We then override the abstract compute() method, defining our base and recursive processes. For the base case, we weigh the animal if there are at most three left in the set. For simplicity, this base case assigns a random number from 0 to 100 as the weight. For the recursive case, we split the work from one WeighAnimalAction object into two WeighAnimalAction instances, dividing the available indices between the two tasks. Some subtasks may end up with little or no work to do, which is fine, as long as they terminate in a base case. Once the task class is defined, creating the ForkJoinPool and starting the task is quite easy.

By default, the ForkJoinPool class will use the number of processors to determine how many threads to create. The following is a sample output of this code:
```
[start=0,middle=5,end=10]
[start=0,middle=2,end=5]
Animal Weighed: 0
Animal Weighed: 2
[start=5,middle=7,end=10]
Animal Weighed: 1
Animal Weighed: 3
Animal Weighed: 5
Animal Weighed: 6
Animal Weighed: 7
Animal Weighed: 8
Animal Weighed: 9
Animal Weighed: 4

Weights: 94 73 8 92 75 63 76 60 73 3
```
#### Working with a RecursiveTask
Let’s say that we want to compute the sum of all weight values while processing the data. Instead of extending RecursiveAction, we could extend the generic RecursiveTask to calculate and return each sum in the compute() method. The following is an updated implementation that uses RecursiveTask<Double>:
```java
public class WeighAnimalTask extends RecursiveTask<Double> {
private int start;
private int end;
private Double[] weights;

   public WeighAnimalTask(Double[] weights, int start, int end) {
       this.start = start;
       this.end = end;
       this.weights = weights;
   }

   protected Double compute() {
      if(end-start <= 3) {
         double sum = 0;
         for(int i=start; i<end; i++) {
             weights[i] = (double)new Random().nextInt(100);
             System.out.println("Animal Weighed: "+i);
             sum += weights[i];
         }
         return sum;
      } else {
          int middle = start+((end-start)/2);
          System.out.println("[start="+start+",middle="+middle+",end="+end+"]");
          RecursiveTask<Double> otherTask = new WeighAnimalTask(weights,start,middle);
          otherTask.fork();
          return new WeighAnimalTask(weights,middle,end).compute() + otherTask.join();
       }
    }
}
```
While our base case is mostly unchanged, except for returning a sum value, the recursive case is quite different. Since the invokeAll() method doesn’t return a value, we instead issue a fork() and join() command to retrieve the recursive data. The fork() method instructs the fork/join framework to complete the task in a separate thread, while the join() method causes the current thread to wait for the results. In this example, we compute the [middle, end] range using the current thread, since we already have one available, and the [start, middle] range using a separate thread. We then combine the results, waiting for the otherTask to complete.
#### Identifying Fork/Join Issues
Tips for Reviewing a Fork/Join Class:
* The class should extend **RecursiveAction** or **RecursiveTask**.
* If the class extends **RecursiveAction**, then it should override a protected **compute()** method that takes no arguments and returns void.
* If the class extends **RecursiveTask**, then it should override a protected **compute()** method that takes no arguments and returns a generic type listed in the class definition.
* The **invokeAll()** method takes two instances of the fork/join class and does not return a result.
* The **fork()** method causes a new task to be submitted to the pool and is similar to the thread executor **submit()** method.
* The **join()** method is called after the **fork()** method and causes the current thread to wait for the results of a subtask.
* Unlike **fork()**, calling **compute()** within a **compute()** method causes the task to wait for the results of the subtask.
* The **fork()** method should be called before the current thread performs a **compute()** operation, with **join()** called to read the results afterward.
* Since **compute()** takes no arguments, the constructor of the class is often used to pass instructions to the task.