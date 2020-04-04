# Generics
* [Generics Classes](#generics-classes)
* [Generic Interfaces](#generic-interfaces)
* [Generic Methods](#generic-methods)
* [Bound Wildcards](#bound-wildcards)
#### Generics Classes
You can introduce generics into your own classes. The syntax for introducing a generic is to declare a formal type parameter in angle brackets. For example, the following class named Crate has a generic type variable declared after the name of the class:
```java
public class Crate<T> {
   private T contents;
   public T emptyCrate() {
      return contents;
   }
   public void packCrate(T contents) {
      this.contents = contents;
   }
}
```
The generic type T is available anywhere within the Crate class. When you instantiate the class, you tell the compiler what T should be for that particular instance:
```java
Elephant elephant = new Elephant();
Crate<Elephant> crateForElephant = new Crate<>();
```
Generic classes aren’t limited to having a single type parameter.

Type Erasure: class **Crate< Robot >** specifying the generic type of **Crate** as **Robot** is like replacing the **T** in the **Crate** class with **Robot**. However, this is just for compile time. Behind the scenes, the compiler replaces all references to **T** in **Crate** with **Object**. In other words, after the code compiles, your generics are actually just **Object** types.

#### Generic Interfaces
Just like a class, an interface can declare a formal type parameter. For example, the following Shippable interface uses a generic type as the argument to its ship() method:
```
public interface Shippable<T> {
   void ship(T t);
}
```
There are 3 ways a class can approach implementing this interface:
* Specify the generic type in the class:
```java
class ShippableRobotCrate implements Shippable<Robot> {
         public void ship(Robot t) { }
      }
```
* The next way is to create a generic class. The following concrete class allows the caller to specify the type of the generic:
```java
class ShippableAbstractCrate<U> implements Shippable<U> {
   public void ship(U t) { }
}
```
* The final way is to not use generics at all. This is the old way of writing code. It generates a compiler warning about Shippable being a raw type, but it does compile:
```java
class ShippableCrate implements Shippable {
   public void ship(Object t) { }
}
```
Here are the things that you can’t do with generics. (And by “can’t,” we mean without resorting to contortions like passing in a class object.)
* Call the constructor. new T() is not allowed because at runtime it would be new Object().
* Create an array of that static type. This one is the most annoying, but it makes sense because you’d be creating an array of Objects.
* Call instanceof. This is not allowed because at runtime List< Integer > and List< String > look the same to Java thanks to type erasure.
* Use a primitive type as a generic type parameter. This isn’t a big deal because you can use the wrapper class instead. If you want a type of int, just use Integer.
* Create a static variable as a generic type parameter. This is not allowed because the type is linked to the instance of the class.
#### Generic Methods
Unless a method is obtaining the generic formal type parameter from the class/interface, it is specified immediately before the return type of the method.
```java
public static <T> void sink(T t) { }
public static <T> T identity(T t) { return t; }
```
#### Bound Wildcards
A wildcard generic type is an unknown generic type represented with a question mark (?). You can use generic wildcards in three ways:
1. **Unbounded wildcard (<?>)**. It represents any data type.
2. **Upper-bounded wildcard (<? extends Number>)**. It represents any class that extends Number or Number itself.
3. **Lower-bounded wildcard (<? super String>)**. It represents list of String objects or a list of some objects that are a superclass of String.

Something interesting happens when we work with upper bounds or unbounded wildcards. The list becomes logically immutable.
The wildcard is not allowed to be on the right side of an assignment.