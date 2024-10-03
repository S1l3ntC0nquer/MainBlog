---
abbrlink: 7f04b563
categories:
  - - StudyNotes
cover: >-
  https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/24/8/Blog_cover%20(15)-min_7cec81cba2ba1adb3e5c3bedbe62b9f9.jpg
date: '2024-08-29T15:03:18.568316+08:00'
tags: []
title: Data Structures for Muggles
updated: '2024-08-29T15:39:10.351+08:00' 
---
# Prologue

**All the following example will be shown in C Programming Language or pseudo code.**

This is the note when I was taking the course in NCKU, 2024. Blablabla.....

Finally, I would like to declare that almost every photo I use comes from the handouts of my course at NCKU, provided by the professor. If any photo comes from another source, I will give proper credit in the caption or description of the image.

# Complexity

## Space Complexity

- The amount of memory that it needs to run to completion.
- $S(P)=c+S_P(I)$
- Total space requirement = Fixed space requirement + Variable space requirements
- We usually care more about the **Variable space requirements**.

For example, we have the following code like this.

**Iterative**

```c
float sum(float list[], int n)
{
    float tempsum=0;
    int i;
    for (i = 0; i < n; i++){
        tempsum += list[i];
    }
    return tempsum;
}
```

In this code, we DO NOT copy the array list, we just passes all parameters by value, so the variable space requirements $S_{sum}(I)=0$. Here's another example, let's look at the code first.

**Recursive** 

```c
float rsum(float list[], int n)
{
    if (n) return rsum(list,n-1)+list[n-1];
    return 0;
}
```

In this case, 1 recursive call requires **K** bytes for *2 parameters* and the *return address*. If the initial length of `list[]` is **N**, the variable space requirements $S_{rsum}(N)=N\times K$ bytes.

## Time Complexity

- The amount of computer time that it needs to run to completion.
- $T(P)=c+T_P(I)$
- Total time requirement = Compile time + Execution time
- We usually care more about the **Execution time**.

For example, this is a segment with execution time independent from the instance characteristics.

```c
float sum(float list[], int n)
{
    float tempsum=0;
    int i;
    for (i = 0; i < n; i++){
	    tempsum += list[i];
    }
    return tempsum;
}
```

![Time Complexity](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240919210100272.png)

On the other hand, a segment with execution time dependent on the instance characteristics will be like this.

```c
float rsum(float list[], int n)
{
    if (n){
         return rsum(list,n-1)+list[n-1];   
    }
    return 0;
}
```

![Time Complexity](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240919210321397.png)

For given parameters, computing time might not be the same. So, we should know the following cases.

- Best-case count: Minimum number of steps that can be executed.
- Worst-case count: Maximum number of steps that can be executed.
- Average count: Average number of steps executed.

Here, I'm going to use **insertion sort** to show you the difference between the best-case & the worst-case. If you don't know what it is, you can go check it out [here](https://www.geeksforgeeks.org/insertion-sort-algorithm/). Following is a snippet of the insertion sort algorithm in C.

```c
for (j = i - 1; j >= 0 && t < a[j]; j--){
	a[j + 1] = a[j];
}
```

![Insertion Sort](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240919211111351.png)

**Worst-case**

- a[0 : i-1] = [1, 2, 3, 4] and t = 0
    - 4 compares
- a[0 : i-1] = [1, 2, 3, …, i] and t = 0
    - i compares

For a list in **decreasing** order, the total compares will be 
$$
1+2+3+4+\dots+(n-1)=\frac{n(n-1)}{2}=\frac{1}{2}n^2-\frac{1}{2}n
$$
**Best-case**

- a[0 : i-1] = [1, 2, 3, 4] and t = 5
    - 1 compare
- a[0 : i-1] = [1, 2, 3, …, i] and t = i + 1
    - 1 compare

For a list in **increasing** order, the total compares will be
$$
1+1+1+\dots+1=n-1
$$

## Asymptotic Complexity

Sometimes determining exact step counts is difficult and not useful, so we will use the Big-O notation to represent space and time complexity. Here's a question, which algorithm is better? The one with $\frac{1}{2}n^2-\frac{1}{2}n$ or $n^2+3n-4$. The answer is, both of them are $O(n^2)$​, so the performances are similar.

Now, we jump back to the example in previous part. What is the time complexity of insertion sort in the Big-O representation?

- Worst case
    - $\frac{1}{2}n^2-\frac{1}{2}n\Rightarrow O(n^2)$
- Best case
    - $n-1\Rightarrow O(n)$

## Why Algorithm Is Important

This is a table of how various functions grow with $n$.

![How various function grow with n](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240919213305045.png)

As you can see, the exponential one grows exponentially, which means a little variation of n can increase a lot of computing time. In fact, on a computer performing 1 billion (109 ) steps per second. $2^{40}$ steps require 18.3 mins, $2^{50}$ steps require 13 days, and $2^{60}$​ steps require 310 years! So that's why improving algorithm may be more useful than improving hardware sometimes.

## Summary

- Space and time complexity
    - Best case, worst case, and average
    - Variable and fixed requirements
- Asymptotic complexity
    - Big-O
- Example:
    - Insertion sort
    - Selection sort

More information about complexity will be introduce when I take the algorithm course next year, or you can search for some sources if you really want to learn more about it.

# Arrays

## Intro

It's a collection of elements, stored at contiguous memory locations. Each element can be accessed by an **index**, which is the position of the element within the array. It's the most fundamental structure and the base of other structures.

## 1-D Array

![1-D Array](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240912192712480.png)

This is an 1-D array with length 10. We can access the first element by the index 0 (zero-indexing). But what should we do if the memory size we need is changing and not always the same? Will it be suitable for all kind of situation?

The answer is negative, and that's why we need something called **Dynamic Memory Allocation**. When we need an area of memory and do not know the size, we can call a function `malloc()` and request the amount we need. The following is an code implementation of a dynamic 1-D array storing integers.

```c
int size;
scanf("%d", &size); // Ask an arbitrary positive number as the size of the array
int *array = (int *)malloc(size * sizeof(int)); // Finish the memory allocation

// Do operations to the array here

free(array); // Free the memory after use
```

- The `free()` function deallocates an area of memory allocated by `malloc()`.

So, that's how to implement the dynamic 1-D array in C. Now, here comes another question. What if we want to store some data like the student ID and his exam scores? Should we store it in an 1-D array?

## 2-D Array

To answer the question above, we can choose 2-D array as the data structure to store something like that. C uses **array-of-array** representation to represent multidimensional array. The following is an example diagram to show how the things work in an 2-D array in C.

![2-D Array](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240912195121434.png)

To be more specific, it shows that the first dimension of the array (blue part) stores the pointers that pointing to the second dimension arrays (orange part), which are storing integer or other data types. Following is an example code of a dynamic 2-D array.

```c
int rows, cols;

// Get the dynamic size
scanf("%d", &cols);
scanf("%d", &rows);

// Allocate the first dimension (blue part in the diagram above)
int **array = (int **)malloc(cols * sizeof(int *));
	// For every rows, allocate the rows size for the row (orange part)
    for (int i = 0; i < cols; i++) {
        array[i] = (int *)malloc(rows * sizeof(int));
    }
```

## Row-Major and Column-Major

To know what is row-major & column-major, we can take a look at this example. If we have a matrix called A.
$$
A=\left[\begin{array}{lll}1 & 2 & 3 \\ 4 & 5 & 6 \\ 7 & 8 & 9\end{array}\right]
$$

- Column-major
  - $A=[1, 4, 7, 2, 5, 8, 3, 6, 9]$
  - Elements of the columns are contiguous in memory
  - Top to bottom, left to right
  - Matlab
  - Fortran
- Row-major
  - $A=[1, 2, 3, 4, 5, 6, 7, 8, 9]$
  - Elements of the rows are contiguous in memory
  - Left to right, top to bottom
  - C
  - C++

# Stacks

## Intro

Stack is also an fundamental structure that is opposite from Queue (will be mentioned later), it has some properties

- Last-In-First-Out (LIFO)
- Like a tennis ball box
- Basic operations
  - Push: Insert an element to the top of the stack 
  - Pop: Remove and return the top element in the stack
  - Peek/Top: Return the top element in the stack
  - IsEmpty: Return true if the stack is empty
  - IsFull: Return true if the stack is full

The following is the diagram of the **push** and **pop** operation of stacks.

![Push and Pop in Stack Operations. Source: https://www.programiz.com/dsa/stack](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/stack.png)

## Application: Balanced Brackets

- Check whether the brackets are balanced.
- Examples:
  - “]” → False
  - “(a+b)*c-d” → True
- Steps:
  - Scan the expression from left to right
  - When a left bracket, “[”, “{”, or “(”, is encountered, push it into stack.
  - When a right bracket, “]”, “}”, or “)”, is encountered, pop the top element from stack and check whether they are matched.
    - If the right bracket and the pop out element are not matched, return False.
  - After scanning the whole expression, if the stack is not empty, return False.

## Dynamic Stacks

If we want to dynamically allocate the size of the stack, we should still use `malloc()`. Here's an example using dynamic arrays.

- Headers and declarations

```c
int *stack;
int capacity = 1; // Initial capacity
int top = -1; // Representing empty stack
```

- Create stack

```c
void createStack() {
    stack = (int*)malloc(capacity * sizeof(int));
    if (stack == NULL) {
        printf("Memory allocation failed\n");
        exit(1);
    }
}
```

- Array doubling when stack is full

```c
void stackFull() {
    capacity *= 2;  // Double the capacity
    stack = (int*)realloc(stack, capacity * sizeof(int));
    if (stack == NULL) {
        printf("Memory allocation failed\n");
        exit(1);
    }
    printf("Stack expanded to capacity: %d\n", capacity);
}
```

- Check if the stack is empty

```c
int isEmpty() {
    return top == -1;
}
```

- Push

```c
void push(int value) {
    if (top == capacity - 1) {  // If the stack is full, double it
        stackFull();
    }
    stack[++top] = value;
}
```

- Pop

```c
int pop() {
    if (isEmpty()) {
        printf("Stack underflow. No elements to pop.\n");
        return -1; 
    }
    return stack[top--];
}
```

- Peek (or Top)

```c
int peek() {
    if (isEmpty()) {
        printf("Stack is empty.\n");
        return -1; 
    }
    return stack[top];
}
```

# Queues

## Intro

As I mentioned, queues are the opposite of stacks. That is because although they are both linear or ordered list, they have totally different properties.

- First-In-First-Out (FIFO)
- New elements are added at the **rear** end
- Old elements are deleted at the **front** end.
- Basic operations
  - Add: Insert element at the **rear** of a queue
  - Delete: Remove element at the **front** of a queue 
  - IsFull: Return true if the queue is full. (`rear == MAX_QUEUE_SIZE - 1`)
  - IsEmpty: Return true if the queue is empty (`front == rear`)

The following is the diagram of the insertion and deletion of the queues.

![Operations of the Queue](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240912211242955.png)

If now e have a queue that contains 5 elements but has the length 6 (null stands for no element there).
$$
Q = [A, B, C, D, E, null]
$$
After we **delete** an element from the front and **add** another one at the rear, it will be like this.
$$
Q = [null, B, C, D, E, F]
$$
The queue will be regarded as full since rear == MAX_QUEUE_SIZE, but we know that actually some empty space remains. So, how to fix it? We can think about when a customer at front of the line leaves, what will other customers do? 

They move forward. So, we should shift other elements forward after deletion. That way, the queue became
$$
Q = [B, C, D, E, F, null]
$$
Voila! We fix it! But if we do this at every deletion, the program will be way too slow. So here's the next question, what should we do to fix it? 

## Circular Queue

In a circular queue, the **front** and the **rear** will have some differences comparing with the normal queue.

- **front**: One position counterclockwise from the position of the front element.
- **rear**: The position of the rear element.

This is how it looks like.

![Circular Queue](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240912213303873.png)

The addition or insertion in an circular queue has the following steps:

1. Move **rear** one position clockwise: 
   `rear = (rear + 1) % MAX_QUEUE_SIZE`

2. Put element into `queue[rear]`

![Insertion in an Circular Queue](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240912213639625.png)

And the deletion steps are below:

1. Move **front** one position clockwise:
   `front = (front + 1) % MAX_QUEUE_SIZE`

2. Return and delete the value of `queue[front]`

![Deletion in an Circular Queue](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240912215134478.png)

In a circular queue, how to know a queue is full? We can think about when a queue is keep adding elements until it's full, what will the **front** and **rear** go? The answer is, when elements are keep being added to a circular queue, the **rear** will keep going clockwise and finally equals to the **front**. 

On the other hand, we can also use this to determine if the queue is empty. Because if the elements in the queue are keep being deleted, the **front** will keep moving clockwise until it equals to **rear**.

So if the condition to determine the full and empty are the same (front == rear), how can we tell which is which?

Here's some of the solutions, but there are still a lot of ways to conquer this.

1. Set a `LastMoveIsFront()` boolean value to recognize the last move is front or rear.
2. Set another variable `count` and modify it everytime we delete or add something.
3. Save an empty slot in the circular queue, so that when the queue is full, the rear will be at the position "just before the front" (because one empty slot is reserved), thus avoiding the confusion with the condition `rear == front` which indicates an empty queue.

# Linked Lists

## Intro

First, I want to talk about the deletion and insertion in sequential representation, which is mentioned a lot in the previous parts. When we deleting or inserting things in a queue or an array, there're always be some empty spaces created by the operations (See the picture below for details.) And that means we have to move many elements many times to maintain a sequential data structure.

![We need to move elements very often](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240919214955011.png)

So, to avoid this waste, we want to store data **dispersedly** in memory, just like the picture below.

![Store Data Dispersedly](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240919215401686.png)

But now, we are facing another problem. How do we know the order of the data if we store an sequential data like this? We use linked list! Let's start talking about linked list right away. Here is the features of it.

- In memory, list elements are stored in an arbitrary order.
- The fundamental unit is called a **node**.
- In a normal linked list, a node contains **data** and **link/pointer**.
- The **link** is used to point to the next element.

To answer the previous question, we need to talk about why linked list is called linked list. It's because every element in the list **linked** to the very next data, so we can know the sequence even thought we didn't save them sequentially. To let you better understand, the following graph is the illustration of how it works.

![Linked List](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240919220159098.png)

Another way that usually seen to draw a linked list is like this.

![Linked List by Arrows](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240919220259929.png)

## Operations

### Concept of Insertion

To insert K between B and C, we have to follow the steps below.

- Get an unused node *a*.
- Set the data field of a to K.
- Set the link of a to point to C.
- Set the link of B to point to *a*.

![Insertion of Linked List](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240919220608080.png)

### Concept of Deletion

To delete C in the linked list, we have to follow the steps below.

- Find the element precedes C.
- Set the link of the element to the position of D.

![Deletion in Linked List](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240919220726626.png)

### Define A Node in C

```c
typedef struct listNode *listPointer;
typedef struct {
    char data;
    listPointer link;
} listNode;
```

![Node](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240919221150498.png)

### Get(0)

```c
desiredNode = first; // get the first node
return desiredNode->data;
```

![Get(0)](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240919221125762.png)

### Get(1)

```c
desiredNode = first->link; // get the second node
return desiredNode->data;
```

![Get(1)](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240919221242223.png)

### Delete "C"

- Step 1: find the node before the node to be removed

```c
beforeNode = first->link;
```

![Step 1 of Deletion](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240919221635938.png)

- Step 2: save pointer to node that will be deleted

```c
deleteNode = beforeNode->link;
```

![Step 2 of Deletion](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240919221729705.png)

- Step 3: change pointer in *beforeNode*

```c
beforeNode->link = beforeNode->link->link;
free(deleteNode);
```

![Step 3 of Deletion](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240919221819687.png)

### Insert “K” before “A"

- Step 1: get an unused node, set its data and link fields

```c
MALLOC( newNode, sizeof(*newNode));
newNode->data = ‘K’;
newNode->link = first;
```

![Step 1](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240919221941849.png)

- Step2: update *first*

```c
first = newNode
```

![Step 2](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240919222029915.png)

### Insert “K” after “B”

![Insert After](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240919222334002.png)

```c
beforeNode = first->link;
MALLOC( newNode, sizeof(*newNode));
newNode->data = ‘K’;
newNode->link = beforeNode->link;
beforeNode->link = newNode;
```

## Circular Linked List

- The last node points to the first node.

![Circular Linked List](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240919222430801.png)

## Doubly Linked List

In general linked list, to find an element will always start at the beginning of the list, but it's not the case in doubly linked list!

- Right link: forward direction
- Left link: backward direction

![Doubly Linked List](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240919222723904.png)

### Node in Doubly Linked List

```c
typedef struct node *nodePointer;
typedef struct node{
    nodePointer llink;
    element data;
    nodePointer rlink;
};
```

![Node in Doubly Linked List](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240919222811745.png)

### Insert “K” after “B”

```c
newNode->llink = node;
newNode->rlink = node->rlink;
```

![Step 1](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/螢幕擷取畫面 2024-09-19 223329.png)

```c
node->rlink->llink = newNode;
node->rlink = newNode;
```

![Step 2](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/223556.png)

## Linked Stacks & Queues

- Linked stack
    - Push: add to a linked list
    - Pop: delete from a linked list
- Linked queue
    - AddQ: add to a rear of a linked list
    - DeleteQ: delete the front of a linked list

![Linked Stack](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240919223932702.png)

![Linked Queue](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240919224001363.png)

## Summary

- Linked lists
- Doubly linked lists
- Doubly circular linked lists
- Operations: Get, Insert, and Delete
- Applications:
    - Linked stacks
    - Linked queues

# Trees & Binary Trees

## Intro

Tree is an data structure that looks like a genealogy, so there's a lot of terms that similar to the terms in a family. Now, let's introduce some terms about the trees.

- Degree of a node
  -  number of subtrees
- Degree of a tree
  - maximum of the degree of the nodes in the tree
- Height (depth) of a tree
  - maximum level of any node in the tree
- Leaf (or Terminal)
  - nodes having degree zeros
- Internal node
  - not leaf & not root
- Children
  - roots of subtrees of a node X
- Parent
  -  X is a parent
- Sibling
  - children of the same parent
- Ancestor
  -  all the nodes along the path from the root to the node 

![Tree](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240920103633068.png)

## List Representation

If we want to represent a tree in a list format, we can use this mathod. Following is an example of a tree and its list representation.

![Tree](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240920104038203.png)

It's list representation will be like $\text{(A (B (E (K, L), F), C (G), D (H (M), I, J)))}$.

## Binary Trees

Binary trees are trees that have the following characteristics.

- Degree-2 tree
- Recursive definition
  - Each node has left subtree and/or right subtree.
  - Left subtree (or right subtree) is a binary tree.

This is an picture of a binary tree.

![Binary Tree](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240920104751233.png)

## Full Binary Trees

A full binary tree is a tree that has height $h$ and $2^h-1$ nodes. This is the graph of the full binary tree.

![Full Binary Tree](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240920105214535.png)

Please note that each full binary tree of the same height will only have one type. If we numbering the nodes in a full binary tree from **top to bottom**, **left to right**, then we will have the following properties.

- Let $n$ be the number of nodes in a FULL binary tree
  - Parent node of $i$ is node $floor(\frac{i}{2})$
  - Left child of node $i$ is node $2i$
  - Right child of node $i$ is node $2i+1$ 

## Skewed Binary Trees

- At least one node at each of first $h$ levels
- Minimum number of nodes for a height $h$

![Skewed Binary Tree](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240920105648862.png)

## Number of Nodes and Height

- Let $n$ be the number of nodes in a binary tree whose height is $h$.
  - $h\le n\le 2^h-1$
  - $\log_2(n+1)\le h$
  - The height **h** of a binary tree is at least $log_2(n+1)$.

## Complete Binary Trees

How to create a complete binary tree? Just follow the steps and check out the graph below.

1. Create a full binary tree which has at least $n$ nodes.
2. Number the nodes sequentially.
3. The binary tree defined by the node numbered $1$ through $n$ is the n-node complete binary tree. 

![Complete Binary Tree](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240920143648565.png)

## Binary Tree Representations

### Array Representation

- Number the nodes using the numbering scheme for a full binary tree. The node that is numbered `i` is stored in `tree[i]`.

![Array Representation](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240920144242000.png)

Since the number should be placed in the FULL binary tree, sometimes there will be some memory waste. For example, if we create a 4-level right-skewed binary tree, it will has the length 15, but only 4 being used.

![Worst Case for Required Space](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240920144539000.png)

### Linked Representation

- Each binary tree node is represented as an object whose data type is `TreeNode`.
- The space required by an `n` node binary tree is `n * (space required by one node)`.

```c
typedef struct node *treePointer;
typedef struct node{
    char data;
    treePointer leftChild, rightChild;
};
```

![Node](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240920145031720.png)

![Linked Representation](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/145204.jpg)

## Binary Tree Traversal

- Visiting each node in the tree exactly *once*
- A traversal produces a *linear order for the nodes* in a tree
  - LVR, LRV, VLR, VRL, RVL, and RLV
    - L: moving left
    - V: visiting the node
    - R: moving right
  - L**V**R: Inorder
  - **V**LR: Preorder
  - LR**V**: Postorder

### Inorder

```c
void inOrder(treePointer ptr){
    if (ptr != NULL){
        inOrder(ptr->leftChild);
        visit(ptr);
        inOrder(ptr->rightChild);
    }
}
```

![Inorder](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240926153006397.png)

### Preorder

```c
void preOrder(treePointer ptr){
    if (ptr != NULL){
        visit(t);
        preOrder(ptr->leftChild);
        preOrder(ptr->rightChild);
    }
}
```

![Preorder](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240926153135561.png)

### Postorder

```c
void postOrder(treePointer ptr){
    if (ptr != NULL){
        postOrder(ptr->leftChild);
        postOrder(ptr->rightChild);
        visit(t);
    }
}
```

![Postorder](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240926153303714.png)

### Difference Between Inorder, Preorder & Postorder

This is an illustration from GeeksforGeeks.

![Comparison between different order. Source: GeeksforGeeks](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/Preorder-from-Inorder-and-Postorder-traversals.jpg)

### Level-Order

- Visiting the nodes following the order of node numbering scheme (sequential numbering)

![Level-Order](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240926210505026.png)

### Iterative Traversal Using Stack

The following is an *inorder* example.

- Push **root** into stack
- Push the **left** into stack until reaching a **null** node
- Pop the **top** node from the stack
  - If there's no node in stack, break
- Push the **right child** of the pop-out node into stack
- Go back to step 2 from the **right child**

Here's the graph to let you more understand the workflow.

![Inorder Iterative Traversal Using Stack](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240926211302158.png)

### Convert Sequences Back to Trees

If we are given a sequence of data and we try to turn it back to a tree, how should we do? The following is an example of turnning a very short sequence data back into trees.

![Example](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240926211637534.png)

We can clearly see that if we are only given a sequence in 1 specific order, then we cannot know which side (left or right) is the original tree grows. But if we are given 2 different orders with 1 of them is **inorder**, then we can do it by the inference.

![image-20240926212026845](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240926212026845.png)

![Step 1](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240926212346555.png)

![Step 2](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240926212412466.png)

![Step 3](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240926212439440.png)

# Binary Search Trees

## Intro

The binary search tree, of course, it's an binary tree. But beside this, it has some unique properties so that we can call it a binary *search* tree.

- Each node has a **(key, value)** pair
- Keys in the tree are distinct
- For every node `x`
  - all keys in the **left** subtree are **smaller** than that in `x`
  - all keys in the **right** subtree are **larger** than that in `x`
- The subtrees are also binary search tree.

## Operations

### Search(root, key)

- k == root's key, terminate
- k < root's key, check left subtree
- k > root's key, check right subtree
- Time complexity is $O(height)=O(n)$, $n$ is number of nodes

![Search](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240926213639144.png)

#### Recursive

```c
if (!root) return NULL;
if (k == root->data.key)
	return root->data;
if (k < root->data.key)
	return search(root->leftChild, k);
return search(root->rightChild,k)
```

> Variable space requirement: $O(height) = O(n)$, $n$ is the number of nodes.

#### Iterative

```pseudocode
while(tree);
	if (k == tree->data.key)
		return tree->data;
	if (k < tree->data.key)
		tree = tree->leftChild;
	else
		tree = tree->rightChild;
return NULL
```

### Insert(root, key, value)

1. Search the tree
   - Matched: Do nothing
   - No match: Obtain `LastNode`, which is the last node during the search.
2. Add new node
   - Create a new node with (key, value)
   - If key > the key of `LastNode`, add the new node as **right child**
   - If key < the key of `LastNode`, add the new node as **left child**

![Insert](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/insert.jpg)

### Delete(key)

 There're 4 cases in the deletion.

- No element with delete key
- Element is a *leaf*
- Element is a degree-1 node
- Element is a degree-2 node

Since the 1st case means we don't do anything, we will start from the 2nd case.

#### Delete a leaf

![Delete a Leaf](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240926215517217.png)

#### Delete a Degree-1 Node

Link the single child of the `DeletedNode` to the parent of `DeletedNode`.

![Delete a Degree-1 Node](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/del.jpg)

#### Delete a Degree-2 Node

There two ways to do it. We can replace the deleted node with 

1. **Largest** pair in its **left** subtree
2. **Smallest** pair in its **right** subtree

> These two pairs must be leaf nodes or degree-one nodes. Why?

![Step 1](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image.jpg)

![Step 2](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/step2.jpg)

## Rank of a Node in a BST

Rank of node $x$

- The number of the nodes whose key values are smaller than $x$
- Position of $x$ **inorder**
- Like the **index** of an array

# Heaps

## Intro

Imagine that we have a program that records patients waiting in the hospital, if the data structure we use is a queue (FIFO), and a dying person comes, we cannot let him queue up to see the doctor first.

So that's why we need a priority queue! But how to implement one? Let's dive into the heaps right away!

![Source: https://www.udemy.com/course/js-algorithms-and-data-structures-masterclass/](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/1*A3D5yfJoI3G8qx-xQkwTEQ.png)

## Min Tree & Max Tree

First, we should know what is min tree and max tree to establish the concept we will use later.

![Min Tree & Max Tree](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240927011129370.png)

## Min Heap & Max Heap

![Min Heap & Max Heap](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240927011217928.png)

## Array Representation

![Array Representation of a Heap](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240927011438942.png)

## Operations

Here I use max heap for example, but it's very similar with the min heap so you can try it by yourself!

### Insert(key)

1. Create a new node while keep the heap being a complete binary tree.
2. Compare the key with the parent.
    - If the key < parent, pass
    - If the key > parent, switch place with parent and recursively compare the parent with parent's parent until it match the definition (key value in any node is the maximum value in the subtree)

![Step 1](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240927012600455.png)

![Step 2](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240927012631188.png)

![Step 3](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240927012652095.png)

This process is also called *bubbling up process* since it looks like a bubble goes up the water surface.

After we know how to insert things to a heap, we also care about it's speed. The time complexity of the inseriton is $O(\log{n})$, which $n$ is the heap size (height/level).

### Delete() or Pop()

- Removing the **root** of the heap
    - **Root** is the min element in a min heap
    - **Root** is the max element in a max heap

Following is the steps to do this operation

1. Removing the **last node** and inserting it into the **root**
2. Moving the node to a proper position
    - Find the child containing max key value and exchange the position

![Step 1](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240927013727411.png)

![Step 2](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240927013759069.png)

![Step 3](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/step3.pn.png)

![Step 4](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240927014228044.png)

And the time complextity of deletion is $O(\log{n})$, where $n$ is also the heap size.
