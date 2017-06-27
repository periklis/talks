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
- 1.0 Release since mid 2015
- Robust memory leak free programming
- Safe data race free multi-threading programming
- Follows Zero Abstraction Cost Philosophy

---

## How does Rust succeed on its promises?

- Simple rules about pointer aliasing and data structure mutation
- Rules enforced through a sophisticated type system
- Enhanced front end compiler support on top of LLVM
- Strong focus on programming language ergonomics
- Long research project history 2004 -2008

---

# Let's dive in Rust

---

## Memory Safety I: No null pointer dereferences

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

### The solution in Rust: Option< T >

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

## Memory Safety II: No dangling pointers

----

### C/C++: Dangling Pointer Strikes Back

```
void proc_str(char* buf) {
    // process buffer
    free(buf);
}

int main(int argc, char* argv[]) {
    proc_str(arg[0]);

    // arg[0] now dangling
}

```

----

### Rust I: Move me, clone me, drop me

```
fn func(str: String) {
    // Process str
} // Drop str

fn main {
    // Move temporary to str
    let str = String::from("Hello World");

    // Clone str into str2
    let str2 = str.clone();

    // Move str to arg str
    func(str);
} // Drop str2
```

----

### Rust II: Borrow me a reference

```
// str is string slice type: &[]
fn proc_str(str: str) {}

fn proc_bytes(buf: &[u8]){}

fn main() {
    // Move temporary to str
    let str = String::from("Hello World");

    // Borrow a string slice reference here
    proc_str(&str);

    proc_bytes(&str.as_byes());
}
```

----

### Rust III: Take me mutable but nobody else

```
fn mut_str(str: mut str) {}

fn main() {
    let str = String::from("Hello World);

    mut_str(&mut str);

    // ERROR
    let other_str = &mut str;
}

```

---

## Memory Safety III: No buffer overruns

----

### C/C++: Programmers overrun buffers

```
void proc_str(char* buf, int len) {
    for(int i = 0; i <= len; i++) {
        // buf[i]
        // Programmer overruns the buffer on i = len
    }
}
```

----

### Rust: Take a slice leave the rest

```
fn proc_str(str: str) {}

fn main() {
    let str = String::from("Hello World);

    proc_str(&[0..4]);
}
```

---

## Safe Multithreading by default

----

### Use the power of all your cores and sleep well

- Data races are another form of aliasing vs. mutation
- Simple rules:
  - Either you move contents into a thread
  - Or you borrow any immutable references
  - Or you borrow one mutable reference but no other else
- Let the compile check the rules FTW

---

## Not only a thing for C/C++ Developers?

- Wake up: We hit the End Moore's Law by 2005
- Your PHP, Go, Ruby, etc. is as memory leak free as their impls/vms
- Your PHP, Go, Ruby, etc. works on a costly abstraction above the OS and machine level

---

## Not only a thing for C/C++ Devs?

- Concurrent and parallel programming styles are going to stay
- Rust enables productive writing of memory safe and concurrent/parallel applications
- Rust enables the same productivity thanks to Cargo, Crates, Rustup, Rustc
- Takes an async/await approach like Scala for web application programming

---

## Rust Key Facts

- Express Ownership explicitly!
- Borrowing down the stack!
- No Exceptions but Result< T >!
- No Null but Option< T >!
- No Garbage Collection!
- No Green Threads!

---

# So long and thanks for the fish!

Periklis Tsirakidis

Github: github.com/periklis

---

## Further reading
