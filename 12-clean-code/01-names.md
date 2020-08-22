# Names
The name of a variable, function, or class, should answer all the big questions. It should tell you why it exists, what it does, and how it is used. If a name requires a comment, then the name does not reveal its intent.

***

Noise words are redundant. The word variable should never appear in a variable name. The word table should never appear in a table name. How is NameString better than Name? Would a Name ever be a floating point number?

***

Use searchable names. Single-letter names and numeric constants have a particular problem in that they are not easy to locate across a body of text.

***

You also don’t need to prefix member variables with m_ anymore. And you should be using an editing environment that highlights or colorizes members to make them distinct.

***

Classes and objects should have noun or noun phrase names like *Customer*, *WikiPage*, *Account*, and *AddressParser*. Avoid words like *Manager*, *Processor*, *Data*, or *Info* in the name of a class. A class name should not be a verb.

***

Methods should have verb or verb phrase names like *postPayment*, *deletePage*, or *save*. Accessors, mutators, and predicates should be named for their value and prefixed with *get*, *set*, and *is* according to the javabean standard. When constructors are overloaded, use static factory methods with names that describe the arguments.

***

Pick one word for one abstract concept and stick with it. For instance, it’s confusing to have fetch, retrieve, and get as equivalent methods of different classes.

***

Don’t pick names that communicate implementation; choose names the reflect the level of abstraction of the class or function you are working in.

***

Use standard nomenclature where possible. Names are easier to understand if they are based on existing convention or usage. For example, if you are using the DECORATOR pattern, you should use the word Decorator in the names of the decorating classes. For example, *AutoHangupModemDecorator* might be the name of a class that decorates a *Modem* with the ability to automatically hang up at the end of a session.

***

The length of a name should be related to the length of the scope. You can use very short variable names for tiny scopes, but for big scopes you should use longer names.

***

Names should describe everything that a function, variable, or class is or does. Don’t hide side effects with a name. Don’t use a simple verb to describe a function that does more than just that simple action.