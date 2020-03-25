# Data structures
## Arrays
* Predefined size;
* Ordered;
### Insertion
O(1). New element is always inserted in last available cell.

O(N) if the array is sorted, new item should be compared with already existing one to find a proper place where ti put it and existing items should be shiftem to make room for a new one.
### Deletion
O(N). You need first to find the item and after deletion shift all items coming after deleted one to fill the empty cell.

O(N) if the array is sorted to find an item will take O(logN) but it is still need to shift the remaining ones to fill the empty cell.
## Search
O(N). In worst case you need to check all items.

O(logN) if the array is sorted we can use binary search.