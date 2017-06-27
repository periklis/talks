---
author:
- Periklis Tsirakidis
title: Why bother and learn Rust?
---

# Why bother and learn Rust?

Periklis Tsirakidis

---

## Short Introduction to Rust Lang

- Systems Programming Language since 2004
- 1.0 since mid 2015, currently 1.18
- Promise I: Memory leak free programming
- Promise II: Safe multi-threading programming
- Promise III: Zero Cost Abstractions

---

## How Rust succeeds on its promises?

- Simple rules about pointer aliasing and data structure mutation
- Rules enforced through a sophisticated type system
- Enhanced front end compiler support on top of LLVM
- Strong focus on programming language ergonomics
- Long research project history 2004 - 2008

---

# Let's dive in Rust

---

## Memory Safety I

No null pointer dereferences

----

### Null Pointer in C/C++: A billion dollar mistake

```
int* func() {...}

int main(int argc, char* argv[]) {
    int* p = func();

    if (p != null) {
      // Do conditional stuff
    }

    // Do further stuff

    free(p); // Upps!!!
}

```

----

### In Rust we like Option< T >

```
fn func() -> Option<T> {...}

fn main () {
    match func() {
        Some(t) => { // do something with t },
        None => { // do something with nothng }
    }
}

```

---

## Memory Safety II

No dangling pointers

----

### C/C++: Dangling Pointer Strikes Back

```
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

```
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

----

### Rust II: Borrow me a reference

```
// str is string slice type: &[]
fn borrow_to_friend(str: str) {
    borrow_to_friends_friend(&str.as_byes());
}

fn borrow_to_friends_friend(buf: &[u8]){}

fn main() {
    // Move temporary to book_title
    let book_title = String::from("Alice in Wonderland");

    // Borrow a string slice reference here
    borrow_to_friend(&book_title);
}
```

----

### Rust III: Take me mutable but nobody else

```
fn add_apocalypse_date(str: mut str) {}

fn main() {
    let book_title = String::from("Apocalypse");

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

```
void check_str(char* buf, int len) {
    for(int i = 0; i <= len; i++) {
        // buf[i]
        // Programmer overruns the buffer on i = len
    }
}

int main(int arc, char* argv[]) {
    char* book = "Learn Overruns in 21 Days";
    char* other = "Smash me in 21 Days";

    // Overrun and smash my stack
    check_str(book, 25);
}
```

----

### Rust: Take a slice leave the rest

```
fn check_str(str: str) {}

fn main() {
    let book = String::from("Learn Slicing in 21 Days");

    check_str(&[0..4]);
}
```

---

## Safe Multi-threading by default

----

### Use the power of all your cores and sleep well

- Data races are another form of aliasing vs. mutation
- Simple rules:
  - Either you move contents into a thread
  - Or you borrow any immutable references
  - Or you borrow one mutable reference but no other else
- Let the compiler check the rules FTW

---

## Why not only for C/C++ Developers?

- Wake up: We hit the End Moore's Law by 2005
- Your PHP, Go, Ruby, etc. is as memory leak free as their impls/vms
- Your PHP, Go, Ruby, etc. works on a costly abstraction above the OS and machine level

---

## Why not only for C/C++ Developers?

- Concurrent and parallel programming styles are going to stay
- Rust enables productive writing of memory safe and concurrent/parallel applications
- Rust enables the same productivity thanks to Cargo, Crates, Rustup, Rustc
- Takes an async/await approach like Scala for web application programming (Currently on Nightly)

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

Periklis Tsirakidis

Github: github.com/periklis

---

## Further reading

- [Jim Blady - Why Rust?](http://www.oreilly.com/programming/free/files/why-rust.pdf)
- [Aaron Turon - Standford Seminar on Rust Lang](https://www.youtube.com/watch?v=O5vzLKg7y-k)
- [Repository - Rust-Learning](https://github.com/ctjhoa/rust-learning)
- [Rust by Example](https://rustbyexample.com/)
- [Official Rust Land Documentation](https://doc.rust-lang.org/book/second-edition/ch01-00-introduction.html)
- [Official Cargo Guide](http://doc.crates.io/guide.html)
- [Official Rust Lang Reference](https://doc.rust-lang.org/stable/reference/)
