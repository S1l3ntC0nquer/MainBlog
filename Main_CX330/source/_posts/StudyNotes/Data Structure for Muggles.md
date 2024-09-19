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

**All the following example will be shown in C Programming Language**.

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
	for (i = 0; i < n; i++)
	tempsum += list[i];
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
    for (i = 0; i < n; i++)
    tempsum += list[i];
    return tempsum;
}
```

![Time Complexity](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240919210100272.png)

On the other hand, a segment with execution time dependent on the instance characteristics will be like this.

```c
float rsum(float list[], int n)
{
    if (n)
    return rsum(list,n-1)+list[n-1];

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

For a list in **decreasing** order, the total compares will be $1+2+3+4+\dots+(n-1)=\frac{n(n-1)}{2}=\frac{1}{2}n^2-\frac{1}{2}n$.

**Best-case**

- a[0 : i-1] = [1, 2, 3, 4] and t = 5
    - 1 compare
- a[0 : i-1] = [1, 2, 3, …, i] and t = i + 1
    - 1 compare

For a list in **increasing** order, the total compares will be $1+1+1+\dots+1=n-1$​. 

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
