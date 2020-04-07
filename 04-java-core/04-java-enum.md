# Enum
* [Working with Enums](#working-with-enums)
* [Adding Constructors, Fields and Methods](#adding-constructors-fields-and-methods)
* [Summary](#summary)
#### Working with Enums
* An enumeration is like a fixed set of constants.
* To create an enum:
```java
public enum Season {
   WINTER, SPRING, SUMMER, FALL
}
```
* An enum provides a method to get an array of all of the values. You can use this like any normal array, including in a loop:
```
for(Season season: Season.values()) {
   System.out.println(season.name() + " " + season.ordinal());
}
```
The output shows that each enum value has a corresponding int value in the order in which they are declared. The int value will remain the same during your program, but the program is easier to read if you stick to the human‐readable enum value.
* You can also create an enum from a String. This is helpful when working with older code. The String passed in must match exactly, though.
```java
Season s1 = Season.valueOf("SUMMER"); // SUMMER
Season s2 = Season.valueOf("summer"); // exception
```
* You can not extend enum.
#### Adding Constructors, Fields and Methods
```java
public enum Season {
   WINTER("Low"), SPRING("Medium"), SUMMER("High"), FALL("Medium");
   private String expectedVisitors;
   private Season(String expectedVisitors) {
      this.expectedVisitors = expectedVisitors;
   }
   public void printExpectedVisitors() {
      System.out.println(expectedVisitors);
   }
}
```
* On line 2 we have a semicolon, it is required if there is anything in the enum besides the values.
* The constructor is private because it can only be called from the enum. Constructor is ran only once when the enum is requested for the first time.
* We can call an enum method like that: *Season.SUMMER.printExpectedVisitors()*;
```java
public enum Season {
   WINTER {
      public void printHours() { System.out.println("9am-3pm"); }
   }, SPRING {
      public void printHours() { System.out.println("9am-5pm"); }
   }, SUMMER {
      public void printHours() { System.out.println("9am-7pm"); }
   }, FALL {
      public void printHours() { System.out.println("9am-5pm"); }
   };
   public abstract void printHours();
}
```
* The enum has an abstract method. This means that each and every enum value is required to implement this method. Or we can create a default implementation and override it only for a special cases.
#### Summary
* Enum constructor is always private. You cannot make it public or protected. If an enum type has no constructor declarations, then a private constructor that takes no parameters is automatically provided.
* An enum is implicitly final, which means you cannot extend it. 
* You cannot extend an enum from another enum or class because an enum implicitly extends java.lang.Enum. But an enum can implement interfaces.
* Since enum maintains exactly one instance of its constants, you cannot clone it. You cannot even override the clone method in an enum because java.lang.Enum makes it final.
* Compiler provides an enum with two public static methods automatically - values() and valueOf(String). The values method returns an array of its constants and valueOf method tries to match the String argument exactly (i.e. case sensitive) with an enum constant and returns that constant if successful otherwise it throws java.lang.IllegalArgumentException.
* It implements java.lang.Comparable (thus, an enum can be added to sorted collections such as SortedSet, TreeSet, and TreeMap).
* It has a method ordinal(), which returns the index (starting with 0) of that constant i.e. the position of that constant in its enum declaration.
* It has a method name(), which returns the name of this enum constant, exactly as declared in its enum declaration.
* A public (or non-public) enum can be defined inside any class.
* An enum can be defined as a static member of any class. You can also have multiple public enums with in the same class.
* An enum cannot be defined inside any method or constructor.
* Unlike a regular java class, you cannot access a non-final static field from an enum's constructor.