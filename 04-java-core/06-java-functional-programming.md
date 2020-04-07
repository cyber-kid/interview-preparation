# Functional Programming
* [Using Variables in Lambdas](#using-variables-in-lambdas)
* [Working with Built-In Functional Interfaces](#working-with-built-in-functional-interfaces)
* [Implementing Supplier](#implementing-supplier)
* [Implementing Consumer and BiConsumer](#implementing-consumer-and-biconsumer)
* [Implementing Predicate and BiPredicate](#implementing-predicate-and-bipredicate)
* [Implementing Function and BiFunction](#implementing-function-and-bifunction)
* [Implementing UnaryOperator and BinaryOperator](#implementing-unaryoperator-and-binaryoperator)
* [Returning an Optional](#returning-an-optional)
* [Summary](#summary)
#### Using Variables in Lambdas
Lambda expressions can access static variables, instance variables, effectively final method parameters, and effectively final local variables. Example:
```java
interface Gorilla { String move(); }

class GorillaFamily {
   String walk = "walk";
   void everyonePlay(boolean baby) {
      String approach = "amble";
      //approach = "run";

      play(() -> walk);
      play(() -> baby ? "hitch a ride": "run");
      play(() -> approach);
   }
   void play(Gorilla g) {
      System.out.println(g.move());
   }
}
```
The normal rules for access control still apply. For example, a lambda can’t access private variables in another class. Remember that lambdas can access a subset of variables that are accessible, but never more than that.

#### Working with Built-In Functional Interfaces
A functional interface has exactly one abstract method. All of the functional interfaces were introduced in Java 8 and are provided in the **java.util.function** package. The convention here is to use the generic type T for type parameter. If a second type parameter is needed, the next letter, U, is used. If a distinct return type is needed, R for return is used for the generic type.

Types of Functional Interfaces:
* **Supplier< T >**
* **Consumer< T >**
* **BiConsumer<T, U>**
* **Predicate< T >**
* **BiPredicate<T, U>**
* **Function<T, R>**
* **BiFunction<T, U, R>**
* **UnaryOperator< T >**
* **BinaryOperator< T >**
#### Implementing Supplier
A Supplier is used when you want to generate or supply values without taking any input. The Supplier interface is defined as:
```java
@FunctionalInterface
public class Supplier<T> {
   public T get();
}
```
A Supplier is often used when constructing new objects.
#### Implementing Consumer and BiConsumer
You use a Consumer when you want to do something with a parameter but not return anything. BiConsumer does the same thing except that it takes two parameters. Omitting the default methods, the interfaces are defined as follows:
```java
@FunctionalInterface
public class Consumer<T> {
   void accept(T t);
}
@FunctionalInterface
public class BiConsumer<T, U> {
   void accept(T t, U u);
}
```
#### Implementing Predicate and BiPredicate
Predicate is often used when filtering or matching. It takes one parameter and returns a boolean. Both are very common operations. A BiPredicate is just like a Predicate except that it takes two parameters instead of one. Omitting any default or static methods, the interfaces are defined as follows:
```java
@FunctionalInterface
public class Predicate<T> {
   boolean test(T t);
}
@FunctionalInterface
public class BiPredicate<T, U> {
   boolean test(T t, U u);
}
```
#### Implementing Function and BiFunction
A Function is responsible for turning one parameter into a value of a potentially different type and returning it. Similarly, a BiFunction is responsible for turning two parameters into a value and returning it. Omitting any default or static methods, the interfaces are defined as the following:
```java
@FunctionalInterface
public class Function<T, R> {
   R apply(T t);
}
@FunctionalInterface
public class BiFunction<T, U, R> {
   R apply(T t, U u);
}
```
#### Implementing UnaryOperator and BinaryOperator
UnaryOperator and BinaryOperator are a special case of a function. They require all type parameters to be the same type. A UnaryOperator transforms its value into one of the same type. For example, incrementing by one is a unary operation. In fact, UnaryOperator extends Function. A BinaryOperator merges two values into one of the same type. Adding two numbers is a binary operation. Similarly, BinaryOperator extends BiFunction. Omitting any default or static methods, the interfaces are defined as follows:
```java
@FunctionalInterface
public class UnaryOperator<T> extends Function<T, T> { }
@FunctionalInterface
public class BinaryOperator<T> extends BiFunction<T, T, T> { }
```
This means that method signatures look like this:
```java
T apply(T t);
T apply(T t1, T t2);
```
If you look at the Javadoc, you’ll notice that these methods are actually declared on the Function/BiFunction superclass. The generic declarations on the subclass are what force the type to be the same.
#### Returning an Optional
How do we express this “we don’t know” or “not applicable” answer in Java? Starting with Java 8, we use the Optional type. An Optional is created using a factory. You can either request an empty Optional or pass a value for the Optional to wrap. Think of an Optional as a box that might have something in it or might instead be empty. Here’s how to code our average method:
```java
public static Optional<Double> average(int… scores) {
   if (scores.length == 0) return Optional.empty();
   int sum = 0;
   for (int score: scores) sum += score;
   return Optional.of((double) sum / scores.length);
}
```
You can see that one Optional contains a value and the other is empty. Normally, we want to check if a value is there and/or get it out of the box. Here’s one way to do that:
```java
Optional<Double> opt = average(90, 100);
if (opt.isPresent()) System.out.println(opt.get()); // 95.0
```
What if we didn’t do the check and the Optional was empty? We’d get an exception since there is no value inside the Optional: java.util.NoSuchElementException: No value present

If value is null, o is assigned the empty Optional. Otherwise, we wrap the value. Since this is such a common pattern, Java provides a factory method to do the same thing:
```java
Optional o = Optional.ofNullable(value);
```
Other instance methods:

Method | When Optional is empty | When Optional contains a value
------ | ---------------------- | ------------------------------
get() | Throws an exception | Returns value
ifPresent(Consumer c) | Does nothing | Calls Consumer c with value
isPresent() | Returns false | Returns true
orElse(T other) | Returns other parameter | Returns value
orElseGet(Supplier s) | Returns result of calling Supplier | Returns value
orElseThrow(Supplier s) | Throws exception created by calling Supplier | Returns value
#### Summary:
* Optional has a static method named of(T t) that returns an Optional object containing the value passed as argument. It will throw NullPointerException if you pass null. If you want to avoid NullPointerException, you should use Optional.ofNullable(T t) method. This will return Optional.empty if you pass null.
* You cannot change the contents of Optional object after creation. Optional does not have a set method.
* The orElse method returns the actual object contained inside the Optional or the argument passed to this method if the Optional is empty. It does not return an Optional object.
* isPresent() returns true if the Optional contains a value, false otherwise.
* ifPresent(Consumer ) executes the Consumer object with the value if the Optional contains a value. Not that it is the value contained in the Optional that is passed to the Consumer and not the Optional itself.