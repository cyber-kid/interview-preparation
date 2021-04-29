# General information
* [Asymptotic Notations](#asymptotic-notations)
* [Common Complexity Scenarios](#common-complexity-scenarios)
* [Invariants](#invariants)
* [Recursion](#recursion)
### Asymptotic Notations
**Big O notation**

One of the asymptotic notations is the Big O notation. A function f(n) is considered O(g(n)) (read as big oh of g(n)) if there exists some positive real constant c and an integer n_0 > 0​, such that the following inequality holds for all n >= n_0: f(n) <= cg(n)

The following graph shows a plot of a function f(n) and cg(n) that demonstrates this inequality.
![](../0-images/big-o.png)
Note that the above inequality does not have to hold for all values of n. For n < n_0n​​, f(n) is allowed to exceed cg(n), but for all values of n beyond some value n_0, f(n) is not allowed to exceed cg(n).

What good is this? It tells us that for very large values of n, f(n) will be at most within a constant factor of g(n). In other words, f(n) will grow no faster than a constant multiple of g(n). Put yet another way, the rate of growth of f(n) is within constant factors of that of g(n).

**Big Omega**

Mathematically, a function f(n) is in Omega(g(n)) if there exists a real constant c > 0 and there exists n_0 > 0 such that f(n) >= cg(n) for n >= n_0. In other words, for sufficiently large values of n, f(n) will grow at least as fast as g(n).

The following graph shows an example of functions f(n) and g(n) that have a Big Omega relationship.
![](../0-images/big-omega.png)
**Big Theta**

If f(n) is in O(g(n)) and f(n) is also in Omega(g(n)) then it is in Theta(n). Formally, f(n) is in Theta(n) if there exist constants c_1 > 0, c_2 > 0 and an n_0 > 0 such that c_1*g(n) <= f(n) <= c_2*g(n) and n >= n_0. If f(n) is Theta(g(n)), then the two functions grow at the same rate, within constant factors.

So, Big Theta is an “asymptotically tight bound.” When a particular running time is Theta(g(n)), the running time is at least c_1*g(n) and at most c_2*g(n), i.e., it is sandwiched between two functions. Here’s how to think of Theta(n)Θ(n):
![](../0-images/big-theta.png)
### Common Complexity Scenarios
**Simple for loop with an increment of size 1**
```java
for (int x = 0; x < n; x++) {
    //statement(s) that take constant time
}
```
<ins>Running time complexity:</ins> T(n) = (2n+2)+cn = (2+c)*n+2. Dropping the leading constants = n + 2. Dropping lower order terms O(n).

<ins>Explanation</ins>: Java for loop increments the value x by 1 in every iteration from 0 until n-1 ([0, 1, 2, …, n-1]). So, n is first 0, then 1, then 2, …, then n-1. This means the loop increment statement x++ runs a total of n times. The comparison statement x < n ; runs n+1 times. The initialization x = 0; runs once. Summing them up, we get a running time complexity of the for loop of n + n + 1 + 1 = 2n + 2. Now, the constant time statements in the loop itself each run n times. Supposing the statements inside the loop account for a constant running time of c in each iteration, they account for a total running time of cn throughout the loop’s lifetime. Hence, the running time complexity is (2n+2) + cn.

**for loop with increments of size k**
```java
for (int x = 0; x < n; x+=k) {
    //statement(s) that take constant time
}
```
<ins>Running time complexity:</ins> 2 + n((2+c)/k) = O(n)

<ins>Explanation:</ins> The initialization x = 0; runs once. Then, x gets incremented by k until it reaches n. In other words, x is incremented to [0, k, 2k, 3k, …, (mk) < n]. Hence, the incrementation part x+=k of the for loop takes floor(n/k) time. The comparison part of the for loop takes the same amount of time and one more iteration for the last comparison. So this loop takes 1+1+n/k+n/k = 2 + 2n/k time. The statements in the loop itself take c*(n/k) time. Hence in total, 2 + 2n/k+cn/k = 2 + n((2+c)/k) times, which eventually give us O(n).

**Simple nested for loop**
```java
for (int i=0; i<n; i++){
    for (int j=0; j<m; j++){
        //Statement(s) that take(s) constant time
    }
}
```
<ins>Running time complexity:</ins> nm(2+c)+2+4n = O(nm)

<ins>Explanation:</ins> The inner loop is a simple for loop that takes (2m+2) + cm time and the outer loop runs it n times so it takes n((2m+2) + cm) time. Additionally, the initialization, increment and test for the outer loop take 2n+2 time so in total, the running time is n((2m+2) + cm)+2n+2 = 2nm+4n+cnm+2 = nm(2+c)+4n+2, which is O(nm).

**Nested for loop with dependent variables**
```java
for (int i=0; i<n; i++){
    for (int j=0; j<i; j++){
        //Statement(s) that take(s) constant time
    }
}
```
<ins>Running time complexity:</ins> O(n^2)

<ins>Explanation:</ins> The outer loop runs n times and for each time the outer loop runs, the inner loop runs i times. So, the statements in the inner loop do not run at the first iteration of the outer loop since i is 0. They run once at the second iteration of the outer loop since i is equal to 1 at that point, then they run twice, then thrice, until i is n-1. So, they run a total of c + 2c + 3c + ... + (n-1)c times = c*(((n-1)*((n-1)+1))/2) = (cn(n-1))/2. The initialization of j in the inner for loop runs once in each iteration of the outer loop. So, this statement incurs a running time of n. In the first iteration of the outer for loop, the j < i statement runs once, in the second iteration it runs twice, and so on. So, it incurs a total running time of 1 + 2 + ... + n = (n(n+1))/2. In each iteration of the outer loop, the j++ statement runs one less time than the j < i statement, so it accounts for a running time of 0 + 1 + 2 + ... + (n-1) = (n(n-1))/2. In total, the inner loop has a running time of (cn(n-1))/2 + (n(n+1))/2 +(n(n-1))/2+n. The outer loop initialization, test and increment operations account for a running time of 2n+2. That means the entire script has a running time of 2n + 2 + (cn(n-1))/2 + n + (n(n+1))/2 +(n(n-1))/2 which is O(n^2)

**Nested for loop With index modification**
```java
for (int i=0; i<n; i++){
    i*=2;
    for (int j=0; j<i; j++){
        // Statement(s) that take(s) constant time
    }
}
```
<ins>Running time complexity:</ins> O(n)

<ins>Explanation:</ins> A pattern is hard to decipher here. So, let’s simplify things. In the outer loop, i is doubled and then incremented each time. If we ignore the increment part, we will be slightly overestimating the number of iterations of the outer for loop. That is fine because we are looking for an upper bound on the worst-case running time (Big O). If i keeps doubling, it will get from 1 to n-1 in roughly log_2(n-1) steps. Therefore, the total number of iterations of the inner for loop is: 2^0 + 2^1 + 2^2 + ... + 2^(log_2(n-1)) = 2^(log_2(n-1)+1)-1=2^(log_2(n-1))2-1 = 2(n-1)-1 = 2n - 3. The contribution from the initialization, test, and increment operations of the outer for loop is 2log_2(n-1)+2. So, the total running time is 2n(2+c)-3c-4+2log_2(n-1)+2. The term linear in n dominates the others, and the time complexity is O(n).

**Loops with log(n) time complexity**
```java
i = //constant
n = //constant
k = //constant
while (i < n){
    i*=k;
    // Statement(s) that take(s) constant time
}
```
<ins>Running time complexity:</ins> O(log_k(n))
<ins>Explanation:</ins> A loop statement that multiplies/divides the loop variable by a constant such as the above takes log_k(n) time because the loop runs that many times.
### Invariants
In many algorithms there are conditions that remain unchanged as the algorithm proceeds. These conditions are called invariants. Recognizing invariants can be useful in understanding the algorithm. In certain situations they may also be helpful in debugging; you can repeatedly check that the invariant is true, and signal an error if it isn’t.
### Recursion
Key features common to all recursive routines:
* It calls itself.
* When it calls itself, it does so to solve a smaller problem.
* There’s some version of the problem that is simple enough that the routine can solve it, and return, without calling itself.

Calling a method involves certain overhead. Control must be transferred from the location of the call to the beginning of the method. In addition, the arguments to the method and the address to which the method should return must be pushed onto an internal stack so that the method can access the argument values and know where to return. Another inefficiency is that memory is used to store all the intermediate arguments and return values on the system’s internal stack. This may cause problems if there is a large amount of data, leading to stack overflow. Recursion is usually used because it simplifies a problem conceptually, not because it’s inherently more efficient.