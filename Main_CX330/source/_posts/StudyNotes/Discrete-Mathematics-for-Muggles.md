---
title: Discrete Mathematics for Muggles
categories: StudyNotes
tags: Math
abbrlink: 6c0b3eb8
date: 2024-09-19 09:09:44
cover: https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/discrete_math.jpg
---

# Chapter 3 - Set Theory

## Terms Definitions

- Set: A well-defined collection of objects. Neither order nor repetition is relevant for a  general set.
- Element: Objects inside the set is called elements. They are said to be members of the set.
- Universe: 
- Subset: If every element of $C$ is an element of  $D$, then $C$ is a subset of $D$, which will be denoted by $C \subseteq D$.
- Proper subset: If, in addition, $D$ contains an element that is not in $C$, then $C$ is called a proper subset of $D$. It will be denoted by $C \subset D$.
- Cardinality: Also called size, the number of elements in a set, denoted by $|A|$.
- Equal: $C$ & $D$ are said to be equal when $C \subseteq D$ and $D \subseteq C$.

## Subset Relations

Let $A, B, C \subseteq U$.

1. If $A \subseteq B$ and $B \subseteq C$, then $A \subseteq C$.
2. If $A \subset B$ and $B \subseteq C$, then $A \subset C$.
3. If $A \subseteq B$ and $B \subset C$, then $A \subset C$.
4. If $A\subset B$ and $B\subset C$, then $A\subset C$.

## Null Set

- The null set, or empty set, is the (unique) set containing no elements. It is  denoted by $\phi$ or $\{\}$.
- $|\phi| = 0$, but $\{0\}\neq\phi$.
- $\phi\neq\{\phi\}$.

### Theorem 3.2

For any universe $U$, let $A \subseteq U$. Then $\phi \subseteq A$, and if $A\neq\phi$, then $\phi\subset A$.

## Power Set

- If $A$ is a set from universe $U$, the  power set of $A$, denoted $P(A)$, is the collection (or set) of all subsets of $A$.

For the set $C=\{1, 2, 3, 4\}$
$$
P(C)=\{\varnothing,\{1\},\{2\},\{3\},\{4\},\{1,2\},\{1,3\},\{1,4\},\{2,3\},\{2, 4\},\{3,4\},\{1,2,3\},\{1,2,4\}\{1,3,4\}\{2,3,4\}, C\}
$$
For any finite set $A$ with $|A|=n \geq 0$, we find that $A$ has $2^n$ subsets and that $|P(A)|=2^n$. For any $0 \leq k \leq n$, there are $\binom{n}{k}$ subsets of size $k$. Counting the subsets of $A$ according to the number, $k$, of elements in a subset, we have the combinatorial identity $\binom{n}{0}+\binom{n}{1}+\binom{n}{2}+\cdots+\binom{n}{n}=\sum_{k=0}^n\binom{n}{k}=2^n$, for $n \geq 0$.

## Gray Code

- A systematic way to represent the subsets of a  given nonempty set can be accomplished by  using a coding scheme known as a Gray code.

![Gray Code](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240919095457692.png)

## Pascal's Triangle

![Pascal's Triangle](https://raw.githubusercontent.com/CX330Blake/MyBlogPhotos/main/image/image-20240919095606249.png)

## Set of Numbers

1. $\mathbf{Z}=$ the set of integers $=\{0,1,-1,2,-2,3,-3, \ldots\}$
2. $\mathbf{N}=$ the set of nonnegative integers or natural numbers $=\{0,1,2,3, \ldots\}$
3. $\mathbf{Z}^{+}=$the set of positive integers $=\{1,2,3, \ldots\}=\{x \in \mathbf{Z} \mid x>0\}$
4. $\mathbf{Q}=$ the set of rational numbers $=\{a / b \mid a, b \in \mathbf{Z}, b \neq 0\}$
5. $\mathbf{Q}^{+}=$the set of positive rational numbers $=\{r \in \mathbf{Q} \mid r>0\}$
6. $\mathbf{Q}^*=$ the set of nonzero rational numbers
7. $\mathbf{R}=$ the set of real numbers
8. $\mathbf{R}^{+}=$the set of positive real numbers
9. $\mathbf{R}^*=$ the set of nonzero real numbers
10. $\mathbf{C}=$ the set of complex numbers $=\left\{x+y i \mid x, y \in \mathbf{R}, i^2=-1\right\}$
11. $\mathbf{C}^*=$ the set of nonzero complex numbers
12. For each $n \in \mathbf{Z}^{+}, \mathbf{Z}_n=\{0,1,2, \ldots, n-1\}$
13. For real numbers $a, b$ with $a<b,[a, b]=\{x \in \mathbf{R} \mid a \leq x \leq b\}$, $(a, b)=\{x \in \mathbf{R} \mid a<x<b\},[a, b)=\{x \in \mathbf{R} \mid a \leq x<b\}$, $(a, b]=\{x \in \mathbf{R} \mid a<x \leq b\}$. The first set is called a closed interval, the second set an open interval, and the other two sets half-open intervals.

## Closed Binary Operations

- The addition and multiplication of positive  integers are said to be closed binary operations on $\mathbf{Z}^{+}$.
- Two operands, hence the operation is called binary.
- Since $a+b \in \mathbf{Z}^{+}$when $a, b \in \mathbf{Z}^{+}$, we say that the binary operation of addition (on $\mathbf{Z}^{+}$) is closed.

## Binary Operations for Sets

For $A, B \subseteq U$, we define the following:

1. Union: $A \cup B = \{x\mid x\in A\vee x \in B\}$.
2. Intersection: $A \cap B = \{x\mid x\in A\wedge x \in B \}$.
3. Symmetric Difference: $A \triangle B=\{x\mid(x\in A\vee \in B)\wedge x\notin A\cap B\}=\{x\mid x \in A\cup B \wedge x \notin A \cap B\}$.

## Disjoint

Let $S, T \subseteq U$. The sets $S$ and $T$ are called disjoint, or mutually disjoint, when $S \cap T = \phi$.

### Theorem 3.3

If $S, T \subseteq U$, then $S, T \subseteq U$ are disjoint if and only if $S \cup T = S \triangle T$.

## Complement

- For a set $A \subseteq U$, the complement of $A$, denoted $U - A$, or $A$, is given by $\{x \mid x \in U \wedge x \notin A\}$.

- For $A, B \subseteq U$, the (relative) complement of $A$ in $B$, denoted $B - A$, is given by $\{x \mid x \in B \wedge x \notin A\}$.

## Subset, Union, Intersection, and  Complement

### Theorem 3.4

For any universe $U$ and any sets $A, B \subseteq U$, the following statements are equivalent:

1. $A \subseteq B$
2. $A \cup B = B$
3. $A\cap B = A$
4. $\overline{B}\subseteq\overline{A}$

## The Laws of Set Theory

For any sets $A, B$ and $C$ taken from a universe $U$.

$\text{Law of Double Complement}$

- $\overline{\overline{A}}=A$

$\text{DeMorgan's Laws}$

- $\overline{A\cup B}=\overline{A}\cap\overline{B}$
- $\overline{A\cap B}=\overline{A}\cup\overline{B}$

$\text{Commutative Laws}$

- $A\cup B=B\cup A$
- $A\cap B=B\cap A$

$\text{Associative Laws}$

- $A\cup(B\cup C)=(A\cup B)\cup C$
- $A\cap(B\cap C)=(A\cap B)\cap C$

$\text{Distributive Laws}$

- $A\cup(B\cap C)=(A\cup B)\cap (A\cup C)$
- $A\cap(B\cup C)=(A\cap B)\cup(A\cap C)$

$\text{Idempotent Laws}$

- $A\cup A=A$
- $A\cap A=A$

$\text{Identity Laws}$

- $A\cup\phi=A$
- $A \cap U = A$

$\text{Inverse Laws}$

- $A\cup\overline{A}=U$
- $A\cap\overline{A}=\phi$

$\text{Domination Laws}$

- $A\cup U=U$
- $A\cap\phi=\phi$

$\text{Absorption Laws}$

- $A\cup(A\cap B)=A$
- $A\cap(A\cup B)=A$

## Dual

Let $s$ be a (general) statement dealing with the equality of two set expressions.  Each such expression may involve one or more  occurrences of sets (such as $A$, $\overline{A}$, $B$, $\overline{B}$, etc.),  one or more occurrences of $\phi$ and $U$, and only  the set operation symbols $\cap$ and $\cup$. The dual of $s$, denoted $s^d$ , is obtained from $s$ by replacing (1)  each occurrence of $\phi$ and $U$ (in $s$) by $U$ and $\phi$, respectively; and (2) each occurrence of $\cap$ and $\cup$ (in $s$) by $\cup$ and $\cap$, respectively.

### Theorem 3.5

- Let $s$ denote a theorem dealing with the equality of two set expressions. Then $s^d$ , the dual of $s$, is also a theorem.
- This result cannot be applied to particular situations but only to results (theorems) about  sets in general

## Index Set

Let $I$ be a nonempty set and $U$ a universe. For each $i\in I$ let $A_i \subseteq U$. Then $I$ is called an index set (or set of indices), and each $i \in I$ is called an index.
