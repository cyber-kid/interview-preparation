# NIO.2
* [Introducing Path](#introducing-path)
* [Creating Instances with Factory and Helper Classes](#creating-instances-with-factory-and-helper-classes)
* [Creating Paths](#creating-paths)
* [Accessing the Underlying FileSystem Object](#accessing-the-underlying-filesystem-object)
* [Working with Legacy File Instances](#working-with-legacy-file-instances)
* [Interacting with Paths and Files](#interacting-with-paths-and-files)
* [Using Path Objects](#using-path-objects)
* [Interacting with Files](#interacting-with-files)
* [Understanding File Attributes](#understanding-file-attributes)
* [Understanding Views](#understanding-views)
* [Reading Attributes](#reading-attributes)
* [Modifying Attributes](#modifying-attributes)
* [Walking a Directory](#walking-a-directory)
* [Searching a Directory](#searching-a-directory)
* [Listing Directory Contents](#listing-directory-contents)
* [Printing File Contents](#printing-file-contents)
#### Introducing Path
The **java.nio.file.Path** interface is the primary entry point for working with the NIO.2 API. A **Path** object represents a hierarchical path on the storage system to a file or directory. In this manner, **Path** is a direct replacement for the legacy **java.io.File** class, and conceptually it contains many of the same properties. Unlike the **File** class, the **Path** interface contains support for symbolic links. A symbolic link is a special file within an operating system that serves as a reference or pointer to another file or directory.

#### Creating Instances with Factory and Helper Classes
You can create an instance of a Path interface using a static method available in the Paths factory class. Note the s at the end of the Paths class to distinguish it from the Path interface. NIO.2 also includes helper classes such as **java.nio.file.File**, whose primary purpose is to operate on instances of Path objects. Helper or utility classes are similar to factory classes in that they are often composed primarily of static methods that operate on a particular class. They differ in that helper classes are focused on manipulating or creating new objects from existing instances, whereas factory classes are focused primarily on object creation.

The reason why **Path** is an interface and not a class is that creating a file or directory is considered a file system–dependent task in NIO.2. When you obtain a Path object from the default file system in NIO.2, the JVM gives you back an object that unlike **java.io.File** transparently handles system-specific details for the current platform.

#### Creating Paths
The simplest and most straightforward way to obtain a Path object is using the **java.nio.files.Paths** factory class. To obtain a reference to a file or you would call the static method **Paths.getPath(String)** method, as shown in the following examples:
```java
Path path1 = Paths.get("pandas/cuddly.png");
Path path2 = Paths.get("c:\\zooinfo\\November\\employees.txt");
Path path3 = Paths.get("/home/zoodirector");
```
You can also create a Path using the Paths class using a vararg of type String, such as **Paths.get(String,String...)**. This allows you to create a Path from a list of String values in which the operating system-dependent path.separator is automatically inserted between elements. As you may remember, **System.getProperty("path.separator")** can be used to get the operating system-dependent file separator from the JVM. Examples:
```java
Path path1 = Paths.get("pandas","cuddly.png");
Path path2 = Paths.get("c:","zooinfo","November","employees.txt");
Path path3 = Paths.get("/","home","zoodirector");
```
Another way to construct a Path using the Paths class is with a URI value. A uniform resource identifier (URI) is a string of characters that identify a resource. It begins with a schema that indicates the resource type, followed by a path value. Examples of schema values include file://, http://, https://, and ftp://. The java.net.URI class is used to create and manage URI values. Examples:
```java
Path path1 = Paths.get(new URI("file://pandas/cuddly.png")); // THROWS EXCEPTION // AT RUNTIME
Path path2 = Paths.get(new URI("file:///c:/zoo-info/November/employees.txt"));
Path path3 = Paths.get(new URI("file:///home/zoodirectory"));
```
The first example actually throws an exception at runtime, as URIs must reference absolute paths at runtime. The URI class does have an **isAbsolute()** method, although this is related to whether or not the URI has a schema, not the file location. Note that the constructor new URI(String) does throw a checked URISyntaxException.

Finally, the Path interface also contains a reciprocal method **toUri()** for converting a Path instance back to a URI instance, as shown in the following sample code:
```java
Path path4 = Paths.get(new URI("http://www.wiley.com"));
URI uri4 = path4.toUri();
```
#### Accessing the Underlying FileSystem Object
The **Paths.getPath()** method used throughout the previous examples is actually shorthand for the class **java.nio.file.FileSystem** method **getPath()**. The **FileSystem** class has a protected constructor, so we use the plural **FileSystems** factory class to obtain an instance of **FileSystem**, as shown in the following example code:
```java
Path path1 = FileSystems.getDefault().getPath("pandas/cuddly.png");
Path path2 = FileSystems.getDefault().getPath("c:","zooinfo","November", "employees.txt");
Path path3 = FileSystems.getDefault().getPath("/home/zoodirector");
```
While most of the time we want access to a Path object that is within the local file system, the FileSystems factory class does give us the ability to connect to a remote file system, as shown in the following sample code:
```java
FileSystem fileSystem = FileSystems.getFileSystem( new URI("http://www.selikoff.net"));
Path path = fileSystem.getPath("duck.txt");
```
This code is useful when we need to construct Path objects frequently for a remote file system. The power of the NIO.2 API here is that it lets us rely on the default file system for files and directories as before, while giving us the ability to build more complex applications that reference external file systems.
#### Working with Legacy File Instances
When Path was added in Java 7, the legacy java.io.File class was updated with a new method, toPath(), that operates on an instance File variable:
```java
File file = new File("pandas/cuddly.png");
Path path = file.toPath();
```
For backward compatibility, the Path interface also contains a method toFile() to return a File instance:
```java
Path path = Paths.get("cuddly.png");
File file = path.toFile();
```
#### Interacting with Paths and Files
Many of the methods in the NIO.2 API that interact with real files and directories take additional options args in the form of a vararg. Note that these descriptions apply to both files and directories. Examples: 
* **NOFOLLOW_LINKS** (Usage: Test file existing, Read file data, Copy file, Move file) If provided, symbolic links when encountered will not be traversed. Useful for performing operations on symbolic links themselves rather than their target.
* **FOLLOW_LINKS** (Usage: Traverse a directory tree) If provided, symbolic links when encountered will be traversed.
* **COPY_ATTRIBUTES** (Usage:Copy file) If provided, all metadata about a file will be copied with it.
* **REPLACE_EXISTING** (Usage: Copy file Move file) If provided and the target file exists, it will be replaced; otherwise, if it is not provided, an exception will be thrown if the file already exists.
* **ATOMIC_MOVE** (Usage: Move file) The operation is performed in an atomic manner within the file system, ensuring that any process using the file sees only a complete record. Method using it may throw an exception if the feature is unsupported by the file system.
#### Using Path Objects
Many of the methods in the Path interface transform the path value in some way and return a new Path object, allowing the methods to be chained.

**Viewing the Path with toString(), getNameCount(), and getName()** The first method, toString(), returns a String representation of the entire path. The second and third methods, getNameCount() and getName(int), are often used in conjunction to retrieve the number of elements in the path and a reference to each element, respectively. For greater compatibility with other NIO.2 methods, the getName(int) method returns the component of the Path as a new Path object rather than a String. The following sample code uses these methods to retrieve path data:
```java
Path path = Paths.get("/land/hippo/harry.happy");
System.out.println("The Path Name is: "+path);
for(int i=0; i<path.getNameCount(); i++) {
   System.out.println(" Element "+i+" is: "+path.getName(i));
}
```
The output is:
```java
The Path Name is: /land/hippo/harry.happy
Element 0 is: land
Element 1 is: hippo
Element 2 is: harry.happy
```
**Accessing Path Components with getFileName(), getParent(), and getRoot()** The Path interface contains numerous methods for retrieving specific subelements of a Path object, returned as Path objects themselves. The first method, getFileName(), returns a Path instance representing the filename, which is the farthest element from the root. Like most methods in the Path interface, getFileName() returns a new Path instance rather than a String. The next method, getParent(), returns a Path instance representing the parent path or null if there is no such parent. If the instance of the Path object is relative, this method will stop at the top-level element defined in the Path object. In other words, it will not traverse outside the working directory to the file system root. The last method, getRoot(), returns the root element for the Path object or null if the Path object is relative.

**Checking Path Type with isAbsolute() and toAbsolutePath()** The Path interface contains two methods for assisting with relative and absolute paths. The first method, isAbsolute(), returns true if the path the object references is absolute and false if the path the object references is relative. The second method, toAbsolutePath(), converts a relative Path object to an absolute Path object by joining it to the current working directory. If the Path object is already absolute, then the method just returns an equivalent copy of it. Keep in mind that if the Path object already represents an absolute path, then the output is a new Path object with the same value.

**Creating a New Path with subpath()** The method subpath(int,int) returns a relative subpath of the Path object, referenced by an inclusive start index and an exclusive end index. It is useful for constructing a new relative path from a particular parent path element to another parent path element, as shown in the following example:
```java
Path path = Paths.get("/mammal/carnivore/raccoon.image");
System.out.println("Path is: "+path);
System.out.println("Subpath from 0 to 3 is: "+path.subpath(0,3));
System.out.println("Subpath from 1 to 3 is: "+path.subpath(1,3));
System.out.println("Subpath from 1 to 2 is: "+path.subpath(1,2));
```
The output is:
```java
Path is: /mammal/carnivore/raccoon.image
Subpath from 0 to 3 is: mammal/carnivore/raccoon.image
Subpath from 1 to 3 is: carnivore/raccoon.image
Subpath from 1 to 2 is: carnivore
```
**Deriving a Path with relativize()** The Path interface provides a method relativize(Path) for constructing the relative path from one Path object to another. Consider the following relative and absolute path examples using the relativize() method.
```java
Path path1 = Paths.get("fish.txt");
Path path2 = Paths.get("birds.txt");
System.out.println(path1.relativize(path2));
System.out.println(path2.relativize(path1));
```
The code snippet produces the following output when executed:
```java
..\birds.txt
..\fish.txt
```
Alternatively, if both path values are absolute, then the method computes the relative path from one absolute location to another, regardless of the current working directory. The following example demonstrates this property:
```java
Path path3 = Paths.get("E:\\habitat");
Path path4 = Paths.get("E:\\sanctuary\\raven");
System.out.println(path3.relativize(path4));
System.out.println(path4.relativize(path3));
```
This code snippet produces the following output when executed:
```java
..\sanctuary\raven
..\..\habitat
```
Note that the file system is not accessed to perform this comparison. For example, the root path element E: may not exist in the file system, yet the code would execute without issue since Java is referencing the path elements and not the actual file values.

The **relativize()** method requires that both paths be absolute or both relative, and it will throw an IllegalArgumentException if a relative path value is mixed with an absolute path value.

**Joining Path Objects with resolve()** The Path interface includes a resolve(Path) method for creating a new Path by joining an existing path to the current path. To put it another way, the object on which the resolve() method is invoked becomes the basis of the new Path object, with the input argument being appended onto the Path. Let’s see what happens if we apply resolve() to an absolute path and a relative path:
```java
final Path path1 = Paths.get("/cats/../panther");
final Path path2 = Paths.get("food");
System.out.println(path1.resolve(path2));
```
The code snippet generates the following output:
```java
/cats/../panther/food
```
If an absolute path is provided as input to the method, such as path1. resolve(path2), then path1 would be ignored and a copy of path2 would be returned.

**Cleaning Up a Path with normalize()** As you saw with the **relativize()** method, file systems can construct relative paths using .. and . values. There are times, however, when relative paths are combined such that there are redundancies in the path value. Luckily, Java provides us with the normalize(Path) method to eliminate the redundancies in the path.

For example, let’s take the output of one of our previous examples that resulted in the path value ..\user\home and try to reconstitute the original absolute path using the resolve() method:
```java
Path path3 = Paths.get("E:\\data");
Path path4 = Paths.get("E:\\user\\home");
Path relativePath = path3.relativize(path4);
System.out.println(path3.resolve(relativePath));
```
The result of this sample code would be the following output:
```java
E:\data\..\user\home
```

You can see that this path value contains a redundancy. Worse yet, it does not match our original value, E:\user\home. We can resolve this redundancy by applying the normalize() method as shown here:
```java
System.out.println(path3.resolve(relativePath).normalize());
```

This modified last line of code nicely produces our original path value:
```java
E:\user\home
```

Like **relativize()**, the **normalize()** method does not check that the file actually exists.

**Checking for File Existence with toRealPath()** The toRealPath(Path) method takes a Path object that may or may not point to an existing file within the file system, and it returns a reference to a real path within the file system. It is similar to the toAbsolutePath() method in that it can convert a relative path to an absolute path, except that it also verifies that the file referenced by the path actually exists, and thus it throws a checked IOException at runtime if the file cannot be located. It is also the only Path method to support the NOFOLLOW_LINKS option. The toRealPath() method performs additional steps, such as removing redundant path elements. In other words, it implicitly calls normalize() on the resulting absolute path. Finally, we can also use the toRealPath() method to gain access to the current working directory, such as shown here:
```java
System.out.println(Paths.get(".").toRealPath());
```
#### Interacting with Files
For starters, many of the same operations available in java.io.File are available to java.nio.file.Path via a helper class called **java.nio.file.Files**. Unlike the methods in the Path and Paths class, most of the options within the Files class will throw an exception if the file to which the Path refers does not exist. The Files class contains numerous static methods for interacting with files, with most taking one or two Path objects as arguments. Some of these methods are capable of throwing the checked IOException at runtime, often when the file being referenced does not exist within the file system, as you saw with the Path method toRealPath().

**Testing a Path with exists()** The Files.exists(Path) method takes a Path object and returns true if, and only if, it references a file that exists in the file system. You can see that this method does not throw an exception if the file does not exist, as doing so would prevent this method from ever returning false at runtime.

**Testing Uniqueness with isSameFile()** The Files.isSameFile(Path,Path) method is useful for determining if two Path objects relate to the same file within the file system. It takes two Path objects as input and follows symbolic links. Despite the name, the method also determines if two Path objects refer to the same directory. The isSameFile() method first checks if the Path objects are equal in terms of equal(), and if so, it automatically returns true without checking to see if either file exists. If the Path object equals() comparison returns false, then it locates each file to which the path refers in the file system and determines if they are the same, throwing a checked IOException if either file does not exist. This isSameFile() method does not compare the contents of the file.

**Making Directories with createDirectory() and createDirectories()** In the NIO.2 API, we can use the Files.createDirectory(Path) method to create a directory. There is also a plural form of the method called createDirectories(), which like mkdirs() creates the target directory along with any nonexistent parent directories leading up to the target directory in the path. The directory creation methods can throw the checked IOException, such as when the directory cannot be created or already exists. For example, the first method, createDirectory(), will throw an exception if the parent directory in which the new directory resides does not exist. Both of these methods also accept an optional list of FileAttribute<?> values to set on the newly created directory or directories.

**Duplicating File Contents with copy()** The NIO.2 Files class provides a set of overloaded copy() methods for copying files and directories within the file system. The primary one is Files.copy(Path,Path), which copies a file or directory from one location to another. The copy() method throws the checked IOException, such as when the file or directory does not exist or cannot be read. Directory copies are shallow rather than deep, meaning that files and subdirectories within the directory are not copied. By default, copying files and directories will traverse symbolic links, although it will not overwrite a file or directory if it already exists, nor will it copy file attributes. These behaviors can be altered by providing the additional options NOFOLLOW_LINKS, REPLACE_EXISTING, and COPY_ATTRIBUTES, respectively. The NIO.2 Files API class contains two overloaded copy() methods for copying files using java.io streams. The first copy() method takes a source java.io.InputStream along with a target Path object. It reads the contents from the stream and writes the output to a file represented by a Path object. The second copy() method takes a source Path object and target java.io.OutputStream. It reads the contents of the file and writes the output to the stream.

**Changing a File Location with move()** The Files.move(Path,Path) method moves or renames a file or directory within the file system. Like the copy() method, the move() method also throws the checked IOException in the event that the file or directory could not be found or moved. By default, the move() method will follow links, throw an exception if the file already exists, and not perform an atomic move. These behaviors can be changed by providing the optional values NOFOLLOW_LINKS, REPLACE_EXISTING, or ATOMIC_MOVE, respectively, to the method. If the file system does not support atomic moves, an AtomicMoveNotSupportedException will be thrown at runtime. The Files.move() method can be applied to non-empty directories only if they are on the same underlying drive. While moving an empty directory across a drive is supported, moving a non-empty directory across a drive will throw an NIO.2 DirectoryNotEmptyException.

**Removing a File with delete() and deleteIfExists()** The Files.delete(Path) method deletes a file or empty directory within the file system. The delete() method throws the checked IOException under a variety of circumstances. For example, if the path represents a non-empty directory, the operation will throw the runtime DirectoryNotEmptyException. If the target of the path is a symbol link, then the symbolic link will be deleted, not the target of the link. The deleteIfExists(Path) method is identical to the delete(Path) method, except that it will not throw an exception if the file or directory does not exist, but instead it will return a boolean value of false. It will still throw an exception if the file or directory does exist but fails, such as in the case of the directory not being empty.

**Reading and Writing File Data with newBufferedReader() and newBufferedWriter()** The first method, Files.newBufferedReader(Path,Charset), reads the file specified at the Path location using a java.io.BufferedReader object. It also requires a Charset value to determine what character encoding to use to read the file. It may also be useful to know that Charset.defaultCharset() can be used to get the default Charset for the JVM. We now present an example of this method:
```java
Path path = Paths.get("/animals/gopher.txt");
try (BufferedReader reader = Files.newBufferedReader(path, Charset.forName("US-ASCII"))) {
   // Read from the stream
   String currentLine = null;
   while((currentLine = reader.readLine()) != null)
      System.out.println(currentLine);
} catch (IOException e) {
   // Handle file I/O exception...
}
```
The second method, Files.newBufferedWriter(Path,Charset), writes to a file specified at the Path location using a BufferedWriter. Like the reader method, it also takes a Charset value:
```java
Path path = Paths.get("/animals/gorilla.txt");
List<String> data = new ArrayList();
try (BufferedWriter writer = Files.newBufferedWriter(path,Charset.forName("UTF-16"))) {
   writer.write("Hello World");
} catch (IOException e) {
   // Handle file I/O exception...
}
```
This code snippet creates a new file with the specified contents, overwriting the file if it already exists. The newBufferedWriter() method also supports taking additional enum values in an optional vararg, such as appending to an existing file instead of overwriting it.

**Reading Files with readAllLines()** The Files.readAllLines() method reads all of the lines of a text file and returns the results as an ordered List of String values. The NIO.2 API includes an overloaded version that takes an optional Charset value. The following sample code reads the lines of the file and outputs them to the user:
```java
Path path = Paths.get("/fish/sharks.log");
try {
   final List<String> lines = Files.readAllLines(path);
   for(String line: lines) {
      System.out.println(line);
   }
} catch (IOException e) {
   // Handle file I/O exception...
}
```
As you might expect, the method may throw an IOException if the file cannot be read. Be aware that the entire file is read when readAllLines() is called, with the resulting String array storing all of the contents of the file in memory at once. Therefore, if the file is significantly large, you may encounter an OutOfMemoryError trying to load all of it into memory.
#### Understanding File Attributes
The Files class also provides numerous methods accessing file and directory metadata, referred to as file attributes. The one thing to keep in mind while reading file metadata in Java is that some methods are operating system dependent. For example, some operating systems may not have a notion of user-level permissions, in which case users can read only files that they have permission to read.

**Reading Common Attributes with isDirectory(), isRegularFile(), and isSymbolicLink()** The Files class includes three methods for determining if a path refers to a directory, a regular file, or a symbolic link. The methods to accomplish this are named Files.isDirectory(Path), Files.isRegularFile(Path), and Files.isSymbolicLink(Path), respectively. Java defines a regular file as one that contains content, as opposed to a symbolic link, directory, resource, or other non-regular file that may be present in some operating systems. If the symbolic link points to a real file or directory, Java will perform the check on the target of the symbolic link. In other words, it is possible for isRegularFile() to return true for a symbolic link, as long as the link resolves to a regular file.

**Checking File Visibility with isHidden()** The Files class includes the Files.isHidden(Path) method to determine whether a file or directory is hidden within the file system. In Linux or Mac-based systems, this is often denoted by file or directory entries that begin with a period character (.), while in Windows-based systems this requires the hidden attribute to be set. The isHidden() method throws the checked IOException, as there may be an I/O error reading the underlying file information.

**Testing File Accessibility with isReadable() and isExecutable()** The Files class includes two methods for reading file accessibility: Files.isReadable(Path) and Files.isExecutable(Path). This is important in file systems where the filename can be viewed within a directory, but the user may not have permission to read the contents of the file or execute it. Like the isDirectory(), isRegularFile(), and isSymbolicLink() methods, the isReadable() and isExecutable() methods do not throw exceptions if the file does not exist but instead return false.

**Reading File Length with size()** The Files.size(Path) method is used to determine the size of the file in bytes. The size returned by this method represents the conceptual size of the data, and this may differ from the actual size on the persistence storage device due to file system compression and organization. The size() method throws the checked IOException if the file does not exist or if the process is unable to read the file information.

**Managing File Modifications with getLastModifiedTime() and setLastModifiedTime()** The Files class provides the method Files.getLastModifiedTime(Path), which returns a FileTime object to accomplish this. The FileTime class is a simple container class that stores the date/time information about when a file was accessed, modified, or created. For convenience, it has a toMillis() method that returns the epoch time. The Files class also provides a mechanism for updating the last-modified date/time of a file using the Files.setLastModifiedTime(Path,FileTime) method. The FileTime class also has a static fromMillis() method that converts from the epoch time to a FileTime object. Both of these methods have the ability to throw a checked IOException when the file is accessed or modified.

**Managing Ownership with getOwner() and setOwner()** The Files.getOwner(Path) method returns an instance of UserPrincipal that represents the owner of the file within the file system. As you may have already guessed, there is also a method to set the owner, called Files .setOwner(Path,UserPrincipal). Note that the operating system may intervene when you try to modify the owner of a file and block the operation. For example, a process running under one user may not be allowed to take ownership of a file owned by another user. Both the getOwner() and setOwner() methods can throw the checked exception IOException in case of any issues accessing or modifying the file. In order to set a file owner to an arbitrary user, the NIO.2 API provides a UserPrincipalLookupService helper class for finding a UserPrincipal record for a particular user within a file system. In order to use the helper class, you first need to obtain an instance of a FileSystem object, either by using the FileSystems.getDefault() method or by calling getFileSystem() on the Path object with which you are working. Example:
```java
UserPrincipal owner = FileSystems.getDefault().getUserPrincipalLookupService().lookupPrincipalByName("jane");
Path path = ...
UserPrincipal owner = path.getFileSystem().getUserPrincipalLookupService().lookupPrincipalByName("jane");
```
#### Understanding Views
A view is a group of related attributes for a particular file system type. A file may support multiple views, allowing you to retrieve and update various sets of information about the file. To request a view, you need to provide both a path to the file or a directory whose information you want to read, as well as a class object, which tells the NIO.2 API method which type of view you would like returned. The Files API includes two sets of methods of analogous classes for accessing view information. The first method, Files.readAttributes(), returns a read-only view of the file attributes. The second method, Files.getFileAttributeView(), returns the underlying attribute view, and it provides a direct resource for modifying file information. Both of these methods can throw a checked IOException, such as when the view class type is unsupported. For example, trying to read Windows-based attributes within a Linux file system may throw an UnsupportedOperationException.

The commonly used attributes and view classes:
* **BasicFileAttributes BasicFileAttributeView** Basic set of attributes supported by all file systems
* **DosFileAttributes DosFileAttributeView** Attributes supported by DOS/ Windows-based systems
* **PosixFileAttributes PosixFileAttributeView** Attributes supported by POSIX systems, such as UNIX, Linux, Mac, and so on
#### Reading Attributes
The NIO.2 API provides a Files.readAttributes(Path,Class<A>) method, which returns read-only versions of a file view. The second parameter uses generics such that the return type of the method will be an instance of the provided class. All attributes classes extend from BasicFileAttributes; therefore it contains attributes common to all supported file systems. We now present a sample application that retrieves BasicFileAttributes on a file and outputs various metadata about the file:
```java
import java.io.IOException;
import java.nio.file.*;
import java.nio.file.attribute.BasicFileAttributes;

public class BasicFileAttributesSample {
   public static void main(String[] args) throws IOException {
      Path path = Paths.get("/turtles/sea.txt");
      BasicFileAttributes data = Files.readAttributes(path, BasicFileAttributes.class);

      System.out.println("Is path a directory? "+data.isDirectory());
      System.out.println("Is path a regular file? "+data.isRegularFile());
      System.out.println("Is path a symbolic link? "+data.isSymbolicLink());
      System.out.println("Path not a file, directory, nor symbolic link? "+ data.isOther());
      System.out.println("Size (in bytes): "+data.size());
      System.out.println("Creation date/time: "+data.creationTime());
      System.out.println("Last modified date/time: "+data.lastModifiedTime());
      System.out.println("Last accessed date/time: "+data.lastAccessTime());
      System.out.println("Unique file identifier (if available): "+ data.fileKey());
   }
}
```
* The **isOther()** method is used to check for paths that are not files, directories, or symbolic links, such as paths that refer to resources or devices in some file systems.
* The **lastAccessTime()** and **creationTime()** methods return other date/time information about the file.
* The **fileKey()** method returns a file system value that represents a unique identifier for the file within the file system or null if it is not supported by the file system.
#### Modifying Attributes
The NIO.2 API provides the Files.getFileAttributeView(Path,Class<V>) method, which returns a view object that we can use to update the file system–dependent attributes. We can also use the view object to read the associated file system attributes by calling readAttributes() on the view object. BasicFileAttributeView is used to modify a file’s set of date/time values. In general, we cannot modify the other basic attributes directly, since this would change the property of the file system object. For example, we cannot set a property to change a directory into a file, since this leaves the files in the future in an ambiguous state. Likewise, we cannot change the size of the object without modifying its contents. Since there is only one update method, setTimes(FileTime lastModifiedTime, FileTime lastAccessTime, FileTime createTime) in the BasicFileAttributeView class, and it takes three arguments, we need to pass three values to the method. The NIO.2 API allows us to pass null for any date/time value that we do not wish to modify. For example, the following line of code would change only the last modified date/time, leaving the other file date/time values unaffected:
```java
view.setTimes(lastModifiedTime,null,null);
```
#### Walking a Directory
The **Files.walk(path)** method returns a **Stream<Path>** object that traverses the directory in a depth-first, lazy manner. By lazy, we mean the set of elements is built and read while the directory is being traversed. For example, until a specific subdirectory is reached, its child elements are not loaded. This performance enhancement allows the process to be run on directories with a large number of descendants in a reasonable manner.

The following is an example of using a stream to walk a directory structure:
```java
Path path = Paths.get("/bigcats");
try {
   Files.walk(path).filter(p -> p.toString().endsWith(".java")).forEach(System.out::println);
} catch (IOException e) {
   // Handle file I/O exception...
}
```
You can see that the method also throws a somewhat expected IOException, as there could be a problem reading the underlying file system. By default, the method iterates up to Integer.MAX_VALUE directories deep, although there is an overloaded version of walk(Path,int) that takes a maximum directory depth integer value as the second parameter. The walk() method will not traverse symbolic links by default. Following symbolic links could result in a infinite cycle. If you have a situation where you need to change the default behavior and traverse symbolic links, NIO.2 offers the FOLLOW_LINKS option as a vararg to the walk() method. It is recommended to specify an appropriate depth limit when this option is used. Also, be aware that when this option is used, the walk() method will track the paths it has visited, throwing a FileSystemLoopException if a cycle is detected.
#### Searching a Directory
The **Files.find(Path,int,BiPredicate)** method behaves in a similar manner as the **Files.walk()** method, except that it requires the depth value to be explicitly set along with a BiPredicate to filter the data. Like walk(), find() also supports the FOLLOW_LINK vararg option. A BiPredicate is an interface that takes two generic objects and returns a boolean value of the form (T, U) -> boolean. In this case, the two object types are Path and BasicFileAttributes. In this manner, the NIO.2 automatically loads the BasicFileAttributes object for you, allowing you to write complex lambda expressions that have direct access to this object. We illustrate this with the following example:
```java
Path path = Paths.get("/bigcats");
long dateFilter = 1420070400000l;
try {
   Stream<Path> stream = Files.find(path, 10, (p,a) -> p.toString().endsWith(".java")
                         && a.lastModifiedTime().toMillis()>dateFilter);
  
   stream.forEach(System.out::println);
} catch (Exception e) {
   // Handle file I/O exception...
}
```
#### Listing Directory Contents
The method **listFiles()** that operated on a **java.io.File** instance and returned a list of File objects representing the contents of the directory that are direct children of the parent. Although you could use the Files. walk() method with a maximum depth limit of 1 to perform this same task, the NIO.2 API includes a new stream method, Files.list(Path), that does this for you. Consider the following code snippet, assuming that the current working directory is /zoo:
```java
try {
   Path path = Paths.get("ducks");
   Files.list(path).filter(p -> !Files.isDirectory(p)).map(p -> p.toAbsolutePath()).forEach(System.out::println);
} catch (IOException e) {
   // Handle file I/O exception...
}
```
#### Printing File Contents
The NIO.2 API in Java 8 now includes a Files.lines(Path) method that returns a Stream<String> object and does not use a lot of memory. The contents of the file are read and processed lazily, which means that only a small portion of the file is stored in memory at any given time.

Sample code:
```java
Path path = Paths.get("/fish/sharks.log");
try {
   Files.lines(path).forEach(System.out::println);
} catch (IOException e) {
   // Handle file I/O exception...
}
```
Taking things one step further, we can leverage other stream methods for a more powerful example:
```java
Path path = Paths.get("/fish/sharks.log");
try {
   System.out.println(Files.lines(path).filter(s -> s.startsWith("WARN ")).map(s -> s.substring(5)).collect(Collectors.toList()));
} catch (IOException e) {
   // Handle file I/O exception...
}
```
This sample code now searches for lines in the file that start with WARN, outputting everything after it to a single list that is printed to the user.