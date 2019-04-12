---
title : "Data Structures and Algorithm in Java"
date: 2019-04-12T01:37:56+08:00
lastmod: 2019-04-12T01:37:56+08:00
draft: false
tags: ["Java", "Interview"]
categories: ["Java fundemental knowledge"]
author: "Yang Zhang"
---
# Data Structure and Algorithm
#### Iterators
Suppose we want to traverse a list, performing some operations on certain links.

As users of a list, what we need is access to reference that can point to any arbitray link. This allows us to examine or modify the link. We should be able to increment the reference so we can traverse along the list, looking at each link in turn, and we should be able to access the link pointed to by the reference.

Objects containing references to items in data structure, used to traverse data structures, are commonly called iterators.

```Java
class ListIterator() {
    private Link current;
    ...
}
```
The current field contains a reference to the link the iterator currently points to.
```Java
public static void main(...){
    LinkedList theList = new LinkedList();
    ListIterator iter1 = theList.getIterator();

    Link alink = iter1.getCurrent();
    iter1.nextLink();
}
```
##### Iterator Methods
- reset(): Sets iterator to the start of the list
- nextLink(): Moves iterator to next link
- getCurrent(): Returns the link at iterator
- tEnd(): Returns true if iterator is at the end of list
- insertAfter(): Inserts a new link after iterator
- insertBefore(): Inserts a new link before iterator
- deleteCurrent(): Deletes the link at the link

The user can position the iterator using reset() and nextLink(), check if it's at the end of the list with atEnd(). 