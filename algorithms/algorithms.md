# General information

### Big O
In computer science the way to say how efficient a computer algorithm is is called “Big O” notation. Big O notation uses the uppercase letter O, which you can think of as meaning “order of.”

Whenever you see one loop nested within another, such as those in the bubble sort, you can suspect that an algorithm runs in **O(N^2)** time. The outer loop executes N times, and the inner loop executes N (or perhaps N divided by some constant) times for each cycle of the outer loop. This means you’re doing something approximately __N*N__ or **N^2** times.
### Invariants
In many algorithms there are conditions that remain unchanged as the algorithm proceeds. These conditions are called invariants. Recognizing invariants can be useful in understanding the algorithm. In certain situations they may also be helpful in debugging; you can repeatedly check that the invariant is true, and signal an error if it isn’t.
### Recursion
Key features common to all recursive routines:
* It calls itself.
* When it calls itself, it does so to solve a smaller problem.
* There’s some version of the problem that is simple enough that the routine can solve it, and return, without calling itself.

Calling a method involves certain overhead. Control must be transferred from the location of the call to the beginning of the method. In addition, the arguments to the method and the address to which the method should return must be pushed onto an internal stack so that the method can access the argument values and know where to return. Another inefficiency is that memory is used to store all the intermediate arguments and return values on the system’s internal stack. This may cause problems if there is a large amount of data, leading to stack overflow. Recursion is usually used because it simplifies a problem conceptually, not because it’s inherently more efficient.