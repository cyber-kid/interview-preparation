# Using Streams
* [Creating Stream Source](#creating-stream-source)
* [To create an infinite stream](#to-create-an-infinite-stream)
* [Using Common Terminal Operations](#using-common-terminal-operations)
* [Using Common Intermediate Operations](#using-common-intermediate-operations)
* [Creating Primitive Streams](#creating-primitive-streams)
* [Using Optional with Primitive Streams](#using-optional-with-primitive-streams)
* [Summarizing Statistics](#summarizing-statistics)
* [Working with Advanced Stream Pipeline Concepts](#working-with-advanced-streampipeline-concepts)

A stream in Java is a sequence of data. A stream pipeline is the operations that run on a stream to produce a result. Think of a stream pipeline as an assembly line in a factory. With a list, you can access any element at any time. With a queue, you are limited in which elements you can access, but all of the elements are there. With streams, the data isn’t generated up front—it is created when needed. Operations occur in a pipeline and they are called stream operations. There are three parts to a stream pipeline:
* **Source**: Where the stream comes from.
* **Intermediate operations**: Transforms the stream into another one. There can be as few or as many intermediate operations as you’d like. Since streams use lazy evaluation, the intermediate operations do not run until the terminal operation runs.
* **Terminal operation**: Actually produces a result. Since streams can be used only once, the stream is no longer valid after a terminal operation completes.
#### Creating Stream Source
In Java, the Stream interface is in the **java.util.stream** package. There are a few ways to
create a finite stream:
```java
Stream<String> empty = Stream.empty(); // count = 0
Stream<Integer> singleElement = Stream.of(1); // count = 1
Stream<Integer> fromArray = Stream.of(1, 2, 3); // count = 2
```
Since streams are new in Java 8, most code that’s already written uses lists. Java provides a convenient way to convert from a list to a stream:
```java
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> fromList = list.stream();
Stream<String> fromListParallel = list.parallelStream();
```
#### To create an infinite stream:
```java
Stream<Double> randoms = Stream.generate(Math::random); //generate the infinite amount of random numbers
Stream<Integer> oddNumbers = Stream.iterate(1, n -> n + 2); //One is the first element that will be part of the stream. The other parameter is a lambda expression that gets passed the previous value and generates the next value.
```
#### Using Common Terminal Operations
You can perform a terminal operation without any intermediate operations but not the other way around. This is why we will talk about terminal operations first. Reductions are a special type of terminal operation where all of the contents of the stream are combined into a single primitive or Object . For example, you might have an int or a Collection.

**count()** The count() method determines the number of elements in a finite stream. For an infinite stream, it hangs. count() is a reduction because it looks at each element in the stream and returns a single value. The method signature is this: long count()

**min() and max()** The min() and max() methods allow you to pass a custom comparator and find the smallest or largest value in a finite stream according to that sort order. Like count(), min() and max() hang on an infinite stream because they cannot be sure that a smaller or larger value isn’t coming later in the stream. Both methods are reductions because they return a single value after looking at the entire stream. The method signatures are as follows: Optional<T> min(<? super T> comparator); Optional<T> max(<? super T> comparator); Notice that the code returns an Optional rather than the value. This allows the method to specify that no minimum or maximum was found.

**findAny() and findFirst()** The findAny() and findFirst() methods return an element of the stream unless the stream is empty. If the stream is empty, they return an empty Optional. This is the first method you’ve seen that works with an infinite stream. Since Java generates only the amount of stream you need, the infinite stream needs to generate only one element. findAny() is useful when you are working with a parallel stream. It gives Java the flexibility to return to you the first element it comes by rather than the one that needs to be first in the stream based on the intermediate operations. However, findAny is deliberately designed to be non-deterministic. Its API specifically says that it may return any element from the stream. These methods are terminal operations but not reductions. The reason is that they sometimes return without processing all of the elements. This means that they return a value based on the stream but do not reduce the entire stream into one value. The method signatures are these: Optional<T> findAny(); Optional<T> findFirst();

**allMatch(), anyMatch() and noneMatch()** The allMatch() , anyMatch() and noneMatch() methods search a stream and return information about how the stream pertains to the predicate. These may or may not terminate for infinite streams. It depends on the data. Like the find methods, they are not reductions because they do not necessarily look at all of the elements. The method signatures are as follows: boolean anyMatch(Predicate <? super T> predicate); boolean allMatch(Predicate <? super T> predicate); boolean noneMatch(Predicate <? super T> predicate);

**forEach() and forEachOrdered()** A looping construct is available. As expected, calling forEach() on an infinite stream does not terminate. Since there is no return value, it is not a reduction. Before you use it, consider if another approach would be better. Developers who learned to write loops first tend to use them for everything. For example, a loop with an if statement should be a filter instead. The method signature is the following: void forEach(Consumer<? super T> action); Notice that this is the only terminal operation with a return type of void . If you want something to happen, you have to make it happen in the loop. While forEach() sounds like a loop, it is really a terminal operator for streams. Streams cannot use a traditional for loop to run because they don’t implement the Iterable interface.  forEachOrdered() method processes the elements of the stream in the order they are present in the underlying source.

**reduce()** The reduce() method combines a stream into a single object. As you can tell from the name, it is a reduction. The method signatures are these: 
```java
T reduce(T identity, BinaryOperator<T> accumulator);
Optional<T> reduce(BinaryOperator<T> accumulator);
<U> U reduce(U identity, BiFunction<U,? super T,U> accumulator, BinaryOperator<U> combiner);
```
**collect()** The collect() method is a special type of reduction called a mutable reduction. It is more efficient than a regular reduction because we use the same mutable object while accumulating. Common mutable objects include StringBuilder and ArrayList. This is a really useful method, because it lets us get data out of streams and into another form. The method signatures are as follows:
```java
<R> R collect(Supplier<R> supplier, BiConsumer<R, ? super T> accumulator, BiConsumer<R, R> combiner);
<R,A> R collect(Collector<? super T, A,R> collector);
```
Example:
```java
Stream<String> stream = Stream.of("w", "o", "l", "f");
StringBuilder word = stream.collect(StringBuilder::new, StringBuilder::append, StringBuilder:append);
```
The first parameter is a Supplier that creates the object that will store the results as we collect data. Remember that a Supplier doesn’t take any parameters and returns a value. In this case, it constructs a new StringBuilder. The second parameter is a BiConsumer, which takes two parameters and doesn’t return anything. It is responsible for adding one more element to the data collection. In this example, it appends the next String to the StringBuilder. The final parameter is another BiConsumer. It is responsible for taking two data collections and merging them. This is useful when we are processing in parallel. Two smaller collections are formed and then merged into one. This would work with StringBuilder only if we didn’t care about the order of the letters. In this case, the accumulator and combiner have similar logic.

Now let’s look at an example where the logic is different in the accumulator and combiner:
```java
Stream<String> stream = Stream.of("w", "o", "l", "f");
TreeSet<String> set = stream.collect(TreeSet::new, TreeSet::add, TreeSet::addAll);
System.out.println(set); // [f, l, o, w]
```
The collector has three parts as before. The supplier creates an empty TreeSet. The accumulator adds a single String from the Stream to the TreeSet. The combiner adds all of the elements of one TreeSet to another in case the operations were done in parallel and need to be merged.

In practice, there are many common collectors that come up over and over. Rather than making developers keep reimplementing the same ones, Java provides an interface with common collectors. This approach also makes the code easier to read because it is more expressive. For example, we could rewrite the previous example as follows:
```java
Stream<String> stream = Stream.of("w", "o", "l", "f");
TreeSet<String> set = stream.collect(Collectors.toCollection(TreeSet::new));
System.out.println(set); // [f, l, o, w]
```
If we didn’t need the set to be sorted, we could make the code even shorter:
```java
Stream<String> stream = Stream.of("w", "o", "l", "f");
Set<String> set = stream.collect(Collectors.toSet());
System.out.println(set); // [f, w, l, o]
```
You might get different output for this last one since toSet() makes no guarantees as to which implementation of Set you’ll get. It is likely to be a HashSet , but you shouldn’t expect or rely on that.
#### Using Common Intermediate Operations
Unlike a terminal operation, intermediate operations deal with infinite streams simply by returning an infinite stream. Since elements are produced only as needed, this works fine.

**filter()** The filter() method returns a Stream with elements that match a given expression. Here is the method signature: Stream<T> filter(Predicate<? super T> predicate);

**distinct()** The distinct() method returns a stream with duplicate values removed. The duplicates do not need to be adjacent to be removed. As you might imagine, Java calls equals() to determine whether the objects are the same. The method signature is as follows: Stream<T> distinct();

**limit() and skip()** The limit() and skip() methods make a Stream smaller. They could make a finite stream smaller, or they could make a finite stream out of an infinite stream. The method signatures are shown here: Stream<T> limit(int maxSize); Stream<T> skip(int n);
The following code creates an infinite stream of numbers counting from 1. The skip() operation returns an infinite stream starting with the numbers counting from 6, since it skips the first five elements. The limit() call takes the first two of those. Now we have a finite stream with two elements:
```java
Stream<Integer> s = Stream.iterate(1, n -> n + 1);
s.skip(5).limit(2).forEach(System.out::print); // 67
```
**map()** The map() method creates a one-to-one mapping from the elements in the stream to the elements of the next step in the stream. The method signature is as follows: <R> Stream<R> map(Function<? super T, ? extends R> mapper); This one looks more complicated than the others you have seen. It uses the lambda expression to figure out the type passed to that function and the one returned. The return type is the stream that gets returned.

**flatMap()** The objective of flatMap is to take each element of the current stream and replace that element with elements contained in the stream returned by the Function that is passed as an argument to flatMap. flatMap is used when each element of a given stream can itself generate a Stream of objects. The purpose of this method is to extract the elements of each of those individual streams and return a stream that contains all those elements.

**sorted()** The sorted() method returns a stream with the elements sorted. Just like sorting arrays, Java uses natural ordering unless we specify a comparator. The method signatures are these: Stream<T> sorted(); Stream<T> sorted(Comparator<? super T> comparator);

**peek()** It is useful for debugging because it allows us to perform a stream operation without actually changing the stream. The method signature is as follows: Stream<T> peek(Consumer<? super T> action); The most common use for peek() is to output the contents of the stream as it goes by.

**boxed()** Is used in primitive streams. Converts all values in the stream to be boxed to the wrapers.
#### Creating Primitive Streams
Here are three types of primitive streams:
* **IntStream**: Used for the primitive types int, short, byte, and char
* **LongStream**: Used for the primitive type long
* **DoubleStream**: Used for the primitive types double and float

Some of the methods for creating a primitive stream are equivalent to how we created the source for a regular Stream:
* You can create an **empty** stream using **empty()** method.
* Another way is to use the **of()** factory method from a single value or by using the varargs overload.
* You can also use the two methods (**generate(Supplier<T> s)**, **iterate(final T seed, final UnaryOperator<T> f)**) for creating infinite streams

The **Random** class provides a method to get primitives streams of random numbers directly. For example, **ints()** generates an infinite stream of int primitives.

Java provides a method that can generate a range of numbers:
```java
IntStream range = IntStream.range(1, 6); //Number 6 is not included. 
```
The other one is:
```java
IntStream range = IntStream.rangeClosed(1, 6); //Number 6 is included.
```

The final way to create a primitive stream is by mapping from another stream type.

Source Stream Class | To create Stream | To create DoubleStream | To create IntStream | To create LongStream
------------------- | ---------------- | ---------------------- | ------------------- | --------------------
Stream | map | mapToDouble | mapToInt | mapToObj
DoubleStream | mapToObj | map | mapToInt | mapToObj
IntStream | mapToObj | mapToDouble | map | mapToObj
LongStream | mapToObj | mapToDouble | mapToInt | map

Obviously, they have to be compatible types for this to work. Java requires a mapping function to be provided as a parameter, for example:
```java
Stream<String> objStream = Stream.of("penguin", "fish");
IntStream intStream = objStream.mapToInt(s -> s.length())
```
Here are examples of function parameters when mapping between types of streams:

Source Stream Class | To create Stream | To create DoubleStream | To create IntStream | To create LongStream
------------------- | ---------------- | ---------------------- | ------------------- | --------------------
Stream | Function | ToDoubleFunction | ToIntFunction | ToLongFunction
DoubleStream | DoubleFunction | DoubleUnaryOperator | DoubleToIntFunction | DoubleToLongFunction
IntStream | IntFunction | IntToDoubleFunction | IntUnaryOperator | IntToLongFunction
LongStream | LongFunction | LongToDoubleFunction | LongToIntFunction | LongUnaryOperator
#### Using Optional with Primitive Streams
Optional types for primitives

Operation | OptionalDouble | OptionalInt | OptionalLong
--------- |-------------- | ----------- | ------------
Getting as a primitive | getAsDouble() | getAsInt() | getAsLong()
orElseGet() parameter type | DoubleSupplier | IntSupplier |LongSupplier
Return type of max() | OptionalDouble | OptionalInt | OptionalLong
Return type of sum() | double | int | long
Return type of avg() | OptionalDouble | OptionalDouble | OptionalDouble
#### Summarizing Statistics
```java
private static int range(IntStream ints) {
   IntSummaryStatistics stats = ints.summaryStatistics();
   if (stats.getCount() == 0) throw new RuntimeException();
   return stats.getMax()—stats.getMin();
}
```
Here we asked Java to perform many calculations about the stream. This includes the minimum, maximum, average, size, and the number of values in the stream. If the stream were empty, we’d have a count of zero. Otherwise, we can get the minimum and maximum out of the summary.
#### Working with Advanced Stream Pipeline Concepts
**Linking Streams to the Underlying Data**
```java
List<String> cats = new ArrayList<>();
cats.add("Annie");
cats.add("Ripley");
Stream<String> stream = cats.stream();
cats.add("KC");
System.out.println(stream.count());
```
A List with two elements is created. Then a stream is created from that List. Remember that streams are lazily evaluated. This means that the stream isn’t actually created. An object is created that knows where to look for the data when it is needed. Than the List gets a new element. After that stream pipeline actually runs. The stream pipeline runs first, looking at the source and seeing three elements.

**Chaining Optionals** By now, you are familiar with the benefits of chaining operations in a stream pipeline. A few of the intermediate operations for streams are available for Optional. Suppose that you are given an Optional<Integer> and asked to print the value, but only if it is a three-digit number.
```java
private static void threeDigit(Optional<Integer> optional) {
   optional.map(n -> "" + n) .filter(s -> s.length() == 3) .ifPresent(System.out::println);
}
```
**Collecting Results** There are many predefined collectors all of them are stored in the Collectors class rather than the Collector class. It is very important to pass the Collector to the collect method. It exists to help collect elements. A Collector doesn’t do anything on its own.

**Collecting into Maps**
```java
Stream<String> ohMy = Stream.of("lions", "tigers", "bears");
Map<String, Integer> map = ohMy.collect( Collectors.toMap(s -> s, String::length));
System.out.println(map); // {lions=5, bears=5, tigers=6}
```
When creating a map, you need to specify two functions. The first function tells the collector how to create the key. In our example, we use the provided String as the key. The second function tells the collector how to create the value. In our example, we use the length of the String as the value.

You need to take care of duplicated keys:
```java
Stream<String> ohMy = Stream.of("lions", "tigers", "bears");
Map<Integer, String> map = ohMy.collect(Collectors.toMap( String::length, k -> k, (s1, s2) -> s1 + "," + s2));
System.out.println(map); // {5=lions,bears, 6=tigers}
System.out.println(map.getClass()); // class. java.util.HashMap
```
It so happens that the Map returned is a HashMap. This behavior is not guaranteed. Suppose that we want to mandate that the code return a TreeMap instead. No problem. We would just add a constructor reference as a parameter:
```java
Stream<String> ohMy = Stream.of("lions", "tigers", "bears");
TreeMap<Integer, String> map = ohMy.collect(Collectors.toMap( String::length, k -> k, (s1, s2) -> s1 + "," + s2, TreeMap::new));
System.out.println(map); //{5=lions,bears, 6=tigers}
System.out.println(map.getClass()); // class. java.util.TreeMap
```
**Collecting Using Grouping, Partitioning, and Mapping** Now suppose that we want to get groups of names by their length. We can do that by saying that we want to group by length:
```java
Stream<String> ohMy = Stream.of("lions", "tigers", "bears");
Map<Integer, List<String>> map = ohMy.collect( Collectors.groupingBy(String::length));
System.out.println(map); // {5=[lions, bears], 6=[tigers]}
```
The groupingBy() collector tells collect() that it should group all of the elements of the stream into lists, organizing them by the function provided. This makes the keys in the map the function value and the values the function results.

Suppose that we don’t want a List as the value in the map and prefer a Set instead. No problem. There’s another method signature that lets us pass a downstream collector. This is a second collector that does something special with the values:
```java
Stream<String> ohMy = Stream.of("lions", "tigers", "bears");
Map<Integer, Set<String>> map = ohMy.collect( Collectors.groupingBy(String::length, Collectors.toSet()));
System.out.println(map); // {5=[lions, bears], 6=[tigers]}
```
We can even change the type of Map returned through yet another parameter:
```java
Stream<String> ohMy = Stream.of("lions", "tigers", "bears");
TreeMap<Integer, Set<String>> map = ohMy.collect( Collectors.groupingBy(String::length, TreeMap::new, Collectors.toSet()));
System.out.println(map); // {5=[lions, bears], 6=[tigers]}
```
This is very flexible. What if we want to change the type of Map returned but leave the type of values alone as a List? There isn’t a method for this specifically because it is easy enough to write with the existing ones:
```java
Stream<String> ohMy = Stream.of("lions", "tigers", "bears");
TreeMap<Integer, List<String>> map = ohMy.collect( Collectors.groupingBy(String::length, TreeMap::new, Collectors.toList()));
System.out.println(map);
```
Partitioning is a special case of grouping. With partitioning, there are only two possible groups—true and false. Partitioning is like splitting a list into two parts. Suppose that we are making a sign to put outside each animal’s exhibit. We have two sizes of signs. One can accommodate names with five or fewer characters. The other is needed for longer names. We can partition the list according to which sign we need:
```java
Stream<String> ohMy = Stream.of("lions", "tigers", "bears");
Map<Boolean, List<String>> map = ohMy.collect( Collectors.partitioningBy(s -> s.length() <= 5));
System.out.println(map); // {false=[tigers], true=[lions, bears]}
```
Notice that there are still two keys in the map—one for each boolean value. It so happens that one of the values is an empty list, but it is still there.