# Design principles
* [Creating JavaBeans](#creating-javabeans)
* [Object composition](#object-composition)
* [Encapsulating Data](#encapsulating-data)
* [Inheritance](#inheritance)

A design principle is an established idea or best practice that facilitates the software design process.
#### Creating JavaBeans
A JavaBean is a design principle for encapsulating data in an object in Java. Main rules:
* Properties are private. 
* Getter for non‐boolean properties begins with get.
* Getters for boolean properties may begin with is or get.
* Setter methods begin with set.
* The method name must have a prefix of set/get/is followed by the first letter of the property in uppercase and followed by the rest of the property name.
#### Object composition
In object‐oriented design, we refer to object composition as the property of constructing a class using references to other classes in order to reuse the functionality of the other classes. In particular, the class contains the other classes in the has‐a sense and may delegate methods to the other classes. One of the advantages of object composition over inheritance is that it tends to promote greater code reuse. By using object composition, you gain access to other classes and methods that would be difficult to obtain via Java’s single‐inheritance model.
#### Encapsulating Data
One fundamental principle of object‐oriented design is the concept of encapsulating data. In software development, encapsulation is the idea of combining fields and methods in a class such that the methods operate on the data, as opposed to the users of the class accessing the fields directly. In Java, it is commonly implemented with private instance members that have public methods to retrieve or modify the data, commonly referred to as getters and setters, respectively.
#### Inheritance
Inheritance is a mechanism in which one class acquires the property of another class. For example, a child inherits the traits of his/her parents. With inheritance, we can reuse the fields and methods of the existing class. Hence, inheritance facilitates Reusability and is an important concept of OOP.