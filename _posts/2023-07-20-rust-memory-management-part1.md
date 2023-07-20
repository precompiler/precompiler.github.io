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
