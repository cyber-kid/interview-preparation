# Data structures
## Arrays
* Predefined size;
* Ordered;
#### Insert
**O(1)** New element is always inserted in last available cell.

**O(N)** if the array is sorted, new item should be compared with already existing one to find a proper place where ti put it and existing items should be shiftem to make room for a new one.
#### Delete
**O(N)** You need first to find the item and after deletion shift all items coming after deleted one to fill the empty cell.

**O(N)** if the array is sorted to find an item will take O(logN) but it is still need to shift the remaining ones to fill the empty cell.
#### Search
**O(N)** In worst case you need to check all items.

**O(logN)** if the array is sorted we can use binary search.
## Stacks
* LIFO (Last-In-First-Out);
#### Push
**O(1)** New element is inserted int the top cell of the Stack.
#### Pop
**O(1)** The element is taken from the top cell of the Stack (and the element is deleted).
#### Peek
**O(1)** The same as Pop() but the element is not deleted.
## Queues
* FIFO (First-In-First-Out);
#### Add (Offer)
**O(1)** New element is inserted int the top cell of the Queue.
#### Remove (Poll)
**O(1)** The element is taken from the bottom cell of the Queue (and the element is deleted).
#### Element (Peek)
**O(1)** The same as Remove() but the element is not deleted.
#### Notes
1. To avoid the problem of not being able to insert more items into the queue even when it’s not full, the Front and Rear pointers wrap around to the beginning of the array. The result is a circular queue (sometimes called a ring buffer).
## Priority Queues
* FIFO (First-In-First-Out);
* Items are prioritized (sorted);
#### Add (Offer)
**O(N)** For new element to be inserted we need to compare it with other elements and if needed tshift them.
#### Remove (Poll)
**O(1)** The element is taken from the bottom cell of the Priority Queue (and the element is deleted).
#### Element (Peek)
**O(1)** The same as Remove() but the element is not deleted.

