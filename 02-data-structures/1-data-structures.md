# Data structures
* [Arrays](#arrays)
* [Stacks](#stacks)
* [Queues](#queues)
* [Priority Queues](#priority-queues)
* [Linked Lists](#linked-lists)
* [Binary tree](#binary-tree)
* [Red-Black tree](#red-black-tree)
* [2-3-4 Trees](#2-3-4-trees)
* [Hash Tables](#hash-tables)
* [Heaps](#heaps)
* [Graphs](#graphs)
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
## Binary tree
* There is only one root in a tree;
* There must be one (and only one!) path from the root to any other node;
* Every node in a tree can have at most two children;
* Left child should be less than parent;
* Right child should be greater or equal to the parent;
#### Find
**O(logN)** The time required to find a node depends on how many levels down it is situated. Might degrade to O(N) if the tree is not balanced.
#### Insert
**O(logN)** To insert a node means to find one that does not exist and update the references. Might degrade to O(N) if the tree is not balanced.
#### Traverse
**O(N)**
#### Delete
**O(logN)** To delete a node means to find the node and remove the references to it. Might degrade to O(N) if the tree is not balanced.
#### Notes on deleting a node
When you’ve found the node, there are three cases to consider:

**The node to be deleted is a leaf (has no children)** To delete a leaf node, you simply change the appropriate child field in the node’s parent to point to null, instead of to the node.

**The node to be deleted has one child** You want to “snip” the node out of this sequence by connecting its parent directly to its child. This process involves changing the appropriate reference in the parent (leftChild or rightChild) to point to the deleted node’s child.

**The node to be deleted has two children** To delete a node with two children, replace the node with its **inorder successor**. For each node, the node with the next-highest key is called its **inorder successor**, or simply its successor. If the right child of the original node has no left children, this right child is itself the successor.

To find a successor first, the program goes to the original node’s right child, which must have a key larger than the node. Then it goes to this right child’s left child (if it has one), and to this left child’s left child, and so on, following down the path of left children. The last left child in this path is the successor of the original node.

If **successor** is the right child of current, things are simplified somewhat because we can simply move the subtree of which successor is the root and plug it in where the deleted node was. This operation requires only two steps:
1. Unplug current from the rightChild field of its parent (or leftChild field if appropriate), and set this field to point to successor.
2. Unplug current’s left child from current, and plug it into the leftChild field of successor.

If **successor** is a left descendant of the right child of the node to be deleted, four steps are required to perform the deletion:
1. Plug the right child of successor into the leftChild field of the successor’s parent.
2. Plug the right child of the node to be deleted into the rightChild field of successor.
3. Unplug current from the rightChild field of its parent, and set this field to point to successor.
4. Unplug current’s left child from current, and plug it into the leftChild field of successor.

![](../0-images/successor-left-child.png)
## Red-Black tree
* The nodes are colored;
* During insertion and deletion, rules are followed that preserve various arrangements of these colors;
    1. Every node is either red or black;
    2. The root is always black;
    3. If a node is red, its children must be black (although the converse isn’t necessarily true);
    4. Every path from the root to a leaf, or to a null child, must contain the same number of black nodes;
* To fix rules violations you can either flip the colors or rotate the nodes;
#### The Efficiency of Red-Black Trees
Like ordinary binary search trees, a red-black tree allows for searching, insertion, and deletion in **O(logN)** time. Search times should be almost the same in the red-black tree as in the ordinary tree because the red-black characteristics of the tree aren’t used during searches. The only penalty is that the storage required for each node is increased slightly to accommodate the red-black color (a boolean variable). The times for insertion and deletion are increased by a constant factor because of having to perform color flips and rotations on the way down and at the insertion point. The advantage is that in a red-black tree sorted data doesn’t lead to slow **O(N)** performance.
## 2-3-4 Trees
* Three kinds of nodes are possible in a 2-3-4 tree: A 2-node has one key and two children, a 3-node has two keys and three children, and a 4-node has three keys and four children;
* In a search in a 2-3-4 tree, at each node the keys are examined. If the search key is not found, the next node will be child 0 if the search key is less than key 0; child 1 if the search key is between key 0 and key 1; child 2 if the search key is between key 1 and key 2; and child 3 if the search key is greater than key 2;
* Insertion into a 2-3-4 tree requires that any full node be split on the way down the tree, during the search for the insertion point. Splitting the root creates two new nodes; splitting any other node creates one new node. The height of a 2-3-4 tree can increase only when the root is split;
* The 2-3-4 tree wastes space because many nodes are not even half full;
#### Efficiency of 2-3-4 Trees
For 2-3-4 trees the increased number of items per node tends to cancel out the decreased height of the tree. The search times for a 2-3-4 tree and for a balanced binary tree such as a red-black tree are approximately equal, and are both **O(logN)**.
## Hash Tables
* A hash table is based on an array.
* The range of key values is usually greater than the size of the array;
* A key value is hashed to an array index by a hash function;
* The hashing of a key to an already-filled array cell is called a collision. Collisions can be handled in two major ways: open addressing and separate chaining;
* There are three kinds of open addressing: linear probing, quadratic probing, and double hashing;
* In linear probing the step size is always 1, so if x is the array index calculated by the hash function, the probe goes to x, x+1, x+2, x+3, and so on;
* In quadratic probing the offset from x is the square of the step number, so the probe goes to x, x+1, x+4, x+9, x+16, and so on;
* In double hashing the step size depends on the key and is obtained from a secondary hash function. If the secondary hash function returns a value s in double hashing, the probe goes to x, x+s, x+2s, x+3s, x+4s, and so on, where s depends on the key but remains constant during the probe;
* It’s crucial that an open-addressing hash table does not become too full;
* In separate chaining, each array element consists of a linked list. All data items hashing to a given array index are inserted in that list;
* A load factor of 1.0 is appropriate for separate chaining;
* Hash table sizes should generally be prime numbers. This is especially important in quadratic probing and separate chaining;
#### Hashing Efficiency
The insertion and searching in hash tables can approach **O(1)** time. If no collision occurs, only a call to the hash function and a single array reference are necessary to insert a new item or find an existing item.
## Heaps
* A heap is an efficient implementation of an ADT priority queue;
* A heap is a binary tree with these characteristics:
    * It’s complete. This means it’s completely filled in, reading from left to right across each row, although the last row need not be full;
    * It’s (usually) implemented as an array;
    * Each node in a heap satisfies the heap condition, which states that every node’s key is larger than (or equal to) the keys of its children.
* The largest item is always in the root;
* Each node has a key less than its parents and greater than its children;
* An item to be inserted is always placed in the first vacant cell of the array and then trickled up to its appropriate position;
* When an item is removed from the root, it’s replaced by the last item in the array, which is then trickled down to its appropriate position;
* The trickle-up and trickle-down processes can be thought of as a sequence of swaps, but are more efficiently implemented as a sequence of copies;
#### The Efficiency of Heaps
A heap offers removal of the largest item, and insertion, in O(N*logN) time.
## Graphs
* Graphs consist of vertices connected by edges;
* An adjacency matrix is a two-dimensional array in which the elements indicate whether an edge is present between two vertices;
* An adjacency list is an array of lists (or sometimes a list of lists). Each individual list shows what vertices a given vertex is adjacent to;
* The two main search algorithms are depth-first search (**DFS**) and breadth-first search (**BFS**);
* Rules for DFS:
    * If possible, visit an adjacent unvisited vertex, mark it, and push it on the stack;
    * If you can’t follow Rule 1, then, if possible, pop a vertex off the stack;
    * If you can’t follow Rule 1 or Rule 2, you’re done;
* Rules for BFS:
    * Visit the next unvisited vertex (if there is one) that’s adjacent to the current vertex, mark it, and insert it into the queue;
    * If you can’t carry out Rule 1 because there are no more unvisited vertices, remove a vertex from the queue (if possible) and make it the current vertex;
    * If you can’t carry out Rule 2 because the queue is empty, you’re done;
* A minimum spanning tree (**MST**) consists of the minimum number of edges necessary to connect all a graph’s vertices;
* In a directed graph, edges have a direction (often indicated by an arrow);
* A topological sorting algorithm creates a list of vertices arranged so that a vertex A precedes a vertex B in the list if there’s a path from A to B;
* A topological sort can be carried out only on a DAG, a directed acyclic (no cycles) graph;
* In a weighted graph, edges have an associated number called the weight, which might represent distances, costs, times, or other quantities;
* The minimum spanning tree in a weighted graph minimizes the weights of the edges necessary to connect all the vertices;
* The shortest-path problem (**SPP**) in a non-weighted graph involves finding the minimum number of edges between two vertices;
* Solving the shortest-path problem for weighted graphs yields the path with the minimum total edge weight;
* The shortest-path problem for weighted graphs can be solved with **Dijkstra’s** algorithm;
* The algorithms for large, sparse graphs generally run much faster if the adjacency- list representation of the graph is used rather than the adjacency matrix;
* The all-pairs shortest-path problem is to find the total weight of the edges between every pair of vertices in a graph. Floyd’s algorithm can be used to solve this problem;
#### Dijkstra’s Algorithm
Let the node at which we are starting be called the initial node. Let the distance of node Y be the distance from the initial node to Y. Dijkstra's algorithm will assign some initial distance values and will try to improve them step by step.

1. Mark all nodes unvisited. Create a set of all the unvisited nodes called the unvisited set.
2. Assign to every node a tentative distance value: set it to zero for our initial node and to infinity for all other nodes. Set the initial node as current.[14]
3. For the current node, consider all of its unvisited neighbours and calculate their tentative distances through the current node. Compare the newly calculated tentative distance to the current assigned value and assign the smaller one. For example, if the current node A is marked with a distance of 6, and the edge connecting it with a neighbour B has length 2, then the distance to B through A will be 6 + 2 = 8. If B was previously marked with a distance greater than 8 then change it to 8. Otherwise, the current value will be kept.
4. When we are done considering all of the unvisited neighbours of the current node, mark the current node as visited and remove it from the unvisited set. A visited node will never be checked again.
5. If the destination node has been marked visited (when planning a route between two specific nodes) or if the smallest tentative distance among the nodes in the unvisited set is infinity (when planning a complete traversal; occurs when there is no connection between the initial node and remaining unvisited nodes), then stop. The algorithm has finished.
6. Otherwise, select the unvisited node that is marked with the smallest tentative distance, set it as the new "current node", and go back to step 3.

When planning a route, it is actually not necessary to wait until the destination node is "visited" as above: the algorithm can stop once the destination node has the smallest tentative distance among all "unvisited" nodes (and thus could be selected as the next "current").