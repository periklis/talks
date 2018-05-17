---
author:
- Periklis Tsirakidis
title: Why bother and learn Rust?
theme: solarized
---

# Why bother and learn Rust?

Periklis Tsirakidis & Stefan Lau

---

## About Rust Lang

- Systems Programming Language since 2004
- 1.0 since mid 2015, currently 1.26
- Promise I: Memory leak free programming
- Promise II: Safe multi-threading programming
- Promise III: Zero Cost Abstractions
- Promise I + II + III = Great Craftmanship

---

## How it keeps its promises

- Simple rules for pointer aliasing and data mutation
- Rules enforced through a sophisticated type system
- Front and mid end compiler support on top of LLVM
- Strong focus on programming language ergonomics
- Long research project history 2004 - 2008

---

# Let's dive in Rust

---

## Memory Safety I

No null pointer dereferences

----

### Null Pointer in C/C++: A billion dollar mistake

```c
obj* func_and_alloc() {...}

int main(int argc, char* argv[]) {
    obj* p = func_and_alloc();

    if (p != null) {
      // Do conditional stuff
    }

    // Do further stuff

    free(p); // Upps!!!
}

```

----

### In Rust we love RAII and Option< T >

```rust
fn func() -> Option<T> {...}

fn main () {
    match func() {
        Some(t) => { // do something with t },
        None => { // do something with nothng }
    } // Deallocate Option and T here
}

```
In Rust all types are sized and leave on the stack except you need the heap for e.g dynamic sized types.

---

## Memory Safety II

No dangling pointers

----

### C/C++: Dangling Pointer Strikes Back

```c
void borrow_to_friend(char* buf) {
    // process buffer
    free(buf);
}

int main(int argc, char* argv[]) {
    borrow_to_friend(arg[0]);

    // arg[0] now dangling
}

```

----

### Rust I: Move me, clone me, drop me

```rust
fn give_to_friend(book: String) { } // Drop book

fn main {
    // Move temporary to str
    let book = String::from("Learn Rust in 21 days");

    // Assignment moves not copy
    let same_book = book;

    // Clone other_book into other_book
    let other_book = same_book.clone();

    // Move str1 to arg str
    give_to_friend(same_book);
} // Drop other_book, empty same_book and empty book
```

[Link to the playground](https://play.rust-lang.org/?gist=db39948f76b97034cf841cc18b905abe&version=stable&mode=debug)

----

### Rust II: Borrow me a reference

```c
// str is string slice type: &[]
fn borrow_to_a_friend(str: str) {
    borrow_to_friends_friend(&str.as_bytes());
}

fn borrow_to_a_friends_friend(buf: &[u8]){}

fn main() {
    // Move temporary to book_title
    let book_title = String::from("Alice in Wonderland");

    // Borrow a string slice reference here
    borrow_to_a_friend(&book_title);
}
```

----

### Rust III: Take me mutable but nobody else

```rust
fn add_apocalypse_date(str: mut str) {}

fn main() {
    let mut book_title = String::from("Apocalypse");

    add_apocalypse_date(&mut book_title);

    // ERROR
    let other_apocalypse = &mut book_title;
}

```

---

## Memory Safety III

No buffer overruns

----

### C/C++: Programmers overrun buffers

```c
void check_str(char* buf, int len) {
    for(int i = 0; i <= len; i++) {
        // buf[i]
        // Programmer overruns the buffer on i = len
    }
}

int main(int arc, char* argv[]) {
    char* book = "Learn Overruns in 21 Days";
    char* other = "Smash me in 21 Days";

    // Overrun book char buffer
    check_str(book, 25);
}
```

----

### Rust: Take a slice leave the rest

```rust
fn check_str(str: &str) {}

fn main() {
    let book = "Learn Slicing in 21 Days";

    check_str(&book[0..4]);
}
```
[Link to playground](https://play.rust-lang.org/?gist=bfcaed6c6d1e081824e655c27f0f2c6c&version=stable&mode=debug)

---

## Safe Multi-Threading By Default

----

### Use the power of all your cores and sleep well

- Data races are another form of aliasing vs. mutation
- Simple rules:
  - Either you move contents into a thread
  - Or you borrow any immutable references
  - Or you borrow only once a mutable reference
- Let the compiler check the rules FTW

----

### Supported concurrency models

- Stdlib support (e.g. Sync, Send Traits)
- Message passing support (e.g. MPSC)
- Sychronization primitives (e.g. Mutex< T >, Arc< T >)
- Parallel data structures/Collections (e.g. Rayon)
- Actors model (e.g. actix)
- Futures, Async/Await (Still immature)


---

## Zero Cost Abstractions

----

### Functional Programming Concepts

- Pattern Matching for enums and structs
- Error handling & propagation
- Null safety
- Traits & Typeclasses w/o memory overhead


Note:
- Enums are named Sum Types
- Structs are named Product Types
- Arrays are homogenous Product Types
- Tuples are anonymous Product Types
- Traits are equivalent to Typeclasses

----

### Enums & Pattern Matching

```rust
enum Door {
  Opened,
  Closed,
  Broken(String)
}

fn check_door(door: &Door) -> () {
  match door {
    Door::Opened          => println!("Door is opened"),
    Door::Closed          => println!("Door is closed"),
    Door::Broken(burglar) =>
      println!("Door is broken by {:?}", burglar),
  }
}
```
[Link to the Playground](https://play.rust-lang.org/?gist=23582899db9202a06a6b3b6aad3ded20&version=stable&mode=debug)

----

### Error Handling & propagation

```rust
enum Door {
  Opened,
  Closed
}

fn is_valid(from: &Door, to: &Door) -> Result<String, String> {
  match (from, to) {
    (Door::Opened, Door::Closed) => Ok("Transition accepted".to_string()),
    (Door::Opened, Door::Opened) => Err("Transition not possiple".to_string()),
    (Door::Closed, Door::Opened) => Ok("Transition accepted".to_string()),
    (Door::Closed, Door::Closed) => Err("Transition not possible".to_string())
  }
}

fn check_door(from: &Door, to: &Door) -> Result<String, String> {
  let res = is_valid(from, to)?;
  Ok(res)
}
```
[Link to the Playground](https://play.rust-lang.org/?gist=165e12ca1e7d2c8f12a2b1af3673d51b&version=stable&mode=debug)

----

### Trait, Typeclasses & Composition


---

## Rust Ecosystem


---

## Why not only for C/C++ Developers?

- Wake up: We hit the end of Moore's Law by 2005
- Your PHP, Go, Ruby, etc. is as memory leak free as their impls/VMs/GCs
- Your PHP, Go, Ruby, etc. works on a costly abstraction above the OS/machine
- Use the power of a great cross-plattform std library and ecosystem


---

## Why not only for C/C++ Developers?

- Concurrent and parallel programming styles are going to stay, e.g.
  - Actors based concurrency
  - Futures/Tasks/Executors based concurrency
  - Map and reduce style data parallelism
  - SIMD
  - Offloading to GPU cores
- Rust enables productive writing of memory safe and concurrent/parallel applications

---

## Why important for web engineering?

- Rust enables the same productivity thanks to Cargo, Crates, Rustup, Rustc
- Targets futures/async/await like Scala/Finagle (Currently on Nightly only, est. release end 2017)
- Support for cross compilation targets: MSVC, IOS, ARM (IoT), WebAssembly & [more](https://forge.rust-lang.org/platform-support.html)

---

## Rust Key Facts

- Express Ownership explicitly!
- No dangling but borrowing down the stack!
- No overruns but trust your slice!
- No Exceptions but Result< T >!
- No null but Option< T >!
- No garbage collection but Box, Rc, Arc, RefCell!
- No green threads but sane standard library support!

---

# So long and thanks for the fish!

Periklis Tsirakidis & Stefan Lau

Github:
- github.com/periklis
- github.com/???

---

## Further reading

- [Jim Blady - Why Rust?](http://www.oreilly.com/programming/free/files/why-rust.pdf)
- [Aaron Turon - Standford Seminar on Rust Lang](https://www.youtube.com/watch?v=O5vzLKg7y-k)
- [Rust for Functional Programmers](http://science.raphael.poss.name/rust-for-functional-programmers.html)
- [Repository - Rust-Learning](https://github.com/ctjhoa/rust-learning)
- [Rust by Example](https://rustbyexample.com/)
- [Official Rust Land Documentation](https://doc.rust-lang.org/book/second-edition/ch01-00-introduction.html)
- [Official Cargo Guide](http://doc.crates.io/guide.html)
- [Official Rust Lang Reference](https://doc.rust-lang.org/stable/reference/)
- [RustBelt: Securing the Foundations of the Rust Programming Language (PDF)](https://www.mpi-sws.org/~dreyer/papers/rustbelt/paper.pdf)
