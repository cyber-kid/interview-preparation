# Nested Classes
* [Creating Nested Classes](#creating-nested-classes)
* [Member Inner Classes](#member-inner-classes)
* [Local Inner Classes](#local-inner-classes)
* [Anonymous Inner Classes](#anonymous-inner-classes)
* [Static Nested Classes](#static-nested-classes)
#### Creating Nested Classes
A nested class is a class that is defined within another class. A nested class that is not static is called an inner class.

There are four of types of nested classes:
1. A **member inner class** is a class defined at the same level as instance variables. It is not static. Often, this is just referred to as an inner class without explicitly saying the type.
2. A **local inner class** is defined within a method.
3. An **anonymous inner class** is a special case of a local inner class that does not have a name.
4. A **static nested class** is a static class that is defined at the same level as static variables.
#### Member Inner Classes
A member inner class is defined at the member level of a class (the same level as the methods, instance variables, and constructors). Member inner classes have the following properties:
* Can be declared public, private, or protected or use default access
* Can extend any class and implement interfaces
* Can be abstract or final
* Cannot declare static fields or methods
* Can access members of the outer class including private members

A member inner class declaration looks just like a stand‐alone class declaration except that it happens to be located inside another class, and that it can use the instance variables declared in the outer class:
```java
public class Outer {
   private String greeting = "Hi";
    protected class Inner {
      public int repeat = 3;
      public void go() {
          for (int i = 0; i < repeat; i++) System.out.println(greeting);
      }
    }
    public void callInner() {
      Inner inner = new Inner();
      inner.go();
    }
    public static void main(String[] args) {
      Outer outer = new Outer();
      outer.callInner();
   }
}
```
The other way to create an instance of inner class is:
```java
public static void main(String[] args) {
  Outer outer = new Outer();
  Inner inner = outer.new Inner(); // create the inner class
  inner.go();
}
```
Inner classes can have the same variable names as outer classes. There is a special way of calling this to say which class you want to access:
```java
System.out.println(x);
System.out.println(this.x);
System.out.println(B.this.x);
System.out.println(A.this.x);
```
#### Local Inner Classes
A local inner class is a nested class defined within a method. Like local variables, a local inner class declaration does not exist until the method is invoked, and it goes out of scope when the method returns. This means that you can create instances only from within the method. Those instances can still be returned from the method. This is just how local variables work. Local inner classes have the following properties:
* They do not have an access specifier.
* They cannot be declared static and cannot declare static fields or methods.
* They have access to all fields and methods of the enclosing class.
* They do not have access to local variables of a method unless those variables are final or effectively final. If the code could still compile with the keyword final inserted before the local variable, the variable is effectively final.
```java
public class Outer {
   private int length = 5;
   public void calculate() {
      final int width = 20;
      class Inner {
          public void multiply() {
            System.out.println(length * width);
          }
      }
      Inner inner = new Inner();
      inner.multiply();
   }
   public static void main(String[] args) {
    Outer outer = new Outer();
    outer.calculate();
  }
}
```
#### Anonymous Inner Classes
An anonymous inner class is a local inner class that does not have a name. It is declared and instantiated all in one statement using the new keyword. Anonymous inner classes are required to extend an existing class or implement an existing interface. They are useful when you have a short implementation that will not be used anywhere else. Here’s an example:
```java
public class AnonInner {
    abstract class SaleTodayOnly {
      abstract int dollarsOff();
   }
    public int admission(int basePrice) {
      SaleTodayOnly sale = new SaleTodayOnly() {
          int dollarsOff() { return 3;}
      };
      return basePrice - sale.dollarsOff();
   }
}
```
To remember:
- Anonymous inner classes can have initialization parameters if they are created for classes.
- Anonymous inner classes cannot be static.

#### Static Nested Classes
A static nested class is a static class defined at the member level. The modifier static pertains only to member classes, not to top level or local or anonymous classes. That is, only classes declared as members of top-level classes can be declared static. Package member classes, local classes (i.e. classes declared in methods) and anonymous classes cannot be declared static. It can be instantiated without an object of the enclosing class, so it can’t access the instance variables without an explicit object of the enclosing class. In other words, it is like a regular class except for the following:
* The nesting creates a namespace because the enclosing class name must be used to refer to it.
* It can be made private or use one of the other access modifiers to encapsulate it.
* The enclosing class can refer to the fields and methods of the static nested class.
```java
public class Enclosing {
   static class Nested {
      private int price = 6;
   }
   public static void main(String[] args) {
      Nested nested = new Nested();
      System.out.println(nested.price);
   }
}
```