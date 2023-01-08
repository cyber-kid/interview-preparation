# Spring

The core of the Spring Framework is based on the principle of inversion of control. IoC is a technique that
externalizes the creation and management of component dependencies.

Spring’s DI implementation is based on two core Java concepts: JavaBeans and interfaces. When you
use Spring as the DI provider, you gain the flexibility of defining dependency configuration within your
applications in different ways (for example, XML files, Java configuration classes, annotations within your
code, or the new Groovy bean definition method). JavaBeans (POJOs) provide a standard mechanism for
creating Java resources that are configurable in a number of ways, such as constructors and setter methods.

Interfaces and DI are technologies that are mutually beneficial. Clearly designing and coding
an application to interfaces makes for a flexible application, but the complexity of wiring together an
application designed using interfaces is quite high and places an additional coding burden on developers.
By using DI, you reduce the amount of code you need to use an interface-based design in your application to
almost zero. Likewise, by using interfaces, you can get the most out of DI because your beans can utilize any
interface implementation to satisfy their dependency. The use of interfaces also allows Spring to utilize JDK
dynamic proxies (the Proxy pattern) to provide powerful concepts such as AOP for crosscutting concerns.

The benefits of using DI rather than a more traditional approach.
* **Reduced glue code** - One of the biggest plus points of DI is its ability to dramatically reduce the amount of code you have to write to glue the components of your application together.
* **Simplified application configuration** - By adopting DI, you can greatly simplify the process of configuring an application. In addition, DI makes it much simpler to swap one implementation of a dependency for another.
* **Ability to manage common dependencies in a single repository** - Using a traditional approach to dependency management of common services—for example, data source connection, transaction, and remote services—you create instances (or lookup from some factory classes) of your dependencies where they are needed (within the dependent class). This will cause the dependencies to spread across the classes in your application, and changing them can prove problematic. When you use DI, all the information about those common dependencies is contained in a single repository, making the management of dependencies much simpler and less error prone.
* **Improved testability** - When you design your classes for DI, you make it possible to replace dependencies easily. This is especially handy when you are testing your application.
* **Fostering of good application design** - Designing for DI means, in general, designing against interfaces. A typical injection-oriented application is designed so that all major components are defined as interfaces, and then concrete implementations of these interfaces are created and hooked together using the DI container. This kind of design was possible in Java before the advent of DI and DI-based containers such as Spring, but by using Spring, you get a whole host of DI features for free, and you are able to concentrate on building your application logic, not a framework to support it.

Spring Core alone, with its advanced DI capabilities, is a worthy tool, but where Spring really excels is in its myriad of additional features, all elegantly designed and built using the principles of DI. Spring provides features for all layers of an application, from helper application programming interfaces (APIs) for data access right through to advanced MVC capabilities. What is great about these features in Spring is that, although Spring often provides its own approach, you can easily integrate them with other tools in Spring, making these tools first-class members of the Spring family.
