# Functions

The blocks within if statements, else statements, while statements, and so on should be one line long. Probably that line should be a function call. Not only does this keep the enclosing function small, but it also adds documentary value because the function called within the block can have a nicely descriptive name. Functions should not be large enough to hold nested structures. Therefore, the indent level of a function should not be greater than one or two.

***

Do one thing. If a function does only those steps that are one level below the stated name of the function, then the function is doing one thing. Another way to know that a function is doing more than “one thing” is if you can extract another function from it with a name that is not merely a restatement of its implementation

***

One level of abstraction per function.

***

We want every function to be followed by those at the next level of abstraction so that we can read the program, descending one level of abstraction at a time as we read down the list of functions.

***

My general rule for switch statements is that they can be tolerated if they appear only once, are used to create polymorphic objects, and are hidden behind an inheritance relationship so that the rest of the system can’t see them.

***

The ideal number of arguments for a function is zero (niladic). Next comes one (monadic), followed closely by two (dyadic). Three arguments (triadic) should be avoided where possible. More than three (polyadic) requires very special justification—and then shouldn’t be used anyway.

***

Output arguments are harder to understand than input arguments. When we read a function, we are used to the idea of information going in to the function through arguments and out through the return value.

***

There are two very common reasons to pass a single argument into a function. You may be asking a question about that argument, as in **boolean fileExists(“MyFile”)**. Or you may be operating on that argument, transforming it into something else and returning it. For example, **InputStream fileOpen(“MyFile”)** transforms a file name String into an InputStream return value. These two uses are what readers expect when they see a function. You should choose names that make the distinction clear, and always use the two forms in a consistent context.

***

Passing a boolean into a function is a truly terrible practice. It immediately complicates the signature of the method, loudly proclaiming that this function does more than one thing. It does one thing if the flag is true and another if the flag is false.

***

Dyads (2 arguments) aren’t evil, and you will certainly have to write them. However, you should be aware that they come at a cost and should take advantage of what mechanims may be available to you to convert them into monads. Maybe extract one of the arguments as an instance variable.

***

When a function seems to need more than two or three arguments, it is likely that some of those arguments ought to be wrapped into a class of their own.

***

In the case of a monad, the function and argument should form a very nice verb/noun pair. For example, **write(name)** is very evocative. Whatever this “name” thing is, it is being “written.” An even better name might be **writeField(name)**, which tells us that the “name” thing is a “field.” Using this form weencode the names of the arguments into the function name. For example, assertEqualsmight be better written as **assertExpectedEqualsActual(expected, actual)**. This strongly mitigates the problem of having to remember the ordering of the arguments.

***

Side effects are lies. Your function promises to do one thing, but it also does other hidden things.

***

Functions should either do something or answer something, but not both. Either your function should change the state of an object, or it should return some information about that object. Doing both often leads to confusion.

***

They confuse the structure of the code and mix error processing with normal processing. So it is better to extract the bodies of the *try* and *catch* blocks out into functions of their own.

***

Don’t Repeat Yourself aka DRY principle.

***

Dijkstra said that every function, and every block within a function, should have one entry and one exit. Following these rules means that there should only be one *return* statement in a function, no *break* or *continue* statements in a loop, and never, ever, any *goto* statements.