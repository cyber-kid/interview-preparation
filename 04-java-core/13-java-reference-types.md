# Java Reference Types
* [Strong references](#strong-references)
* [Soft references](#soft-references)
* [Weak References](#weak-references)
* [PhantomReference](#phantom-reference)
* [ReferenceQueue](#referencequeue)

There are actually 4 kinds of reference types in Java.
* **Strong references**
* **Soft references**
* **Weak references**
* **Phantom references**

These references are different solely because of the existence of a garbage collection mechanism in the JVM. That is because, the decision of reclaiming memory from the object heap depends not only on the fact that there are active references to an object, but also on the type of reference to the object.

#### Strong references
Any object with an active strong reference to it, will never be garbage collected(except in rare circumstances, such as cyclical references).

#### Soft references
The soft reference class is used to create a soft reference to an existing object that is already referred to by a strong reference.
```java
Employee emp = new Employee();
SoftReference<Employee> softRef = new SoftReference<employee>(emp);
```
Let us assume that at a later point of time, we nullify the 'emp' reference as follows and no new strong references were created to this emp object. Now let`s assume that the garbage collector decides to run at this point of time. What it will see is that the current employee object only has a soft reference pointing to it, and no strong references. The garbage collector may optionally choose to reclaim the memory occupied by the employee object. But what makes soft references even more special is the fact that the garbage collector will always reclaim the memory of objects that have only soft references pointing to them before it throws an OutOfMemory Error. This gives the garbage collector some sort of a last chance to save the life of the JVM. At any given point of time, if the garbage collector has not yet reclaimed the memory of the actual referent object, you can get a strong
reference to the referent via the get method.
```java
Employee resurrectedEmp = softRef.get();
```

#### Weak References
The garbage collector has no regard for an object that only has a weak reference. What that means is that the garbage collector will make it eligible for garbage collection because object only has a weak reference pointing to it.

To create a weak reference use:
```java
DBConnection emp = new Employee();
WeakReference<DBConnection> weakRef = new WeakReference<DBConnection>(emp);
```
Weak references are primarily used in conjunction with a WeakHashMap. This is a special kind of hashmap where the keys are all made of weak references. So, in our database example, we could effectively create a weak reference of the Database connection and store it in the WeakHashMap and the metadata of the user as the value in the hashmap. In this way, when the application no longer holds a strong reference to the Database connection, the only reference to the database connection object will be the one that we have via the WeakReference entry in the WeakHashMap. When the garbage collector detects this, it will mark the object for garbage collection. When the object is garbage collected, the entry from the WeakHashMap will be removed.
#### Phantom Reference
Method **get()** of this type of references always returns **null**. Exactly because of this PhantomReference is used with ReferenceQueue. GC will put this PhantomReference in ReferenceQueue after the **finalize()** method is executed. So basically the object is still in the memory.
#### ReferenceQueue
In order to associate a weak or a soft reference with a reference queue, you can use the 2 argument constructor as shown below.
```
Employee emp1 = new Employee();
Employee emp2 = new Employee();
ReferenceQueue softQueue = new ReferenceQueue();
ReferenceQueue weakQueue = new ReferenceQueue();
SoftReference softRef = new SoftReference(emp1, softQueue);
WeakReference softRef = new WeakReference(emp2, weakQueue)
```
