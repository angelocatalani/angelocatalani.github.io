---
layout: post
title: References and borrowing in Rust
subtitle: How Rust enforces memory safety
cover-img: /assets/img/rust.jpg
thumbnail-img: /assets/img/rust.jpg
share-img: /assets/img/rust.jpg
tags: [rust,programming-language,memory]
comments: true
---

During the last month I have been studying Rust, and I have started to appreciate the power of its memory safety constraints.

**Rules**:

- You cannot have more than one mutable borrows to the same value with overlapping lifetimes.
- You cannot have mutable and immutable borrows  to the same value with overlapping lifetimes.
- You cannot move out a value from a borrowed context.

**Goal**:

- at any given point in time, you can only have one usable reference to modify a particular value.
- equivalent to: at any given point in time you cannot have 2 valid  references that modify the same variable.
- forbid dangling references.

The compiler does the best but sometimes it is too strict. NLL (non-lexical lifetimes) has improved the flexibility of the compiler to be more permissive but the problem is still not completely solved.

**Mental model** to understand how the compiler works:

- A  mutable reference to the value: `let r = &mut v`, invalidate all the previous references to `v`.

- A mutable reference can transfer temporary  the *ability* to modify the value it refer to. That *ability* will be given back as soon as the original reference is used again and the revorrow invalidated.

  Given: `let r1 = &mut v`. 
  - `let r2 = &mut v` invalidates the reference: `r1`
  - `let r2 = &mut (*r1)` does not invalidate `r1` , because `r1` is reborrowed. This means `r2` is valid as long as `r1` is not used later

**Reborrows**

The idea behind reborrows is that a mutable reference can transfer its power to another variable for a limited amount of time.

For example in the code below:

```rust
1: let mut v = vec![0];
2: let r = &mut v;
3: let r1 = &mut *r; // reborrows
4: r1.push(1);
5: r.push(2);
```

`r1` is reborrowed from `r` and it is valid as long as `r` is not used. It is crucial to note that reborrows do not invalidate the previous references to the same variable. 

However, a reborrowed reference cannot be used after the original reference is used again.

Tthe code below does not compile because `r1` reborrows temporarily from `r` but it is invalidated at line: 4 and cannot be used at line: 5.

```rust
1: let mut v = vec![0];
2: let r = &mut v;
3: let r1 = &mut *r; // reborrows
4: r.push(1);
5: r1.push(2);
```

Reborrows are useful for mutable references as function parameters because mutable references do not implement the `Copy` trait. This means that the mutable reference is no longer valid after it is passed as a parameter in the function.

Fortunately, mutable references are automatically reborrowed as function parameters. This is the reason why the following code works:

```rust
1: let mut v = vec![0];
2: let r = &mut v;
4: r.push(1); // r is reborrowed as &mut (*r) and not moved
5: r.push(2); // r is not moved by the previous call
```

Reborrows explain why the following code works even if we have 2 mutable references to the same variable:

```rust
1: let mut v = vec![0];
2: let r1 = &mut v;
3: let r2 = &mut (*r1) // r2 reborrows from r1 and it is valid as long as r1 is not modified
4: r2.push_back(0); // a new reborrows is implicitly created from r2 and it does not invalidate anything
5: r1.push_back(0); // r1 has not been invalidated since it has given temporary power to r2
// from this point we can no longer use r2 because it has been invalidated by the previous line that claims back the power previously given to r1
```

However, reborrows invalidate other previous reborrows to the same variable, following the classical rule of having only one mutable reference at a time.

This is why the code below does not work.

```rust
1: let mut v = vec![0];
2: let r = &mut v;
3: let r1 = &mut *r; // r1 reborrows r 
4: r.push(1); // r is reborrowed as &mut (*r) invalidating all previous references to (*r) -> r1
5: r1.push(2); // error: r1 has been invalidated by the previous call
```



**2-Phase Borrows And Mutable References**:

- ```rust
  let mut v = vec![1];
  let r = &mut v;
  r.push(r.len());
  ```

  Without the 2 phase borrows the code above will not compile because the outer call (`push()`) creates a reborrow of `r`: `&mut *r`, and the inner call another borrow of the reserved value.

  With 2-phase borrows, the first reborrow is converted to `&mut2 *r` and later activated when the second borrow is out of scope.

- ```rust
  let mut v = vec![1];
  let r = &mut v;
  r.push(v.len());
  ```

  Even with the 2-phase borrows it does not compile.

  The inner call, causes an inplicitly borrow of `v` that clashes with `r`

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

