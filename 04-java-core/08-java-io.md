# IO
* [Introducing the File Class](#introducing-the-file-class)
* [Working with a File Object](#working-with-a-file-object)
* [Stream Fundamentals](#stream-fundamentals)
* [Review of java.io Class Properties](#review-of-javaio-class-properties)
* [The java.io stream classes](#the-javaio-stream-classes)
* [Common Stream Operations](#common-stream-operations)
* [The FileInputStream and FileOutputStream Classes](#the-fileinputstream-and-fileoutputstream-classes)
* [The BufferedInputStream and BufferedOutputStream Classes](#the-bufferedinputstream-and-bufferedoutputstream-classes)
* [The FileReader and FileWriter classes](#the-filereader-and-filewriter-classes)
* [The BufferedReader and BufferedWriter Classes](#the-bufferedreader-and-bufferedwriter-classes)
* [The ObjectInputStream and ObjectOutputStream Classes](#the-objectinputstream-and-objectoutputstream-classes)
* [The Serializable Interface](#the-serializable-interface)
* [Serializing and Deserializing Objects](#serializing-and-deserializing-objects)
* [The PrintStream and PrintWriter Classes](#the-printstream-and-printwriter-classes)
* [Other Stream Classes](#other-stream-classes)
* [Interacting with Users](#interacting-with-users)
#### Introducing the File Class
The **java.io.File** class is used to read information about existing files and directories, list the contents of a directory, and create/delete files and directories. An instance of a File class represents the pathname of a particular file or directory on the file system. The File class cannot read or write data within a file, although it can be passed as a reference to many stream classes to read or write data.

A File object often is initialized with String containing either an absolute or relative path to the file or directory within the file system.

For convenience, Java offers two options to retrieve the local separator character: a system property and a static variable defined in the File class. Both of the following examples will output the separator character:
```java
System.out.println(System.getProperty("file.separator"));
System.out.println(java.io.File.separator);
```
The following code creates a File object and determines if the path it references exists within the file system:
```java
import java.io.File;
public class FileSample {
   public static void main(String[] args) {
      File file = new File("/home/smith/data/zoo.txt");
      System.out.println(file.exists());
   }
}
```
There are other constructors, such as the one that joins an existing File path with a relative child path, as shown in the following example:
```java
File parent = new File("/home/smith");
File child = new File(parent,"data/zoo.txt");
```
#### Working with a File Object
The File class contains numerous useful methods for interacting with files and directories within the file system.
* **exists()** Returns true if the file or directory exists.
* **getName()** Returns the name of the file or directory denoted by this path.
* **getAbsolutePath()**  Returns the absolute pathname string of this path.
* **isDirectory()** Returns true if the file denoted by this path is a directory.
* **isFile()** Returns true if the file denoted by this path is a file.
* **length()** Returns the number of bytes in the file. For performance reasons, the file system may allocate more bytes on disk than the file actually uses.
* **lastModified()** Returns the number of milliseconds since the epoch when the file was last modified.
* **delete()** Deletes the file or directory. If this pathname denotes a directory, then the directory must be empty in order to be deleted.
* **renameTo(File)** Renames the file denoted by this path.
* **mkdir()** Creates the directory named by this path.
* **mkdirs()** Creates the directory named by this path including any nonexistent parent directories.
* **getParent()** Returns the abstract pathname of this abstract pathname’s parent or null if this pathname does not name a parent directory.
* **listFiles()** Returns a File[] array denoting the files in the directory.

#### Stream Fundamentals
The contents of a file may be accessed or written via a stream, which is a list of data elements presented sequentially. Streams should be conceptually thought of as a long, nearly never-ending “stream of water” with data presented one “wave” at a time.

Each type of stream segments data into a “wave” or “block” in a particular way. For example, some stream classes read or write data as individual byte values. Other stream classes read or write individual characters or strings of characters. On top of that, some stream classes read or write groups of bytes or characters at a time, specifically those with the word Buffered in their name.

Java provides three built-in streams, System.in, System.err, and System.out

#### Review of java.io Class Properties
* A class with the word **InputStream** or **OutputStream** in its name is used for reading or writing binary data, respectively.
* A class with the word **Reader** or **Writer** in its name is used for reading or writing character or string data, respectively.
* Most, but not all, input classes have a corresponding output class.
* A low-level stream connects directly with the source of the data.
* A high-level stream is built on top of another stream using wrapping.
* A class with Buffered in its name reads or writes data in groups of bytes or characters and often improves performance in sequential file systems.
* When wrapping a stream you can mix and match only types that inherit from the same abstract parent stream.

#### The java.io stream classes:
* **InputStream** The abstract class all InputStream classes inherit from
* **OutputStream** The abstract class all OutputStream classes inherit from
* **Reader** The abstract class all Reader classes inherit from
* **Writer** The abstract class all Writer classes inherit from
* **FileInputStream** (low level stream) Reads file data as bytes
* **FileOutputStream** (low level stream) Writes file data as bytes
* **FileReader** (low level stream) Reads file data as characters
* **FileWriter** (low level stream) Writes file data as characters
* **BufferedReader** (high level stream) Reads character data from an existing Reader in a buffered manner, which improves efficiency and performance
* **BufferedWriter** (high level stream) Writes character data to an existing Writer in a buffered manner, which improves efficiency and performance
* **ObjectInputStream** (high level stream) Deserializes primitive Java data types and graphs of Java objects from an existing InputStream
* **ObjectOutputStream** (high level stream) Serializes primitive Java data types and graphs of Java objects to an existing OutputStream
* **InputStreamReader** (high level stream) Reads character data from an existing InputStream
* **OutputStreamWriter** (high level stream) Writes character data to an existing OutputStream
* **PrintStream** (high level stream) Writes formatted representations of Java objects to a binary stream
* **PrintWriter** (high level stream) Writes formatted representations of Java objects to a text-based output stream

#### Common Stream Operations
* **Closing the Stream** Since streams are considered resources, it is imperative that they be closed after they are used lest they lead to resource leaks. In a file system, failing to close a file properly could leave it locked by the operating system such that no other processes could read/write to it until the program is terminated.
* **Flushing the stream** When data is written to an OutputStream, the underlying operating system does not necessarily guarantee that the data will make it to the file immediately. In many operating systems, the data may be cached in memory, with a write occurring only after a temporary cache is filled or after some amount of time has passed. If the data is cached in memory and the application terminates unexpectedly, the data would be lost, because it was never written to the file system. To address this, Java provides a flush() method, which requests that all accumulated data be written immediately to disk. You do not need to call the flush() method explicitly when you have finished writing to a file, since the close() method will automatically do this. In some cases, calling the flush() method intermittently while writing a large file, rather than performing a single large flush when the file is closed, may appear to improve performance by stretching the disk access over the course of the write process.
* **Marking the Stream** The InputStream and Reader classes include mark(int) and reset() methods to move the stream back to an earlier position. Before calling either of these methods, you should call the markSupported() method, which returns true only if mark() is supported. Not all java.io input stream classes support this operation, and trying to call mark(int) or reset() on a class that does not support these operations will throw an exception at runtime. Once you’ve verified that the stream can support these operations, you can call mark(int) with a read-ahead limit value. You can then read as many bytes as you want up to the limit value. If at any point you want to go back to the earlier position where you last called mark(), then you just call reset() and the stream will “revert” to an earlier state. In practice, it’s not actually putting the data back into the stream but storing the data that was already read into memory for you to read again. Therefore, you should not call the mark() operation with too large a value as this could take up a lot of memory. Finally, if you call reset() after you have passed your mark() read limit, an exception may be thrown at runtime since the marked position may become invalidated. We say “an exception may be thrown” as some implementations may use a buffer to allow extra data to be read before the mark is invalidated.
* **Skipping over Data** The InputStream and Reader classes also include a skip(long) method, which as you might expect skips over a certain number of bytes. It returns a long value, which indicates the number of bytes that were actually skipped. If the return value is zero or negative, such as if the end of the stream was reached, no bytes were skipped. Calling the skip() operation is equivalent to calling read() and discarding the output. For skipping a handful of bytes, there is virtually no difference. On the other hand, for skipping a large number of bytes, skip() will often be faster, because it will use arrays to read the data.

#### The FileInputStream and FileOutputStream Classes
They are used to read bytes from a file or write bytes to a file, respectively. These classes include constructors that take a File object or String, representing a path to the file.

The data in a **FileInputStream** object is commonly accessed by successive calls to the **read()** method until a value of -1 is returned, indicating that the end of the stream—in this case the end of the file—has been reached. Although less common, you can also choose to stop reading the stream early just by exiting the loop, such as if some search String is found. The FileInputStream class also contains overloaded versions of the read() method, which take a pointer to a byte array where the data is written. The method returns an integer value indicating how many bytes can be read into the byte array. It is also used by Buffered classes to improve performance.

A **FileOutputStream** object is accessed by writing successive bytes using the **write(int)** method. Like the FileInputStream class, the FileOutputStream also contains overloaded versions of the write() method that allow a byte array to be passed and can be used by Buffered classes.

The following code uses FileInputStream and FileOutputStream to copy a file:
```java
import java.io.*;
public class CopyFileSample {
   public static void copy(File source, File destination) throws IOException {
      try (InputStream in = new FileInputStream(source);
             OutputStream out = new FileOutputStream(destination)) {
         int b;
         while((b = in.read()) != -1) {
            out.write(b);
         }
      }
   }
   public static void main(String[] args) throws IOException {
      File source = new File("Zoo.class");
      File destination = new File("ZooCopy.class");
      copy(source,destination);
   }
}
```
When reading a single value of a FileInputStream instance, the read() method returns a primitive int value rather than a byte value. It does this so that it has an additional value available to be returned, specifically -1, when the end of the file is reached. If the class did return a byte instead of an int, there would be no way to know whether the end of the file had been reached based on the value returned from the read() method, since the file could contain all possible byte values as data. For compatibility, the FileOutputStream also uses int instead of byte for writing a single byte to a file.

#### The BufferedInputStream and BufferedOutputStream Classes
We can enhance our implementation with only a few minor code changes by wrapping the **FileInputStream** and **FileOutputStream** classes with the **BufferedInputStream** and **BufferedOutputStream** classes, respectively. Instead of reading the data one byte at a time, we use the underlying **read(byte[])** method of BufferedInputStream, which returns the number of bytes read into the provided byte array. The number of bytes read is important for two reasons. First, if the value returned is 0, then we know that we have reached the end of the file and can stop reading from the BufferedInputStream. Second, the last read of the file will likely only partially fill the byte array, since it is unlikely for the file size to be an exact multiple of our buffer array size. For example, if the buffer size is 1,024 bytes and the file size is 1,054 bytes, then the last read will be only 30 bytes. The length value tells us how many of the bytes in the array were actually read from the file. The remaining bytes of the array will be filled with leftover data from the previous read that should be discarded.

The data is written into the **BufferedOutputStream** using the **write(byte[],int,int)** method, which takes as input a byte array, an offset, and a length value, respectively. The offset value is the number of values to skip before writing characters, and it is often set to 0. The length value is the number of characters from the byte array to write.

Example:
```java
import java.io.*;
public class CopyBufferFileSample {
   public static void copy(File source, File destination) throws IOException {
      try (InputStream in = new BufferedInputStream(new FileInputStream(source));
           OutputStream out = new BufferedOutputStream(new FileOutputStream(destination))) {
         byte[] buffer = new byte[1024];
         int lengthRead;
         while ((lengthRead = in.read(buffer)) > 0) {
            out.write(buffer,0,lengthRead);
            out.flush();
         }
      }
   }
}
```

#### The FileReader and FileWriter classes
Like the **FileInputStream** and **FileOutputStream** classes, the **FileReader** and **FileWriter** classes contain **read()** and **write()** methods, respectively. These methods read/write char values instead of byte values; although similar to what you saw with streams, the API actually uses an int value to hold the data so that -1 can be returned if the end of the file is detected. The **Writer** class, which **FileWriter** inherits from, offers a **write(String)** method that allows a String object to be written directly to the stream. Using FileReader also allows you to pair it with BufferedReader in order to use the very convenient readLine() method.

#### The BufferedReader and BufferedWriter Classes
**BufferedReader** introduces a new method **readLine()** to read the whole line in a row and return the String object. When the end of the file is reached the method returns null. BufferedWriter has a **write(String s)** method to write a String object to the file. Another useful method is **newLine()** to insert a line break into the file, as our readLine() method split on line breaks.

#### The ObjectInputStream and ObjectOutputStream Classes
The process of converting an in-memory object to a stored data format is referred to as serialization, with the reciprocal process of converting stored data into an object, which is known as deserialization.

#### The Serializable Interface
In order to serialize objects using the java.io API, the class they belong to must implement the **java.io.Serializable** interface. The Serializable interface is a tagging or marker interface. The purpose of implementing the Serializable interface is to inform any process attempting to serialize the object that you have taken the proper steps to make the object serializable, which involves making sure that the classes of all instance variables within the object are also marked Serializable. Many of the built-in Java classes including the String class, are marked Serializable.

A process attempting to serialize an object will throw a **NotSerializableException** if the class or one of its contained classes does not properly implement the **Serializable** interface. Let’s say that you have a particular object within a larger object that is not serializable, such as one that stores temporary state or metadata information about the larger object. You can use the **transient** keyword on the reference to the object, which will instruct the process serializing the object to skip it and avoid throwing a **NotSerializableException**. The only limitation is that the data stored in the object will be lost during the serialization process. Besides transient instance variables, static class members will also be ignored during the serialization and deserialization process. This should follow logically, as static class variables do not belong to one particular instance. If you need to store static class information, it will be need to be copied to an instance object and serialized separately.

This **serialVersionUID** is stored with the serialized object and assists during the deserialization process. The serialization process uses the serialVersionUID to identify uniquely a version of the class. That way, it knows whether the serialized data for an object will match the instance variable in the current version of the class. If an older version of the class is encountered during deserialization, an exception may be thrown. Alternatively, some deserialization tools support conversions automatically. We recommend that you do not rely on the generated serialVersionUID provided by the Java compiler and that you explicitly declare one in each of your Serializable classes. Different Java compiler versions across different platforms may differ in their implementation of the generated serialVersionUID.

#### Serializing and Deserializing Objects
The java.io API provides two stream classes for object serialization and deserialization called **ObjectInputStream** and **ObjectOutputStream**. The **ObjectOutputStream** class includes a method to serialize the object to the stream called void **writeObject(Object)**. For the reciprocal process, the **ObjectInputStream** class includes a deserialization method that returns an object called **readObject()**. Notice that the return type of this method is the generic type **java.lang.Object**, indicating that the object will have to be cast explicitly at runtime to be used.

Next, the **readObject()** throws the checked exception, **ClassNotFoundException**, since the class of the deserialized object may not be available to the JRE. Finally, since we are reading objects, we can’t use a -1 integer value to determine when we have finished reading a file. Instead, the proper technique is to catch an **EOFException**, which marks the program encountering the end of the file.

When you deserialize an object, the constructor of the serialized class is not called. In fact, Java calls the first no-arg constructor for the first nonserializable parent class, skipping the constructors of any serialized class in between.

#### The PrintStream and PrintWriter Classes
The **PrintStream** and **PrintWriter** classes are high-level stream classes that write formatted representation of Java objects to a text-based output stream. For convenience, both of these classes include constructors that can open and write to files directly. Furthermore, the **PrintWriter** class even has a constructor that takes an **OutputStream** as input, allowing you to wrap a PrintWriter class around an OutputStream. System.out and System.err are actually PrintStream objects.

Because **PrintStream** inherits **OutputStream** and **PrintWriter** inherits from **Writer**, both support the underlying **write()** method while providing a slew of print-based methods.  These print-based methods do not throw any checked exceptions. If they did, you would have been required to catch a checked exception anytime you called **System.out.println()**. Both classes provide a method, **checkError()**, that can be used to detect the presence of a problem after attempting to write data to the stream.

Method examples:
* **print()** The most basic of the print-based methods is print(), which is overloaded with all Java primitives as well as String and Object. In general, these methods perform String.valueOf() on the argument and call the underlying stream’s write() method, although they also handle character encoding automatically.
* **println()** The next methods available in the PrintStream and PrintWriter classes are the println() methods, which are virtually identical to the print() methods, except that they insert a line break after the String value is written. The classes also include a version of println() that takes no arguments, which terminates the current line by writing a line separator.
* **format() and printf()** Like the String.format() methods, the format() method in PrintStream and PrintWriter takes a String, an optional locale, and a set of arguments, and it writes a formatted String to the stream based on the input. In other words, it is a convenience method for formatting directly to the stream. For convenience, as well as to make C developers feel more at home in Java, the PrintStream and PrintWriter APIs also include a set of printf() methods, which are straight pass-through methods to the format() methods.

#### Other Stream Classes
The high-level **InputStreamReader** and **OutputStreamWriter** are useful in practice. The **InputStreamReader** class takes an **InputStream** instance and returns a **Reader** object. Likewise, the **OutputStreamWriter** class takes an **OutputStream** instance and returns a **Writer** object. In this manner, these classes convert data between character and byte streams.

#### Interacting with Users
The **Console** class was introduced in Java 6 as a more evolved form of the System.in and System.out stream classes. It is now the recommended technique for interacting with and displaying information to the user in a text-based environment.

**The Old Way** Similar to how **System.out** returns a **PrintStream** and is used to output text data to the user, **System.in** returns an **InputStream** and is used to retrieve text input from the user. It can be chained to a BufferedReader to allow input that terminates with the Enter key. Before we can apply the BufferedReader, though, we need to wrap the System.in object using the InputStreamReader class, which allows us to build a Reader object out of an existing InputStream instance. Closing System.in would prevent our application from accepting user input for the remainder of the application execution.

**The New Way** To begin, the **Console** class is a singleton. It is created automatically for you by the JVM and accessed by calling the **System.console()** method. Be aware that this method will return null in environments where text interactions are not supported.

Methods available in the Console class:
* **reader() and writer()** The Console class provides access to an instance of Reader and PrintWriter using the methods reader() and writer(), respectively. Access to these classes is analogous to calling System.in and System.out directly, although they use the Reader/Writer classes instead of the InputStream/OutputStream classes, which are more appropriate for working with character and String data. In this manner, they handle the underlying character encoding automatically.
* **format() and printf()** For outputting data to the user, you can use the PrintWriter writer() object or use the convenience format(String,Object...) method directly. The format() method takes a String format and list of arguments, and it behaves in the exact same manner as String. Note that the Console class defines only one format() method, and it does not define a format() method that takes a locale variable. In this manner, it uses the default system locale to establish the formatter.
* **flush()** The flush() method forces any buffered output to be written immediately. It is recommended that you call the flush() method prior to calling any readLine() or readPassword() methods in order to ensure that no data is pending during the read. Failure to do so could result in a user prompt for input with no preceding text, as the text prior to the prompt may still be in a buffer.
* **readLine()** The basic readLine() method retrieves a single line of text from the user, and the user presses the Enter key to terminate it. The Console class also supports an overloaded version of the readLine() method with the signature readLine(String format, Object... args), which displays a formatted prompt to the user prior to accepting text.
* **readPassword()** The readPassword() method is similar to the readLine() method, except that echoing is disabled. By disabling echoing, the user does not see the text they are typing, meaning that their password is secure if someone happens to be looking at their screen. Also like the readLine() method, the Console class offers an overloaded version of the readPassword() method with the signature readPassword(String format, Object... args) used for displaying a formatted prompt to the user prior to accepting text. Unlike the readLine() method, though, the readPassword() method returns an array of characters instead of a String.
Example: 
```java
String name = console.readLine(“Please enter your name: “);
```
String values are added to a shared memory pool for performance reasons in Java. This means that if a password that a user typed in were to be returned to the process as a String, it might be available in the String pool long after the user entered it. As soon as the data is read and used, the sensitive password data in the array can be “erased” by writing garbage data to the elements of the array. This would remove the password from memory long before it would be removed by garbage collection if a String value were used. Note that you can use Array.fill(password,'x') to wipe an array’s data.