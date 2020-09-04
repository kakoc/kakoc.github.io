+++
title = "MYOX: C compiler: 0"
description = "Lets implement C compiler from scratch with Rust"
+++

[MYOX: what does it mean?](@/blog/2020-06-29_myox.md)

## Foreword

Have you ever thought how the tool which you use every day actually works? At least on the basic level? In this series I want to demonstrate how to build C compiler from scratch. Actually there are a lot of stuff in the Internet about how to build interpreters. But the amount of work needed to build the interpreter vs compiler is extremely different. Compilers are much harder to implement, develop and maintain. This is why nowadays we mostly use interpreted languages. But it doesn't mean that compilers aren't more needed. It's a development, it's all about tradeoffs. Compilers can offer things which interpreters can not and vise versa. For the compilers introduction I redirect you to [wiki](https://en.wikipedia.org/wiki/Compiler).  
From the learning point of view I think that after understanding how to write the compiler you can easily understand how to write an intepreter almost without any additional knowledge. But if reverse that statement it won't be true. A lot of pieces will be missed. This is why we will write the compiler.  

## How

This series will be splitted into articles. Every article is a self sufficient chunk. The next article will modify the previous one + append something new.

## Base

Now lets create a project with a base structure. As I said earlier we will develope incrementally. Every new chapter it's a new verison of our compiler. We can do that with our version control system. But I prefer to have previous versions explicitely because it's easier to analyze with previous/next versions, make changes. From the Rust perspective I'm going to use [workspaces](https://doc.rust-lang.org/book/ch14-03-cargo-workspaces.html) for that.


```bash
cargo new c_compiler
cd c_compiler
rm -r src
```

Since we are going to use workspaces lets go to the `Cargo.toml`, remove all its contets and append:

```toml
[workspace]
members = [
  "<here we will place a name of the workspace>",
]
```

Next versions of our compilers will be created via: 

```bash
cargo new <name>
```