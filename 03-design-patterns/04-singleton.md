# Singleton Pattern
It is a creational pattern. It is used when we need to create an object in memory only once in an application and have it shared by multiple classes.

Rules for implementing Singleton Pattern: 
* Singletons in Java are created as private static variables within the class, often with the name instance.
* They are accessed via a single public static method, often named getInstance(), which returns the reference to the singleton object.
* Finally, all constructors in a singleton class are marked private, which ensures that no other class is capable of instantiating another version of the class. By marking the constructors private, we have implicitly marked the class final. 
```java
public class HayStorage {
   private int quantity = 0;
   private HayStorage() {}
  
   private static final HayStorage instance = new HayStorage();
  
   public static HayStorage getInstance() { return instance; }
  
   public synchronized void addHay(int amount) { quantity += amount; }
   public synchronized boolean removeHay (int amount) {
      if(quantity < amount) return false;
      quantity -= amount;
      return true;
   }
  
   public synchronized int getHayQuantity() { return quantity; }
}
```
You can create a singleton instance in few ways:
* instantiate the singleton object directly in the definition of the instance reference(previous example);
* using a static initialization block when the class is loaded:
```java
// Instantiation using a static block
public class StaffRegister {
   private static final StaffRegister instance;
   static {
      instance = new StaffRegister();
      // Perform additional steps
   }

   private StaffRegister() { }

   public static StaffRegister getInstance() { return instance; }
}
```
* create a singleton when the getInstance() method is called (lazy loading):
```java
// Lazy instantiation
public class VisitorTicketTracker {
   private static VisitorTicketTracker instance;
   private VisitorTicketTracker() { }

   public static synchronized VisitorTicketTracker getInstance() {
      if(instance == null) {
         instance = new VisitorTicketTracker();
      }
      return instance;
   }
}
```
Lazy instantiation reduces memory usage and improves performance when an application starts up.