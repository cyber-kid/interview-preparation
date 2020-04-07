# Data structures
* [Arrays](#arrays)
* [Stacks](#stacks)
* [Queues](#queues)
* [Priority Queues](#priority-queues)
* [Linked Lists](#linked-lists)
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
## Linked Lists
* The list itself consists of 2 parts, the list object and the inner container objects for the data;
* Each container object has a link to a next container object;
* The list object has a link to the first container object;
* The double-ended Linked List in the list object has links to the last and first elements. This kind of list allows to insert items in the begging and the end of the list;
* The doubly Linked List in each container object has the link to previous and next containers (right and left);

#### Add (addLast(), addFirst())
**O(1)** New item is inserted in the beginning (end) of the list. To do that only 2 links should be updated.
#### Add at index
**O(N)** To find the place for a new item you need to follow all the links to get to he needed index.
#### Element
**O(N)** To get the need item you need to follow the links from container to container to find it.
#### Delete
**O(1)** To delete an item form the beginning or the you just need to update 2 links.

