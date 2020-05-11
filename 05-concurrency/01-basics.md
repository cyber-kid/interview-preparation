# Concurrency basics

* [Risks of Threads](#risks-of-threads)
* [Thread Safety](#thread-safety)
* [Atomicity](#atomicity)
* [Locking](#locking)
* [Visibility](#visibility)
* [Thread confinement](#thread-confinement)
* [Instance Confinement](#instance-confinement)
* [Immutable objects](#immutable-objects)
* [Publish objects](#publish-objects)
* [Designing a Thread safe Class](#designing-a-thread-safe-class)
* [The Java Monitor Pattern](#the-java-monitor-pattern)
* [Delegating Thread Safety](#delegating-thread-safety)
* [Client-side Locking](#client-side-locking)

#### Risks of Threads
Because threads share the same memory address space and run concurrently, they can access or modify variables that other threads might be using. This is a tremendous convenience, because it makes data sharing much easier than would other inter thread communications mechanisms. But it is also a significant risk: threads can be confused by having data change unexpectedly. Allowing multiple threads to access and modify the same variables introduces an element of non sequentiality into an otherwise sequential programming model, which can be confusing and difficult to reason about. For a multithreaded program's behavior to be predictable, access to shared variables must be properly coordinated so that threads do not interfere with one another.

In the absence of synchronization, the compiler, hardware, and runtime are allowed to take substantial liberties with the timing and ordering of actions, such as caching variables in registers or processor local caches where they are temporarily (or even permanently) invisible to other threads. These tricks are in aid of better performance and are generally desirable, but they place a burden on the developer to clearly identify where data is being shared across threads so that these optimizations do not undermine safety.

**race condition**

A race condition occurs when the correctness of a computation depends on the relative timing or interleaving of multiple threads by the runtime; in other words, when getting the right answer relies on lucky timing. The most common type of race condition is check then act, where a potentially stale observation is used to make a decision on what to do next.

**livenes**

A liveness failure occurs when an activity gets into a state such that it is permanently unable to make forward progress. The use of threads introduces additional liveness risks. For example, if thread A is waiting for a resource that thread B holds exclusively, and B never releases it, A will wait forever. Like most concurrency bugs, bugs that cause liveness failures can be elusive because they depend on the relative timing of events in different threads, and therefore do not always manifest themselves in development or testing.

**performance**

In well designed concurrent applications the use of threads is a net performance gain, but threads nevertheless carry some degree of runtime overhead. Context switches when the scheduler suspends the active thread temporarily so another thread can run are more frequent in applications with many threads, and have significant costs: saving and restoring execution context, loss of locality, and CPU time spent scheduling threads instead of running them. When threads share data, they must use synchronization mechanisms that can inhibit compiler optimizations, flush or invalidate memory caches, and create synchronization traffic on the shared memory bus. All these factors introduce additional performance costs.
#### Thread Safety
Even if your program never explicitly creates a thread, frameworks may create threads on your behalf, and code called from these threads must be thread safe. This can place a significant design and implementation burden on developers, since developing thread safe classes requires more care and analysis than developing non thread safe classes.

Informally, an object's state is its data, stored in state variables such as instance or static fields. An object's state may include fields from other, dependent objects. An object's state encompasses any data that can affect its externally visible behavior. By shared, we mean that a variable could be accessed by multiple threads; by mutable, we mean that its value could change during its lifetime. We may talk about thread safety as if it were about code, but what we are really trying to do is protect data from uncontrolled concurrent access. Making an object thread safe requires using synchronization to coordinate access to its mutable state; failing to do so could result in data corruption and other undesirable consequences. The primary mechanism for synchronization in Java is the synchronized keyword, which provides exclusive locking, but the term "synchronization" also includes the use of volatile variables, explicit locks, and atomic variables. 

If multiple threads access the same mutable state variable without appropriate synchronization, your program is broken. There are three ways to fix it: 
* Don't share the state variable across threads; 
* Make the state variable immutable;
* Use synchronization whenever accessing the state variable.

A class is thread safe if it behaves correctly when accessed from multiple threads, regardless of the scheduling or interleaving of the execution of those threads by the runtime environment, and with no additional synchronization or other coordination on the part of the calling code.

The transient state for a particular computation exists solely in local variables that are stored on the thread's stack and are accessible only to the executing thread.

Stateless objects are always thread safe.

The definition of thread safety requires that invariants be preserved regardless of timing or interleaving of operations in multiple threads.
#### Atomicity
Operations A and B are atomic with respect to each other if, from the perspective of a thread executing A, when another thread executes B, either all of B has executed or none of it has. An atomic operation is one that is atomic with respect to all operations, including itself, that operate on the same state.

To ensure thread safety, check then act operations (like lazy initialization) and read modify write operations (like increment) must always be atomic.

The java.util.concurrent.atomic package contains atomic variable classes for effecting atomic state transitions on numbers and object references. By replacing the long counter with an AtomicLong, we ensure that all actions that access the counter state are atomic.

To preserve state consistency, update related state variables in a single atomic operation. 
#### Locking
A synchronized block has two parts: a reference to an object that will serve as the lock, and a block of code to be guarded by that lock. A synchronized method is shorthand for a synchronized block that spans an entire method body, and whose lock is the object on which the method is being invoked. (Static synchronized methods use the Class object for the lock.)

Every Java object can implicitly act as a lock for purposes of synchronization; these built in locks are called intrinsic locks or monitor locks. The lock is automatically acquired by the executing thread before entering a synchronized block and automatically released when control exits the synchronized block, whether by the normal control path or by throwing an exception out of the block. The only way to acquire an intrinsic lock is to enter a synchronized block or method guarded by that lock. Intrinsic locks in Java act as mutexes (or mutual exclusion locks), which means that at most one thread may own the lock. Since only one thread at a time can execute a block of code guarded by a given lock, the synchronized blocks guarded by the same lock execute atomically with respect to one another.

When a thread requests a lock that is already held by another thread, the requesting thread blocks. But because intrinsic locks are reentrant, if a thread tries to acquire a lock that it already holds, the request succeeds. Reentrancy means that locks are acquired on a per thread rather than per invocation basis. Reentrancy is implemented by associating with each lock an acquisition count and an owning thread. When the count is zero, the lock is considered unheld. When a thread acquires a previously unheld lock, the JVM records the owner and sets the acquisition count to one. If that same thread acquires the lock again, the count is incremented, and when the owning thread exits the synchronized block, the count is decremented. When the count reaches zero, the lock is released.

For each mutable state variable that may be accessed by more than one thread, all accesses to that variable must be performed with the same lock held. In this case, we say that the variable is guarded by that lock. Every shared, mutable variable should be guarded by exactly one lock. Make it clear to maintainers which lock that is. For every invariant that involves more than one variable, all the variables involved in that invariant must be guarded by the same lock.

Whenever you use locking, you should be aware of what the code in the block is doing and how likely it is to take a long time to execute. Holding a lock for a long time, either because you are doing something compute intensive or because you execute a potentially blocking operation, introduces the risk of liveness or performance problems.
#### Visibility
For example, in the absence of synchronization, the Java Memory Model permits the compiler to reorder operations and cache values in registers, and permits CPUs to reorder operations and cache values in processor specific caches.

stale data
Unless synchronization is used every time a variable is accessed, it is possible to see a stale value for that variable. Worse, staleness is not all or nothing: a thread can see an up to date value of one variable but a stale value of another variable that was written first

Intrinsic locking can be used to guarantee that one thread sees the effects of another in a predictable manner. When thread A executes a synchronized block, and subsequently thread B enters a synchronized block guarded by the same lock, the values of variables that were visible to A prior to releasing the lock are guaranteed to be visible to B upon acquiring the lock. In other words, everything A did in or prior to a synchronized block is visible to B when it executes a synchronized block guarded by the same lock. Without synchronization, there is no such guarantee.

Locking is not just about mutual exclusion; it is also about memory visibility. To ensure that all threads see the most up to date values of shared mutable variables, the reading and writing threads must synchronize on a common lock.

The Java language also provides an alternative, weaker form of synchronization, volatile variables, to ensure that updates to a variable are propagated predictably to other threads. When a field is declared volatile, the compiler and runtime are put on notice that this variable is shared and that operations on it should not be reordered with other memory operations. Volatile variables are not cached in registers or in caches where they are hidden from other processors, so a read of a volatile variable always returns the most recent write by any thread.
#### Thread confinement
This technique, thread confinement, is one of the simplest ways to achieve thread safety. When an object is confined to a thread, such usage is automatically thread safe even if the confined object itself is not

Stack confinement is a special case of thread confinement in which an object can only be reached through local variables. Just as encapsulation can make it easier to preserve invariants, local variables can make it easier to confine objects to a thread. Local variables are intrinsically confined to the executing thread; they exist on the executing thread's stack, which is not accessible to other threads.

A more formal means of maintaining thread confinement is ThreadLocal, which allows you to associate a per thread value with a value holding object. ThreadLocal provides get and set accessor methods that maintain a separate copy of the value for each thread that uses it, so a get returns the most recent value passed to set from the currently executing thread.
#### Instance Confinement
Encapsulation simplifies making classes thread-safe by promoting instance confinement, often just called confinement. When an object is encapsulated within another object, all code paths that have access to the encapsulated object are known and can be therefore analyzed more easily than if that object was accessible to the entire program. Combining confinement with an appropriate locking discipline can ensure that otherwise non-thread-safe objects are used in a thread-safe manner. Confined objects must not escape their intended scope. An object may be confined to a class instance (such as a private class member), a lexical scope (such as a local variable), or a thread (such as an object that is passed from method to method within a thread, but not supposed to be shared across threads). Objects don't escape on their own, of course - they need help from the developer, who assists by publishing the object beyond its intended scope. Confinement makes it easier to build thread-safe classes because a class that confines its state can be analyzed for thread safety without having to examine the whole program.
#### Immutable objects
Immutable objects are always thread safe.

Just as it is a good practice to make all fields private unless they need greater visibility, it is a good practice to make all fields final unless they need to be mutable.

The combination of an immutable holder object for multiple state variables related by an invariant, and a volatile reference used to ensure its timely visibility, allows the object to be thread safe even though it does no explicit locking.
#### Publish objects
To publish an object safely, both the reference to the object and the object's state must be made visible to other threads at the same time. A properly constructed object can be safely published by: 
* Initializing an object reference from a static initializer; 
* Storing a reference to it into a volatile field or AtomicReference; 
* Storing a reference to it into a final field of a properly constructed object; 
* Storing a reference to it into a field that is properly guarded by a lock;

Static initializers are executed by the JVM at class initialization time; because of internal synchronization in the JVM, this mechanism is guaranteed to safely publish any objects initialized in this way

The publication requirements for an object depend on its mutability: 
* Immutable objects can be published through any mechanism; 
* Effectively immutable objects must be safely published; 
* Mutable objects must be safely published, and must be either thread safe or guarded by a lock.

The most useful policies for using and sharing objects in a concurrent program are: 
**Thread confined:** A thread confined object is owned exclusively by and confined to one thread, and can be modified by its owning thread. 

**Shared read only:** A shared read only object can be accessed concurrently by multiple threads without additional synchronization, but cannot be modified by any thread. Shared read only objects include immutable and effectively immutable objects. 

**Shared thread safe:** A thread safe object performs synchronization internally, so multiple threads can freely access it through its public interface without further synchronization. 

**Guarded:** A guarded object can be accessed only with a specific lock held. Guarded objects include those that are encapsulated within other thread safe objects and published objects that are known to be guarded by a specific lock.
#### Designing a Thread safe Class
The design process for a thread-safe class should include these three basic elements: 
* Identify the variables that form the object's state; 
* Identify the invariants that constrain the state variables; 
* Establish a policy for managing concurrent access to the object's state.

You cannot ensure thread safety without understanding an object's invariants and post-conditions. Constraints on the valid values or state transitions for state variables can create atomicity and encapsulation requirements.
#### The Java Monitor Pattern
An object following the Java monitor pattern encapsulates all its mutable state and guards it with the object's own intrinsic lock.

#### Delegating Thread Safety
All but the most trivial objects are composite objects. The Java monitor pattern is useful when building classes from scratch or composing classes out of objects that are not thread-safe. But what if the components of our class are already thread-safe? Do we need to add an additional layer of thread safety? The answer is . . . "it depends". In some cases a composite made of thread-safe components is thread-safe, and in others it is merely a good start. The delegation examples so far delegate to a single, thread-safe state variable. We can also delegate thread safety to more than one underlying state variable as long as those underlying state variables are independent, meaning that the composite class does not impose any invariants involving the multiple state variables.

If a class has compound actions, delegation alone is again not a suitable approach for thread safety. In these cases, the class must provide its own locking to ensure that compound actions are atomic, unless the entire compound action can also be delegated to the underlying state variables.

If a state variable is thread-safe, does not participate in any invariants that constrain its value, and has no prohibited state transitions for any of its operations, then it can safely be published.

#### Client-side Locking
Client-side locking entails guarding client code that uses some object X with the lock X uses to guard its own state. In order to use client-side locking, you must know what lock X uses. The documentation for Vector and the synchronized wrapper classes states, albeit obliquely, that they support client-side locking, by using the intrinsic lock for the Vector or the wrapper collection (not the wrapped collection).