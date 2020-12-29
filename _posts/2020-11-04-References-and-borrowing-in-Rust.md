---
layout: post
title: References and borrowing in Rust
subtitle: How Rust enforces memory safety
cover-img: /assets/img/thor.jpg
thumbnail-img: /assets/img/batman.jpg
share-img: /assets/img/batman.jpg
tags: [rust,programming-language,memory]
comments: true
---

During the last month I have been studying Rust, and I have started to appreciate the power of its memory safety constraints.

**Rules**:

- You cannot have more than one mutable borrows to the same value with overlapping lifetimes.
- You cannot have a mutable and immutable borrows to the same value with overlapping lifetimes.
- You cannot move out a value from a borrowed context.

**Goal**:

- at any given point in time, you can only have one usable reference to modify a particular value.
- equivalent to: at any given point in time you cannot have 2 valid  references that modify the same variable.
- forbid dangling references.

The compiler does the best but sometimes it is too strict. NLL have improved the flexibility of the compiler to be more permissive but the problem is still not completely solved.

**Mental model** to understand how the compiler works:

- A  mutable reference to the value: `let r = &mut v`, invalidates all the previous references to `v`.
- Given: `let r1 = &mut v`. 
  - `let r2 = &mut v` invalidates the reference: `r1`
  - `let r2 = &mut (*r1)` does not invalidate `r1` because it refers to `*r1` not to `v` even if the pointed memory is the same

**Reborrows**

- A function parameter that is a mutable reference causes a reborrow of the receiver:

  - ```rust
    1: let mut v = vec![0];
    2: let r = &mut v;
    3: let r1 = &mut *r;
    r.push_back(1); // r is reborrowed as &mut (*r) invalidating all previous references to (*v) -> r1
    4: r1.push_back(2); // error because 
    ```

    The error is due to the invalidation of `r1` when `r.push_back(1);` is called

    The last call, forces the lifetime of `r1` to last from `3 to 4 ` and it overlaps with `r.push_back(1); ` that creates a new reference to `&mut (*r)`

    With NLL, without the last call the lifetime of `r` would have lasted only on line: `3`.

- Reborrows do not break the rule to have only one mutable reference at a time:

  - ```rust
    1: let mut v = vec![0];
    2: let r1 = &mut v;
    3: let r2 = &mut (*r1)
    4: r2.push_back(0);
    5: r1.push_back(0);
    ```

    Following our mental model it is ok because `r2` is a mutable reference to `*r1` while `r2` is a mutable reference to `v`.

    However, the last line invalidates `r2` because the re-borrow creates a reference to `*r1`.



**2-Phase Borrows And Mutable References**:

- ```rust
  let mut v = vec![1];
  let r = &mut v;
  r.push(r.len());
  ```

  Without the 2 phase borrows the code will not compile because the outer call creates a reborrow of `r`: `&mut *r`, and the inner call a new immutable reborrow of the same value: `& *r`. 

  With 2-phase borrows, the first reborrows is converted to `&mut2 *r` and later activated when the second reborrow is out of scope.

- ```rust
  let mut v = vec![1];
  let r = &mut v;
  r.push(v.len());
  ```

  Even with the 2-phase borrows it does not compile.

  The inner call, cause a reborrow of `v` that clashes with `r`

- ```rust
  let mut v = vec![1];
  let r = &mut v;
  r.push({r.push(0);0});
  ```

  Even with the 2-phase borrows it does not compile.

  The inner call requires a `&mut2` reborrows of `*r` that is not allowed by the 2-phase borrows since the outer call already created a `&mut2` reborrows of `*r`.

**References**:

- [Reborrows](https://stackoverflow.com/questions/51015503/why-does-re-borrowing-only-work-on-de-referenced-pointers)
- [Question on nested calls](https://users.rust-lang.org/t/nested-method-calls-with-existing-mutable-references/53345/2)

Have a nice day 🚀 

![Tree](/assets/img/tree.jpg)

