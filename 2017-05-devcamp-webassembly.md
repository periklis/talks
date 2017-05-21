---
author:
- Periklis Tsirakidis
title: WebAssembly - Zero-Latency Backends in your Browser
---

# WebAssembly

### Zero-Latency Backend in your Browser

---

## A new standard for the web

"WebAssembly or wasm is a new portable, size- and load-time-efficient format suitable for compilation to the web."

__Developed by the W3C Community Group with Apple, Google, Mozilla and Microsoft__

---

## WebAssembly promises

1. Designed to be encoded in a size- and load-time-efficient binary format.
2. WebAssembly is designed to be pretty-printed in a textual format for debugging, testing, etc.
3. Describes a memory-safe, sandboxed execution environment that may even be implemented inside existing JavaScript virtual machines.

---

## History of browser JS engines

----

### Mozilla's *monkeys

| Engine       | Live       | Function               |
| ------------ | ---------- | ---------------------- |
| SpiderMonkey | R.I.P.2008 | The interpreter        |
| Tracemonkey  | R.I.P.2011 | Initial JIT compiler   |
| Jägermonkey  | R.I.P.2013 | Baseline JIT           |
| IonMonkey    | 2012 -     | Full optimizing JIT    |
| Baseline     | 2013 -     | Jägermonkey repl.      |
| OdinMonkey   | 2013 -     | Ahead-Of-Time Compiler (ASM.JS) |
| BaldrMonkey  | 2016 -     | WASM Compiler          |

----

### Google Chrome Tentacles

| Engine        | Live        | Function                  |
| ------------- | ----------- | ------------------------- |
| Full-code-gen | R.I.P.2017? | Fast compiler unopt code  |
| Crankshaft    | R.I.P.2017? | Slow compiler opt. code   |
| Ignition      | 2016 -      | The bytecode interpreter  |
| Turbofan      | 2015 -      | The optimizing JIT        |

---

## Native code on the browser before WebAssembly

1. ASM.JS: Highly optimized subset of javascript for ahead-of-time compilation
2. (P)NaCl: Google chrome's extensions and binary format for a native compilation target for the web

---

## Why WebAssembly?

Cause Online Games and pr0n rule the web?

No there is more: Cryptography, Audio, Image Processing, VR, Virtual Machines, CAD,...

---

## Why WebAssembly again?

- Avoid plugins (e.g. flash, activex, java)
- Bring existing applications to the web (too big to rewrite)
- Port high-performance native libraries to the web

---

## I want my native code running on the web now! But how?

---

## Emscripten Toolchain

- C/C++/Rust => LLVM => Emscripten => WASM
- A Bunch of APIs for Filesystem, WebGL, HTML5, etc.
- Integrated Binders: WebIDL, embind

----

### Emscripten toolchain intermediates

C/C++ => LLVM => Emscripten => Binaryen

C/C++ => Bitcode => _WAST_ => WASM

---

## Step by Step: Build native code to webassembly

----

### Starter: The obligatory Hello World

```
#include <iostream>

int main(int argc, char* argv[]) {

  std::cout << "Hello world" << "\n";

  return 0;
}
```

Compile with:
```
$ em++ hello.cpp -o hello.js -s BINARYEN=1 -s "BINARYEN_METHOD='native-wasm'"
$ ls -la
-rw-r--r-- 1 user staff 1774226 May 15 21:44 hello.asm.js
-rw-r--r-- 1 user staff     110 May 15 21:40 hello.cpp
-rw-r--r-- 1 user staff  276090 May 15 21:44 hello.js
-rw-r--r-- 1 user staff  394912 May 15 21:44 hello.wasm
```

----

### Step 1: Prepare your buildsystem to use emscripten

Autotools: Use emconfigure / emmake wrappers

```
cd project-dir/
emconfigure ./configure --prefix=...
emmake make
```

CMake: Use Emscripten.Toolchain

```
cd project-dir/build
cmake -DCMAKE_TOOLCHAIN_FILE=/PATH/TO/Emscripten.cmake ../src
```

----

### Step 2: Check your library dependencies

- Build library dependencies with emscripten, too!!!
- Build and link library dependencies as static libraries
- No support for shared libraries yet (Post-MVP: ES6 Modules)

----

### Step 3: Check for webassembly limitations

- NO arch assumption: endianess, alignment
- NO stack manipulation: setjmp/longjmp
- NO dlopen/dlsym support
- NO native assembler support (e.g. cryptography)
- NO threading support yet
- NO SIMD support yet
- NO cross-language exception handling support
- NO 64-bit support yet
- NO full IEEE 754-2008 floating point support yet

----

### Step 4: Prepare your interface for your JS-Code

```
#ifndef IMAGEPROCESSOR_H_
#define IMAGEPROCESSOR_H_

#include <map>
#include <string>
#include <vector>

namespace mayflower  {
namespace wasm {

class ImageProcessor {
 public:
  explicit ImageProcessor(const std::string& path)
      : path_(path) {};

  ImageProcessor(const ImageProcessor& ip) = delete;
  ImageProcessor& operator=(const ImageProcessor& ip) = delete;

  std::map<std::string, long> dimensions();
  std::vector<int> histogram();
  void resize(int w, int h);
  void crop(int start_x, int start_y, int end_x, int end_y);

 private:
  std::string path_;
};
} // namespace wasm
} // namespace mayflower

#endif  // IMAGEPROCESSOR_H_

```

----

### Step 5: Write the binders for javascript

e.g. embind:
```
#include <emscripten/bind.h>
#include "imageprocessor.hpp"

EMSCRIPTEN_BINDINGS(ImageProcessor){
emscripten::register_vector<int>("VectorInt");
emscripten::register_map<std::string, long>("MapStringLong");

emscripten::class_<mayflower::wasm::ImageProcessor>("ImageProcessor")
  .constructor<std::string>()
  .function("dimensions", &mayflower::wasm::ImageProcessor::dimensions)
  .function("histogram", &mayflower::wasm::ImageProcessor::histogram)
  .function("resize", &mayflower::wasm::ImageProcessor::resize)
  .function("crop", &mayflower::wasm::ImageProcessor::crop);
}

```

----

### Step 6: Run your build
```
$ cd build
$ make
$ ls -la
-rw-r--r-- 1 user staff 4571975 Apr 28 imageprocessor.asm.js
-rw-r--r-- 1 user staff  178348 Apr 28 imageprocessor.js
-rw-r--r-- 1 user staff  578728 Apr 28 imageprocessor.wasm
```

----

### Step 7: Test you build in nodejs

For example with cmake & ctest:
```
$ cd build
$ ctest

Test project /Path/to/wasm-imageeditor/build
    Start 1: imageprocessor-test
1/1 Test #1: imageprocessor-test ..............   Passed    0.67 sec

100% tests passed, 0 tests failed out of 1

Total Test time (real) =   0.68 sec
```

----

### Step 8: Load your wasm module in your react app
Load the module in a react component through fetch:
```
fetch('imageprocessor.wasm')
  .then((response) => response.arrayBuffer())
  .then((buffer) => {
    Module.arguments = this.props.binArguments;
    Module.environment = this.props.environment;
    Module.locateFile = this.props.locateMemFile;
    Module.logReadFiles = this.props.logReadFiles;
    Module.noExitRuntime = this.props.noExitRuntime;
    Module.noInitialRun = this.props.noInitialRun;
    Module.print = this.props.print;
    Module.printErr = this.props.printErr;
    Module.preInit = this.props.preInit;
    Module.preRun = this.props.preRun;
    Module.postRun = this.props.postRun;
    Module.wasmBinary = buffer;
   });
```

----

### Step 9: Load emscripten shell code
```
fetch('imageprocessor.js')
  .then((response) => response.blob())
  .then((responseBlob) => {
    const script = document.createElement('script');
    const src = URL.createObjectURL(responseBlob);

    script.src = src;
    script.async = false;
    document.getElementById('ImageProcessor').appendChild(script);
});
```

----

### Step 10: Call your wasm module in your react app

```
// Instantiate wasm through window.Module latter ES6 module
const ip = new Module.ImageProcessor(targetFilename);

// Call your native good through your bindings
const dims = ip.dimensions();
const newWidth = dims.get("x") * (zoomFactor / 100);
const newHeight = dims.get("y") * (zoomFactor / 100);

ip.crop(0, 0, newWidth, newHeight);

// Destroy your native module
ip.delete();
```

---

## Demo
### WebAssembly + ReactJS Imageeditor

[Open Demo](http://localhost:8080/)

- Frontend: ReactJS + Redux + Typescript
- Backend:  C++ + Boost/Gil + libjpg
- Bundler: Webpack2
- Buildsystem: cmake

https://github.com/periklis/wasm-imageeditor

---

## WebAssembly Integrations

| Type             | Possibility                       |
| ---------------- | --------------------------------- |
| Filesystem       | NodeFS, IndexedDB, MEMFS          |
| Audio            | AudioContext through SDL          |
| WebGL            | WebGL through SDL                 |
| WebAPIs          | emschripten/html5.h               |
| Call JS in C/C++ | Use emscripten.h                  |
| Worker API       | Use emscripten.h for "threads"    |
| Stdio            | Use Module.print/printErr         |

---

## WebAssembly Browser Support

| Browser | [Can i use wasm?](http://caniuse.com/#feat=wasm) | [Can i use asm.js](http://caniuse.com/#search=asm) |
| ------- | ----------- | ------ |
| Firefox | 53          | 52     |
| Chrome  | 57          | 49*    |
| Edge    | 14          | 14     |

_*Chrome without Ahead-Of-Time-Compilation_

---

# So long and thanks for the fish!

Periklis Tsirakidis

Github: github.com/periklis
