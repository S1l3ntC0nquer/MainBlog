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

TODOTODOTODOTODOTODOTODOTODOTODOTODOTODOTODOTODOTODOTODOTODOTODOTODOTODOTODO

```c

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

