# Notes
When an object is instantiated, fields and blocks of code are initialized first. Then the constructor is run.

***

Java uses pass-by-value, which means that calls to methods create a copy of the parameters. Assigning new values to those parameters in the method doesn’t affect the caller’s variables.

***

Overloaded methods are methods with the same name but a different parameter list. Java calls the most specific method it can find. Exact matches are preferred, followed by wider primitives. After that comes autoboxing and finally varargs.

***

The order of initialization is the superclass: 
* static variables and static initializers in the order they appear; 
* instance variables and instance initializers in the order they appear; 
* constructor;

Note that static blocks initiated first in both classes and only then goes instance vars and constructors.

***

Virtual method is a method in which the specific implementation is not determined until runtime. In fact, all non-final, non-static, and non-private Java methods are considered virtual methods, since any of them can be overridden at runtime. Virtual method invocation means that Java will look at subclasses when finding the right method to call. This is true, even from within a method in the superclass.

***

A Java class that extends another class inherits all of its public and protected methods and variables. The first line of every constructor is a call to another constructor within the class using this() or a call to a constructor of the parent class using the super() call. If the parent class doesn’t contain a no-argument constructor, an explicit call to the parent constructor must be provided.

***

The Java compiler allows methods to be overridden in subclasses if certain rules are followed: 
* a method must have the same signature;
* be at least as accessible as the parent method;
* must not declare any new or broader exceptions;
* and must use covariant return types;

***

Abstract classes and methods may not be marked as final or private.

***

The equals() method implements an equivalence relation on non‐null object references:
* It is reflexive: For any non‐null reference value x, x.equals(x) should return true;
* It is symmetric: For any non‐null reference values x and y, x.equals(y) should return true if and only if y.equals(x) returns true;
* It is transitive: For any non‐null reference values x, y, and z, if x.equals(y) returns true and y.equals(z) returns true, then x.equals(z) should return true;
* It is consistent: For any non‐null reference values x and y, multiple invocations of x.equals(y) consistently return true or consistently return false, provided no information used in equals comparisons on the objects is modified;
* For any non‐null reference value x, x.equals(null) should return false;
* Within the same program, the result of hashCode() must not change. This means that you shouldn’t include variables that change in figuring out the hash code. In our hippo example, including the name is fine. Including the weight is not because hippos change weight regularly;
* If equals() returns true when called with two objects, calling hashCode() on each of those objects must return the same result. This means hashCode() can use a subset of the variables that equals() uses. You saw this in the card example. We used only one of the variables to determine the hash code;
* If equals() returns false when called with two objects, calling hashCode() on each of those objects does not have to return a different result. This means hashCode() results do not need to be unique when called on unequal objects;

***

clone() method has default implementation of shallow copy (creating copy of the object, copying the references). As interfaces doesn't contain implementation, it is placed in Object class (root) and made Cloneable as marker interface (without any methods). This makes the class which is Cloneable to just implement Cloneable interface without providing implementation for clone() method (which would not have been possible if clone method is not in Object class).

One more thing to note is that clone() method in Object class has access modifier protected, this is to restrict call to this method from outside by default. The class which is Cloneable can change the access modifier to public to allow access to outside.

***

Difference between Runtime.exit() and Runtime.halt()

You might have noticed so far that the difference between the two methods is that Runtime.exit() invokes the shutdown sequence of the underlying JVM whereas Runtime.halt() forcibly terminates the JVM process. So, Runtime.exit() causes the registered shutdown hooks to be executed and then also lets all the uninvoked finalizers to be executed before the JVM process shuts down whereas Runtime.halt() simply terminates the JVM process immediately and abruptly.

***

Narrowing primitive conversion

A narrowing primitive conversion may be used if all of the following conditions are satisfied:
* The expression is a constant expression of type int.
* The type of the variable is byte, short, or char.
* The value of the expression (which is known at compile time, because it is a constant expression) is representable in the type of the variable.
* Note that narrowing conversion does not apply to long or double. So, char ch = 30L; will fail although 30 is representable by a char.

Remember these rules for primitive types:
* Anything bigger than an int can NEVER be assigned to an int or anything smaller than int ( byte, char, or short) without explicit cast.
* CONSTANT values up to int can be assigned (without cast) to variables of lesser size ( for example, short to byte) if the value is representable by the variable.( that is, if it fits into the size of the variable).
* Operands of mathematical operators are ALWAYS promoted to AT LEAST int. (i.e. for byte * byte both bytes will be first promoted to int.) and the return value will be AT LEAST int.
* Compound assignment operators ( +=, *= etc)  have strange ways so read this carefully:

A compound assignment expression of the form E1 op= E2 is equivalent to E1 = (T)((E1) op (E2)), where T is the type of E1, except that E1 is evaluated only once.
Note that the implied cast to type T may be either an identity conversion or a narrowing primitive conversion.
For example, the following code is correct:

short x = 3;
x += 4.6;

and results in x having the value 7 because it is equivalent to:

short x = 3;
x = (short)(x + 4.6);

***

Class inheritance
* Variables are SHADOWED and methods are OVERRIDDEN.
* Which variable will be used depends on the class that the variable is declared of.
* Which method will be used depends on the actual class of the object that is referenced by the variable.
* In case of overriding, the return type of the overriding method must match exactly to the return type of the overridden method if the return type is a primitive.
* In case of objects, the return type of the overriding method may be a subclass of the return type of the overridden method (covariant return types).

***

Lambdas

There is a simple trick to identify invalid lambda constructs. When you write a lambda expression for a functional interface, you are essentially providing an implementation of the method declared in that interface but in a very consize manner.  Therefore, the lambda expression code that you write must contain all the pieces of the regular method code except the ones that the compiler can easily figure out on its own such as the parameter types, return keyword, and brackets. So, in a lambda expression, just check that all the information is there and that the expression follows the basic syntax -

(parameter list) OR single_variable_without_type -> { regular lines of code } OR just_an_expression_without_semicolon

***

Operator precedence
* The . (dot) operator has more precedence than the cast operator.
* Boolean operators have more precedence than =. (In fact, = has least precedence of all operators.)

***

A subclass can ALWAYS be assigned to a super class variable without any cast. It will always compile and run without any exception. For example, a Dog  "IS A" Animal, so you don't need to cast it.  But an Animal may not always be a Dog. So you need to cast it to make it compile and during the runtime the actual object referenced by animal should be a Dog  otherwise it will throw a ClassCastException.

***

These are the six facts on Strings:
* Literal strings within the same class in the same package represent references to the same String object.
* Literal strings within different classes in the same package represent references to the same String object.
* Literal strings within different classes in different packages likewise represent references to the same String object.
* Strings computed by constant expressions are computed at compile time and then treated as if they were literals.
* Strings computed at run time are newly created and therefore are distinct.
* The result of explicitly interning a computed string is the same string as any pre-existing literal string with the same contents.
* String s = null will print "null"
* replace() method creates a new String object. replace() method returns the same String object if both the parameters are same, i.e. if there is no change. "String".replace('g','g')=="String"
* compareTo() does a lexicographical (like a dictionary) comparison. It stops at the first place where the strings have different letters. If left hand side is bigger, it returns a positive number otherwise it returns a negative number. The value is equal to the difference of their unicode values. If there is no difference then it returns zero. In this case,  it will return ( 'h' - 'H') which is 32.  compareTo returns 0 exactly when the equals(Object) method would return true.

***

Data types supported by switch statement include the following:

* int and Integer
* byte and Byte
* short and Short
* char and Character
* String
* enum values

The values in each case statement must be compile-time constant values of the same data type as the switch value. This means you can use only literals, enum constants, or final constant variables of the same data type.

***

Overloading
1. The compiler always tries to choose the most specific method available with least number of modifications to the arguments.
2. Java designers have decided that old code should work exactly as it used to work before boxing-unboxing functionality became available.
3. Widening is preferred to boxing/unboxing (because of rule 2), which in turn, is preferred over var-args.

***

If the literal null or a variable reference pointing to null is used to check instanceof, the result is false.

***

The compilation check only applies when instanceof is called on a class. When checking whether an object is an instanceof an interface, Java waits until runtime to do the check. The reason is that a subclass could implement that interface and the compiler wouldn’t know it.

***

Abstract class can not be a functional interface.

***

ResultSetMetaData gives you the information about the result of executing a query. You can retrieve this object by calling getMetaData() on ResultSet. ResultSetMetaData contains several methods that tell you about the ResultSet. Some important methods are: getColumnCount(), getColumnName(int col), getColumnLabel(int col), and getColumnType(int col). Remember that the column index starts from 1.

***

Remember that HashMap supports adding null key as well as null values but ConcurrentHashMap does not. Inserting null key or null in a ConcurrentHashMap will throw a NullPointerException.

***

The keys of a TreeMap must be mutually comparable;

***

The synchronized keyword can be applied only to non-abstract methods that are defined in a class or a block of code appearing in a method or static or instance initialization blocks. It cannot be applied to methods in an interface.

***

Unlike popular belief an anonymous class can never be static. Even if created in a static method.

***

Inner class can extend the outer class.

***

Manipulating a stream doesn't manipulate the backing source of the stream.

***

Unlike a regular java class, you cannot access a non-final static field from an enum's constructor.

***

The auto-closeable variables defined in the try-with-resources statement are implicitly final. Thus, they cannot be reassigned.

***

A class (or an interface) can invoke a default method of an interface that is explicitly mentioned in the class's implements clause (or the interface's extends clause) by using the same syntax i.e. <InterfaceName>.super.<methodName>.However, this technique cannot be used to invoke a default method provided by an interface that is not directly implemented (or extended) by the caller.

***

Default method cannot be overridden by a static method. You can, however, redeclare a static method of a super interface as a default method in the sub interface.

***

All instances of wrapper classes and String class are immutable.

***

Also remember that if a super class declares that it implements Serializable, all its subclasses automatically become serializable.

***

Anonymous classes are implicitly final.

***

Calling Thread.sleep() does not release the lock.