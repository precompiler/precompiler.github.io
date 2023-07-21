---
layout: post
title: "Rust Memory Management - Part 1"
date: 2023-07-20 15:00:00
categories: Programming
tags: Rust
---

> Rust is a very different language. It doesn't use a garbage collector, but users don't have to worry about allocating/freeing memory. Rust is blazing fast, but you may often need to wrestle with the compiler to get things right. It'll help a lot if you understand how Rust manages the memory.

If a variable is defined and it's size can be determined at compile time, it'll be stored in the stack


```rust
let x: i32 = 1;
let color = RGBColor{r: 100, g: 100, b: 100};
```

![var in stack](https://github.com/precompiler/precompiler.github.io/raw/master/_posts/rust-mem/p1.PNG)

If a variable is stored in the stack, and you try to assign it to another variable. If the type of the variable implements the [Copy](https://doc.rust-lang.org/std/marker/trait.Copy.html#) trait, it's value will be copied (stored in stack as well) and assigned to the new variable.

```rust
let x: i32 = 1;
let y = x;
println!("{}", x);
println!("{}", y);
```

![var in stack](https://github.com/precompiler/precompiler.github.io/raw/master/_posts/rust-mem/p2.PNG)


If the variable doesn't implement the Copy trait, it's value will be copied and assigned to the new variable, but the old variable will be marked as moved and will be dropped once it's out of scope.
```rust
let x: RGBColor{r: 100, g: 100, b: 100};
let y = x;
println!("{}", y);
println!("{}", x); // compile error, x has been marked as moved and will be dropped once it's out of scope
```

![var in stack](https://github.com/precompiler/precompiler.github.io/raw/master/_posts/rust-mem/p3.PNG)


