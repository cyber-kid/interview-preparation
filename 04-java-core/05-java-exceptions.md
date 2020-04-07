# Exceptions
* [Exception examples](#exception-examples)
* [Try Statement](#try-statement)
* [Creating Custom Exceptions](#creating-custom-exceptions)
* [Using Multi-catch](#using-multi-catch)
* [Using Try-With-Resources](#using-try-with-resources)

Remember that a **runtime exception**, or **unchecked exception**, may be caught, but it is not required that it be caught. After all, if you had to check for **NullPointerException**, every piece of code that you wrote would need to deal with it. A **checked exception** is any class that extends **Exception** but is not a **runtime exception**. **Checked exceptions** must follow the handle or declare rule where they are either caught or thrown to the caller. Only **checked exceptions** that have the potential to be thrown are allowed to be caught.An **error** is fatal and should not be caught by the program. While it is legal to catch an error, it is not a good practice.

#### Exception examples:
- **ArithmeticException** Thrown by the JVM when code attempts to divide by zero.
- **ArrayIndexOutOfBoundsException** Thrown by the JVM when code uses an illegal index to access an array.
- **ClassCastException** Thrown by the JVM when an attempt is made to cast an object to a subclass of which it is not an instance.
- **IllegalArgumentException** Thrown by the program to indicate that a method has been passed an illegal or inappropriate argument.
- **NullPointerException** Thrown by the JVM when there is a null reference where an object is required.
- **NumberFormatException** Thrown by the program when an attempt is made to convert a string to a numeric type, but the string doesn’t have an appropriate format.
- **java.text.ParseException** Converting a String to a number. Is checked exception.
- **java.io.IOException**, java.io.FileNotFoundException, java.io.NotSerializableException Dealing with IO and NIO.2 issues. IOException is the parent class. There are a number of subclasses. You can assume any java.io exception is checked.
- **java.sql.SQLException** Dealing with database issues. SQLException is the parent class. Again, you can assume any java.sql exception is checked.
- **java.lang.ArrayStoreException** Trying to store the wrong data type in an array. Is unchecked exception.
- **java.time.DateTimeException** Receiving an invalid format string for a date. Is unchecked exception.
- **java.util.MissingResourceException** Trying to access a key or resource bundle that does not exist. Is unchecked exception.
- **java.lang.IllegalStateException**, java.lang.UnsupportedOperationException Attempting to run an invalid operation in collections and concurrency are unchecked exceptions.
#### Try Statement
The try statement consists of a mandatory **try** clause. It can include one or more **catch** clauses to handle the exceptions that are thrown. It can also include a **finally** clause, which runs regardless of whether an exception is thrown. This is all still true for both try statements and **try-with-resources** statements.
Example:
```java
try{
   //protected code
}catch(exceptiontype identifier){
   //exception handler
}finally{
   //finally block
}
```
Rules:
* A try statement is required to have either or both of the catch and finally clauses. This is true for try statements, but is not true for try-with-resources statements.
* Java checks the catch blocks in the order in which they appear. It is illegal to declare a subclass exception in a catch block that is lower down in the list than a superclass exception because it will be unreachable code.
* Java will not allow you to declare a catch block for a checked exception type that cannot potentially be thrown by the try clause body. This is again to avoid unreachable code.
#### Creating Custom Exceptions
When creating your own exception, you need to decide whether it should be a checked or unchecked exception. While you can extend any exception class, it is most common to extend Exception (for checked) or RuntimeException (for unchecked.)

The following example shows the three most common constructors defined by the Exception class:
```java
public class CannotSwimException extends Exception {
   public CannotSwimException() {
      super();
   }
   public CannotSwimException(Exception e) {
      super(e);
   }
   public CannotSwimException(String message) {
      super(message);
   }
}
```
The first constructor is the default constructor with no parameters. The second constructor shows how to wrap another exception inside yours. The third constructor shows how to pass a custom error message.

The error messages that we’ve been showing are called a stack trace . They show the exception along with the method calls it took to get there. Java automatically prints the stack trace when the program handles an exception.
You can also print the stack trace on your own:
```java
try {
   throw new CannotSwimException();
} catch (CannotSwimException e) {
   e.printStackTrace();
}
```
#### Using Multi-catch
In Java 7, they introduced the ability to catch multiple exceptions in the same catch block, also known as multi-catch. Now we have an elegant solution to the problem:
```java
public static void main(String[] args) {
   try {
      Path path = Paths.get("dolphinsBorn.txt");
      String text = new String(Files.readAllBytes(path));
      LocalDate date = LocalDate.parse(text);
      System.out.println(date);
   } catch (DateTimeParseException | IOException e) {
      e.printStackTrace();
      throw new RuntimeException(e);
   }
}
```
It’s like a regular catch clause, except two or more exception types are specified separated by a pipe. The pipe is also used as the “or” operator, making it easy to remember that you can use either/or of the exception types. Notice how there is only one variable name in the catch clause. Java is saying that the variable named e can be of type Exception1 or Exception2.

Java intends multi-catch to be used for exceptions that aren’t related, and it prevents you from specifying redundant types in a multi-catch.

Variable e in multi-catch block is effectively final and can not be reassigned.
#### Using Try-With-Resources
```java
public void newApproach(Path path1, Path path2) throws IOException {
   try (BufferedReader in = Files.newBufferedReader(path1);
        BufferedWriter out = Files.newBufferedWriter(path2)) {
      out.write(in.readLine());
   }
}
```
There is no longer code just to close resources. The new try-with-resources statement automatically closes all resources opened in the try clause. This feature is also known as automatic resource management, because Java automatically takes care of the closing.

* Remember that only a try-with-resources statement is permitted to omit both the catch and finally blocks. A traditional try statement must have either or both.
* Notice that one or more resources can be opened in the try clause. Also, notice that parentheses are used to list those resources and semicolons are used to separate the declarations.
* A try-with-resources statement is still allowed to have catch and/or finally blocks. They are run in addition to the implicit one. The implicit finally block runs before any programmer-coded ones.
* The resources created in the try clause are only in scope within the try block. This is another way to remember that the implicit finally runs before any catch/finally blocks that you code yourself. The implicit close has run already, and the resource is no longer available.
* Resources are closed after the try clause ends and before any catch/finally clauses.
* Resources are closed in the reverse order from which they were created.
* The auto-closeable variables defined in the try-with-resources statement are implicitly final. Thus, they cannot be reassigned.
#### AutoCloseable
In order for a class to be created in the try clause, Java requires it to implement an interface called AutoCloseable. The AutoCloseable interface has only one method to implement:
```java
public void close() throws Exception;
```
An overriding method is allowed to declare more specific exceptions than the parent or even none at all. By declaring Exception, the AutoCloseable interface is saying that implementers may throw any exceptions they choose.

The try-with-resources statement throws a checked exception. And you know that checked exceptions need to be handled or declared. Tricky isn’t it? This is something that you need to watch for on the exam. If the main() method declared an Exception, this code would compile. Java strongly recommends that close() not actually throw Exception. It is better to throw a more specific exception. Java also recommends to make the close() method idempotent. Idempotent means that the method can called be multiple times without any side effects or undesirable behavior on subsequent runs. For example, it shouldn’t throw an exception the second time or change state or the like. Both these negative practices are allowed. They are merely discouraged.

What happens if the close() method throws an exception? Clearly we need to handle such a condition. We already know that the resources are closed before any programmer-coded catch blocks are run. This means that we can catch the exception thrown by close() if we wish. Alternatively, we can allow the caller to deal with it. Just like a regular exception, checked exceptions must be handled or declared.

Before the AutoCloseable the Closeable interface existed. Closeable restricts the type of exception thrown to IOException. AutoCloseable extends Closeable.
#### Suppressed Exceptions
This seems reasonable enough. What happens if the try block also throws an exception? Java 7 added a way to accumulate exceptions. When multiple exceptions are thrown, all but the first are called suppressed exceptions. The idea is that Java treats the first exception as the primary one and tacks on any that come up while automatically closing, for example:
```java
try (JammedTurkeyCage t = new JammedTurkeyCage()) {
   throw new IllegalStateException("turkeys ran off");
} catch (IllegalStateException e) {
   System.out.println("caught: " + e.getMessage());
   for (Throwable t: e.getSuppressed())
   System.out.println(t.getMessage());
}
```
In this case the try block throws an exception IllegalStateException, then Java automatically calls the close() method of resource which also throws an exception, which is added as a suppressed exception. The loop in a catch block goes through the suppressed exceptions that are returned by getSuppressed() method of primary exception. Keep in mind that the catch block looks for matches on the primary exception. Java remembers the suppressed exceptions that go with a primary exception even if we don’t handle them in the code. Finally, keep in mind that suppressed exceptions apply only to exceptions thrown in the try clause.
#### Re-throwing Exceptions
It is a common pattern to log and then throw the same exception. Suppose that we have a method that declares two checked exceptions:
public void parseData() throws SQLException, DateTimeParseException {}

We can catch, log and re-throw them in such way:
```java
public void rethrowing() throws SQLException, DateTimeParseException {
   try {
      parseData();
   } catch (Exception e) {
      System.err.println(e);
      throw e;
   }
}
```