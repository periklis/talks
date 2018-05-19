---
author:
- Periklis Tsirakidis & Stefan Lau
title: Why bother and learn Rust?
theme: solarized
---

# Why bother and learn Rust?

Periklis Tsirakidis & Stefan Lau

---

## About Rust Lang

- Systems Programming Language since 2004
- 1.0 since mid 2015, currently 1.26
- No 2.0 ever! But Epoch 2018, ... releases!
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

[Playground link](https://play.rust-lang.org/?gist=db39948f76b97034cf841cc18b905abe&version=stable&mode=debug)

----

### Rust II: Borrow me a reference

```rust
// str is string slice type: &[]
fn borrow_to_a_friend(str: &str) {
    borrow_to_a_friends_friend(&str.as_bytes());
}

fn borrow_to_a_friends_friend(_buf: &[u8]){}

fn main() {
    // Move temporary to book_title
    let book_title = String::from("Alice in Wonderland");

    // Borrow a string slice reference here
    borrow_to_a_friend(&book_title);
}
```
[Playground link](https://play.rust-lang.org/?gist=4a072aca73115e14fc3091fa74542509&version=stable&mode=debug)
----

### Rust III: Take me mutable but nobody else

```rust
fn add_apocalypse_date(str: &mut str){
    // ERROR
    add_another_apocalypse_date(&mut str);
}

fn add_another_apocalypse_date(_str: &mut str) {}

fn main() {
    let mut book_title = String::from("Apocalypse");

    add_apocalypse_date(&mut book_title);
}
```
[Playground link](https://play.rust-lang.org/?gist=3a7a72deaea33480d83db654de9d07c6&version=stable&mode=debug)

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
[Playground link](https://play.rust-lang.org/?gist=bfcaed6c6d1e081824e655c27f0f2c6c&version=stable&mode=debug)

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
- Message passing support (e.g. MPSC, MPMC)
- Sychronization primitives (e.g. Mutex< T >, Arc< T >)
- Parallel data structures/Collections (e.g. Rayon)
- Actors model (e.g. actix)
- Futures, Async/Await (Still immature)

---

## Basic Types

----

### Numerical Types

Scalar
```rust
char
u8, u16, u32, ...
i8, i16, i32, ...
usize
```

Floating Point
```rust
f32, f64
```

----

### Compound types

Enums
```rust
enum Door { Opened, Closed, Broken(String) }
```

Tuples
```rust
(u32, u32)
```

Structs
```rust
struct Point { x: f64, y: f64 }
```

Fixed Size Arrays
```rust
let u: [u8; 16] = [0; 16];
```

----

### Collections

Vectors
```rust
let v: Vec<u32> = vec!(1, 2, 3);
```

String
```rust
let s: String = String::from("test");
```

HashMap
```rust
use std::collections::HashMap;

let mut s: HashMap<&str, &str> = HashMap::new();
s.insert("presenter1", "Peri");
s.insert("presenter2", "Stefan");
```

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
[Playground link](https://play.rust-lang.org/?gist=23582899db9202a06a6b3b6aad3ded20&version=stable&mode=debug)

----

### Structs & Pattern Matching

```rust
struct Point {
  x: f64,
  y: f64
}

fn get_x(point: Option<Point>) -> Option<f64> {
  match point {
    Some(Point { x, .. }) => Some(x),
    None              => None
  }
}

fn main() {
    let point = Point { x: 0.0, y: 1.0 };
    println!("{:?}", get_x(Some(point)));
}
```

[Playground link](https://play.rust-lang.org/?gist=502b76e8e8e8208e2f4cad368e921971&version=stable&mode=debug)

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
[Playground link](https://play.rust-lang.org/?gist=165e12ca1e7d2c8f12a2b1af3673d51b&version=stable&mode=debug)

----

### Traits I

```rust
trait NeedsFixing {
    fn needs_fixing(&self) -> bool;
}

enum Door {
  Opened,
  Closed,
  Broken(String)
}

impl NeedsFixing for Door {
    fn needs_fixing(&self) -> bool {
        match self {
            Door::Broken(_) => true,
            _ => false
        }
    }
}

struct Machine {
    running: bool,
    total_belts: u32,
    failed_belts: u32
}

impl NeedsFixing for Machine {
    fn needs_fixing(&self) -> bool {
        self.failed_belts > 0
    }
}

fn main() {
    println!("{}", Door::Opened.needs_fixing());
    println!("{}", Door::Broken(String::from("Stefan")).needs_fixing());
    println!("{}", Machine { running: true, total_belts: 2, failed_belts: 1 }.needs_fixing());
}
```

[Playground link](https://play.rust-lang.org/?gist=3a37c0444ccca650aa0b239f0b2deb3a&version=stable&mode=debug)

----

### Traits II

```rust
trait NeedsParts {
    type Part;

    fn needs_parts(&self) -> Vec<Self::Part>;
}

#[derive(Clone, Debug)]
enum DoorPart {
    Handle,
    Wood
}

enum Door {
  Opened,
  Closed,
  Broken(DoorPart)
}

impl NeedsParts for Door {
    type Part = DoorPart;

    fn needs_parts(&self) -> Vec<Self::Part> {
        match self {
            Door::Broken(DoorPart::Handle) => vec!(DoorPart::Handle),
            Door::Broken(DoorPart::Wood) => vec!(DoorPart::Wood),
            _ => vec!()
        }
    }
}

#[derive(Clone, Debug)]
struct Belt;

struct Machine {
    running: bool,
    total_belts: u32,
    failed_belts: u32
}

impl NeedsParts for Machine {
    type Part = Belt;

    fn needs_parts(&self) -> Vec<Self::Part> {
        vec![Belt; self.failed_belts as usize]
    }
}

fn main() {
    println!("{:?}", Door::Opened.needs_parts());
    println!("{:?}", Door::Broken(DoorPart::Handle).needs_parts());
    println!("{:?}", Machine { running: true, total_belts: 2, failed_belts: 2 }.needs_parts());
}
```

[Playground link](https://play.rust-lang.org/?gist=3a0c1d73d86ef07d09841378ab765a75&version=stable&mode=debug)

----

### Generic Types

```rust
fn identity<T>(item: T) -> T {
    item
}

trait Fuck {
    fn fuck(&self) -> ();
}

impl<T: std::fmt::Debug> Fuck for T {
    fn fuck(&self) -> () {
        println!("Fuck {:?} in particular", self);
    }
}

fn main() {
    println!("{:?}, {:?}, {:?}", identity(1), identity("test"), identity(Some("test")));
    1.fuck();
    "test".fuck();
    Some("test").fuck();
}
```

[Playground link](https://play.rust-lang.org/?gist=57c07af2a0a6860f2cb816eeda22733f&version=stable&mode=debug)

---

## Rust Ergonomics

- Type inference: [Example](https://play.rust-lang.org/?gist=b9a713507cc475b0988d620920972c12&version=stable&mode=debug)
- Lifetime elision: [Example](https://play.rust-lang.org/?gist=34871f0f2709cbe3428ccb33f1edfb4b&version=stable&mode=debug)
- Propagating errors: [Example](https://play.rust-lang.org/?gist=fb2028e83d08353dc74b5058a77d2531&version=stable&mode=debug)
- Compiler error messages: [Example1](https://play.rust-lang.org/?gist=a937854c15c4b4e9374090a0bd2cbf39&version=stable&mode=debug) [Example2](https://play.rust-lang.org/?gist=bae6bd9f85906d8cfa9249eb99cf6fba&version=stable&mode=debug)
- Strong focus on backwards compatibility

---

## Rust Ecosystem

----

### Cargo

- Package (crate) manager: `cargo update`
- Build system: `cargo build`
- Test runner: `cargo test`
- Documentation: `cargo doc`
- Extendable: E.g. `cargo-vendor`, `cargo-profiler`

----

### Crates.io

- [Link](https://crates.io)
- Central crate registry
- Example (important) crates:
  - serde: Serialization / Deserialization
  - regex: Regular Expressions
  - libc: FFI types
  - futures: Implementation of futures / streams
  - tokio: Runtime for async applications
  - rayon: Data-parallelism library


----

### Docs.rs

- [Link](https://docs.rs)
- Central website for documenation for crates
- Auto-generated documentation for every crate published to crates.io
- Documentation can be extended by using documentation comments

----

### Other

- Rustup: Toolchain manager
- Rust Language Server: Implementation of language server protocol for editors
- rustfmt: Code formatter
- [Rfcs](https://github.com/rust-lang/rfcs): Discussions about further development of the language

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

- Rust enables great productivity thanks to Cargo, Crates, Rustup
- Targets futures/async/await like Scala/Finagle (Currently on Nightly only, est. release mid 2018)
- Support for cross compilation targets: MSVC, IOS, ARM (IoT), WebAssembly & [more](https://forge.rust-lang.org/platform-support.html)

---

## Rust Key Facts

- FP Support with C-Model integration!
- Express Ownership explicitly!
- No dangling but borrowing down the stack!
- No overruns but trust your slice!
- No Exceptions but Result< T >!
- No null but Option< T >!
- No garbage collection but Box, Rc, Arc, RefCell!
- No green threads but sane standard library support!

---

## So long and thanks for the fish!

Periklis Tsirakidis & Stefan Lau

Github:
- github.com/periklis
- github.com/selaux

---

## Further reading

- [Jim Blady - Why Rust?](http://www.oreilly.com/programming/free/files/why-rust.pdf)
- [Aaron Turon - Standford Seminar on Rust Lang](https://www.youtube.com/watch?v=O5vzLKg7y-k)
- [Rust 2018 - An Epochal Release](https://www.infoq.com/presentations/rust-2018)
- [Rust for Functional Programmers](http://science.raphael.poss.name/rust-for-functional-programmers.html)
- [Repository - Rust-Learning](https://github.com/ctjhoa/rust-learning)
- [Rust by Example](https://rustbyexample.com/)
- [Official Rust Land Documentation](https://doc.rust-lang.org/book/second-edition/ch01-00-introduction.html)
- [Official Cargo Guide](http://doc.crates.io/guide.html)
- [Official Rust Lang Reference](https://doc.rust-lang.org/stable/reference/)
- [RustBelt: Securing the Foundations of the Rust Programming Language (PDF)](https://www.mpi-sws.org/~dreyer/papers/rustbelt/paper.pdf)
