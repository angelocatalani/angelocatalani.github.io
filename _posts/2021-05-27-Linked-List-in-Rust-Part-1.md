---
layout: post
title: Ok(linked list) in Rust
subtitle: Learn Rust With Entirely Too Many Linked Lists
cover-img: /assets/img/linked-list2.png
thumbnail-img: /assets/img/linked-list.jpg
share-img: /assets/img/linked-list.jpg
tags: [rust,programming-language,memory]
comments: true
---

My journey to learning Rust took me to the mind-blowing revalatory book that describes how to implement a linked list:

[Learn Rust With Entirely Too Many Linked Lists](Learn Rust With Entirely Too Many Linked Lists).

**TL;DR**  After... the code compiled it worked correctly without any segmentation fault. This is the crucial distinguishing trait of `Rust` versus `C/C++`.

Even if I will summarise just the first part of the book, there are many `Rust` concepts I completly ignored before and helped me to have a better understanding of the `Rust` memory model.

**First implementation**

The recursive definition of linked list bring us to the classical approach below:

```rust
pub enum Node<T> {
  Empty,
  Elem(T, Node<T>),
}
```

The reason why the above definition does not compile highlights the difference between the `Stack` and `Heap` memory.

Data stored on the stack must have a known, fixed size at compile time.

This because the compiler access to stack data using pre-computed offset at compile time.

There was a proposal to introduce Variable Length Arrays in `C++11`, but eventually was dropped, because it would need large changes to the type system in C++.

Stack data is allocated when a new function is called and deallocated when the function returns.
It is local to the current thread.
It is safe to use.

But, in the example above the `enum Node` has a possibility infinite size that cannot be defined at compile time. So, we cannot use the `Stack` to allocate the next node.

Data with an unknown size at compile time, or a size that might change can only be stored on the `heap`.

A reference to the data stored in the heap is kept on the stack.

Heap data can persist even across multiple function call and is accessible to multiple threads.

However, It is dangerous to use and managing safely  heap data is why `Rust` exists.

In fact in languages such as `C/C++` dealing with heap data results often in segmentation fault, double free error and memory leaks. While memory leaks could happen also in Rust (even if you must do it on purpose), the (safe) Rust binary is always memory safe.

From the consideration above, the next design will use the `Box<T>` to allocate data on the heap:

~~~rust
```
pub enum Node<T> {
    Empty,
    Elem(T, Box<Node<T>>),
}
```
~~~

**Ok implementation**

The code now compiles but there are two design problems with this implementation related to the:

- Split of the linked list into sublist
- Allocation of the last empty node

The first issue is consequence of the fact our nodes are not uniformly allocated: the first node is always allocated on the stack while the following on the heap.

If we want to split the list into 2 sublist, we need to move the content of the `Box<Node<T>>` for some node,
from the heap to the stack, copying data from the heap to the stack.

This is a crucial limitation since one of the few strengths of linked list is that it is possible to create new sub-lists just by using references without copying data.

The following snippet shows the actual implementation of a split and why it is so inefficient:

```rust
fn show_split_sucks() {
  let last = FirstBadStack::Empty;
  let third = FirstBadStack::Elem(3, Box::new(last));
  let second = FirstBadStack::Elem(2, Box::new(third));
  // this is our current list:
  // 1 -> 2 -> 3 -> Empty
  let mut original_head = FirstBadStack::Elem(1, Box::new(second));
  let mut second_head = FirstBadStack::Empty;

  // we want to split the original list into:
  // 1 -> Empty
  // 2 -> 3 -> Empty

  let original_head_ref = &mut original_head;
  match original_head_ref {
    FirstBadStack::Empty => {}
    FirstBadStack::Elem(_, r) => {
      // second head is allocated on the stack.
      // here we take the value from the heap inside the box
      // to the stack variable: `second_head`.
      // this move of ownership requires copying the memory from the heap to the stack.
      // this copy can be avoided and it does not leverage the linked list
      // strength to create new sub-lists just by using references
      // without modifying/copy existing data.
      second_head = mem::replace(&mut (**r), FirstBadStack::Empty);
    }
  }
}
```



The second issue is that we need to allocate on the heap an empty node just to mark the end of the list.

To solve the first issue is enough to have an extra level of indirection to the nodes.
In this way all nodes are uniformly allocated.

```rust
pub struct List<T> {
    head: Box<Node<T>>
}
pub enum Node<T> {
    Empty,
    Elem(T, Box<Node<T>>),
}
```



To solve the second issue we need to distinguish between the node and  the link to the next node:
 the node cannot be empty while the link can be.



```rust
pub struct List<T> {
    head: Option<Box<Node<T>>>
}
pub struct Node<T> {
    value: T,
    next: Option<Box<T>>
}
```

The latter can be refactored with a type alias:

```rust
pub struct List<T> {
    head: Link
}
type Link<T> = Option<Box<T>>;
pub struct Node<T> {
    value: T,
    next: Link
}
```

**References**:

- [Learn Rust With Entirely Too Many Linked Lists](https://rust-unofficial.github.io/too-many-lists/)
- [The Rust Programming Language](https://doc.rust-lang.org/book/)

Have a nice day 🚀 

![My next bike](/assets/img/bike.png)

