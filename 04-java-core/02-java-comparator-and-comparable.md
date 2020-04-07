# Comparator and Comparable
* [Comparable](#comparable)
* [Comparator](#comparator)
* [Searching and Sorting](#searching-and-sorting)

You can also sort objects that you create. Java provides an interface called **Comparable**. If your class implements **Comparable**, it can be used in these data structures that require comparison. There is also a class called **Comparator**, which is used to specify that you want to use a different order than the object itself provides.

#### Comparable
The **java.lang.Comparable** interface has only one method:
```java
public interface Comparable<T> {
   public int compareTo(T o);
}
```
Method **compareTo** returns int:
* The **number zero** is returned when the current object is equal to the argument to compareTo().
* A **number less than zero** is returned when the current object is smaller than the argument to compareTo().
* A **number greater than zero** is returned when the current object is larger than the argument to compareTo().

If you write a class that implements Comparable, you introduce new business logic for determining equality. The compareTo() method returns 0 if two objects are equal, while your equals() method returns true if two objects are equal. A natural ordering that uses compareTo() is said to be consistent with equals if, and only if, x.equals(y) is true whenever x.compareTo(y) equals 0. You are strongly encouraged to make your Comparable classes consistent with equals because not all collection classes behave predictably if the compareTo() and equals() methods are not consistent.
#### Comparator
Sometimes you want to sort an object that did not implement Comparable, or you want to sort objects in different ways at different times.
```java
package java.util;
public interface Comparator<T> {
   int compare(T o1, T o2);
}
```
Method returns the same result as compareTo.

You can chain comparators.
Example:
```java
public class ChainingComparator implements Comparator<Squirrel> {
   public int compare(Squirrel s1, Squirrel s2) {
      Comparator<Squirrel> c = Comparator.comparing(s -> s.getSpecies());
      c = c.thenComparingInt(s -> s.getWeight());
      return c.compare(s1, s2);
   }
}
```
#### Searching and Sorting
The sort method uses the compareTo() method to sort. It expects the objects to be sorted to be Comparable.

To sort a collection you should use:
```java
Collections.sort(List<T> list);
```
You can pass a Comparator to this method.

To search a collection you should use:
```
Collections.binarySearch(List<T> list, T key);
```
You can pass a Comparator to this method.