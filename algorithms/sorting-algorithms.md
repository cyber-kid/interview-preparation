# Sorting algorithms

## Bubble sort
The idea is to put the smallest item at the beginning of the array (index 0) and the largest item at the end (index nElems-1). The loop counter _**out**_ in the outer for loop starts at the end of the array, at nElems-1, and decrements itself each time through the loop. The items at indices greater than _**out**_ are always completely sorted. The _**out**_ variable moves left after each pass by in so that items that are already sorted are no longer involved in the algorithm.
The inner loop counter _**in**_ starts at the beginning of the array and increments itself each cycle of the inner loop, exiting when it reaches _**out**_. Within the inner loop, the two array cells pointed to by _**in**_ and _**in+1**_ are compared, and swapped if the one in _**in**_ is larger than the one in _**in+1**_.
#### Invariant
In the bubble sort, the invariant is that the data items to the right of _**out**_ are sorted. This remains true throughout the running of the algorithm. (On the first pass, nothing has been sorted yet, and there are no items to the right of _**out**_ because it starts on the rightmost element.)
#### Efficiency of the Bubble Sort
In general, where N is the number of items in the array, there are N-1 comparisons on the first pass, N-2 on the second, and so on. The formula for the sum of such a series is

```(N–1) + (N–2) + (N–3) + ... + 1 = N*(N–1)/2```

Thus, the algorithm makes about N^2⁄2 comparisons (ignoring the –1, which doesn’t make much difference, especially if N is large).
There are fewer swaps than there are comparisons because two items are swapped only if they need to be. If the data is random, a swap is necessary about half the time, so there will be about N^2⁄4 swaps. (Although in the worst case, with the initial data inversely sorted, a swap is necessary with every comparison.)
Both swaps and comparisons are proportional to N^2. Because constants don’t count in **Big O** notation, we can ignore the 2 and the 4 and say that the bubble sort runs in **O(N^2)** time.
## Selection Sort
The selection sort improves on the bubble sort by reducing the number of swaps necessary from **O(N^2)** to **O(N)**. Unfortunately, the number of comparisons remains **O(N^2)**. However, the selection sort can still offer a significant improvement for large records that must be physically moved around in memory, causing the swap time to be much more important than the comparison time. (Typically, this isn’t the case in Java, where references are moved around, not entire objects.)

The outer loop, with loop variable _ **out**-, starts at the beginning of the array (index 0) and proceeds toward higher indices.
The inner loop, with loop variable _**in**_, begins at _**out**_ and likewise proceeds to the right. At each new position of _**in**_, the elements a[in] and a[min] are compared. If a[in] is smaller, then _**min**_ is given the value of _**in**_. At the end of the inner loop, _**min**_ points to the minimum value, and the array elements pointed to by _**out**_ and _**min**_ are swapped.
#### Invariant
In the selection sort, the data items with indices less than or equal to _**out**_ are always sorted.
#### Efficiency of the Selection Sort
The selection sort performs the same number of comparisons as the bubble sort: __N*(N-1)/2__. For 10 data items, this is 45 comparisons. However, 10 items require fewer than 10 swaps. With 100 items, 4,950 comparisons are required, but fewer than 100 swaps. For large values of N, the comparison times will dominate, so we would have to say that the selection sort runs in **O(N^2)** time, just as the bubble sort did. However, it is unquestionably faster because there are so few swaps. For smaller values of N, the selection sort may in fact be considerably faster, especially if the swap times are much larger than the comparison times.
## Insertion Sort
In most cases the insertion sort is the best of the elementary sorts. It still executes in **O(N^2)** time, but it’s about twice as fast as the bubble sort and somewhat faster than the selection sort in normal situations. It’s also not too complex, although it’s slightly more involved than the bubble and selection sorts. It’s often used as the final stage of more sophisticated sorts, such as quicksort.

In the outer for loop, _**out**_ starts at 1 and moves right. It marks the leftmost unsorted data. In the inner while loop, _**in**_ starts at _**out**_ and moves left, until either _**temp**_ is smaller than the array element there, or it can’t go left any further. Each pass through the while loop shifts another sorted element one space right.
#### Invariant
At the end of each pass, following the insertion of the item from _**temp**_, the data items with smaller indices than _**outer**_ are partially sorted.
#### Efficiency of the Insertion Sort
On the first pass, it compares a maximum of one item. On the second pass, it’s a maximum of two items, and so on, up to a maximum of N-1 comparisons on the last pass. This is

```1 + 2 + 3 + … + N-1 = N*(N-1)/2```

However, because on each pass an average of only half of the maximum number of items are actually compared before the insertion point is found, we can divide by 2, which gives __N*(N-1)/4__

The number of copies is approximately the same as the number of comparisons. However, a copy isn’t as time-consuming as a swap, so for random data this algorithm runs twice as fast as the bubble sort and faster than the selection sort. In any case, like the other simple sort routines, the insertion sort runs in __O(N2)__ time for random data.

For data that is already sorted or almost sorted, the insertion sort does much better. When data is in order, the condition in the while loop is never true, so it becomes a
simple statement in the outer loop, which executes N-1 times. In this case the algorithm runs in __O(N)__ time. If the data is almost sorted, insertion sort runs in almost __O(N)__ time, which makes it a simple and efficient way to order a file that is only slightly out of order.

However, for data arranged in inverse sorted order, every possible comparison and shift is carried out, so the insertion sort runs no faster than the bubble sort.
## Comparing the Simple Sorts
The bubble sort very simple but it’s practical only if the amount of data is small.

The selection sort minimizes the number of swaps, but the number of comparisons is still high. This sort might be useful when the amount of data is small and swapping data items is very time-consuming compared with comparing them.
  
The insertion sort is the most versatile of the three and is the best bet in most situations, assuming the amount of data is small or the data is almost sorted. For larger amounts of data, quicksort is generally considered the fastest approach.
  
We’ve compared the sorting algorithms in terms of speed. Another consideration for any algorithm is how much memory space it needs. All three of the algorithms in this chapter carry out their sort in place, meaning that, besides the initial array, very little extra memory is required. All the sorts require an extra variable to store an item temporarily while it’s being swapped.
  