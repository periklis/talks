---
author:
- Periklis Tsirakidis
title: Deep Dive Nixpkgs Bootstrap
theme: serif
---


## Deep Dive Nixpkgs Bootstrap

Periklis Tsirakidis

---

## Motivation

- Provide an overview of Nixpkgs bootstrap process
- Explain the design of Nixpkgs bootstrap process
- List some usage hints for package overrides

---

## Nixpkgs: key facts

- Single repository for all packages incl. DSL-packages
  - e.g. gcc and haskellPackages.pandoc

- Single repository for all supported platforms
  - e.g. x86_64-linux and x86_64-darwin

- Single repository for all library code
  - e.g. debug, strings, vm, release routines

----


### Overview - What kicks in when?

```
nix-env -f <nixpkgs> ....
```

1. Check Nix version compatibility
2. Handle environment impurities
   - Any Local-/CrossSystem given by user?
   - Any extra configuration available?
   - Any overlays available?
3. Boot stdenv and all packages
   - Import nixpkgs libraries
   - Import top-level entries
   - Boot the stdenv
   - Provide a fix point of nixpkgs incl. stdenv

----

### Detail Overview: The bootstrap

```
nixpkgs/default.nix
├── ./pkgs/top-level/impure.nix
    ├── ~/.config/nixpkgs/config.nix
    ├── ~/.config/nixpkgs/overlays.nix
    ├── ~/.config/nixpkgs/overlays/*.nix
    ├── ./pkgs/top-level/default.nix
        ├── nixpkgs/lib/*.nix
        ├── ./pkgs/stdenv/default.nix
        ├── ./pkgs/top-level/stage.nix
        |   ├── ./pkgs/stdenv/adapters.nix
        |   ├── ./pkgs/build-support/trivial-builders.nix
        |   ├── ./pkgs/top-level/splice.nix
        |   ├── ./pkgs/top-level/all-packages.nix
        |   ├── ./pkgs/top-level/aliases.nix
        ├── ./pkgs/stdenv/booter.nix
```

A trace of the imports during the nixpkgs bootstrap process.

----

### Detail Overview: Enter nixpkgs

#### File: nixpkgs/default.nix

##### Why do we check the nix version on bootstrap?

- Nixpkgs uses builtins thus backward compatibility limit
- Nixpkgs is huge thus evaluation performance is critical
- Future Nix language changes

----

### Detail Overview: Enter Top-Level

#### File: pkgs/top-level/impure.nix
```nix
{ localSystem ? builtins.intersectAttrs { ... } args}
```
```nix
homeDir = builtins.getEnv "HOME";
```
```nix
config ? let
      configFile = getEnv "NIXPKGS_CONFIG";
      configFile2 = homeDir + "/.config/nixpkgs/config.nix";
      configFile3 = homeDir + "/.nixpkgs/config.nix"; # obsolete
      in ...
```
```nix
overlays ? let
      isDir = path: pathExists (path + "/.");
      pathOverlays = try (toString <nixpkgs-overlays>) "";
      homeOverlaysFile = homeDir +"/.config/nixpkgs/overlays.nix";
      homeOverlaysDir = homeDir + "/.config/nixpkgs/overlays";
      overlays = ...
```
```nix
localSystem={ system = builtins.currentSystem; } // localSystem;
```
Central point to handle all impurities and side effects before pure functional evaluation!
----

### Detail Overview: Enter Top-Level

#### File: pkgs/top-level/default.nix

```nix
{ # Import the stdenv stages
  stdenvStages ? import ../stdenv } @ args:
```
```nix
# Make nixpkgs a function to accept: --argstr withXYZ true
allPackages = newArgs: import ./stage.nix ({
  inherit lib nixpkgsFun;
} // newArgs);
```
```nix
# Import the stdenv boot facility, just a foldLeft
boot = import ../stdenv/booter.nix { inherit lib allPackages; };
```
```nix
# Produce list of stdenv stages for a local/cross system:
# Each stdenv is separated in a list of functions called stages.
stages = stdenvStages {
  inherit lib localSystem crossSystem config overlays;
};
```
```nix
# Boot the stdenv with a fix attrset of all packages
pkgs = boot stages;
```

----

### Detail Overview: Enter Stdenv

What is actually the standard environment?

- A set of tools to build and package other software,
  - coreutils, gcc/clang, bash, gzip
- A set of important library functions
  - mkDerivation
- A set of adapters, e.g:
  - overrideCC, overrideSetup, keepDebugInfo
- A closure for platform impurities (darwin)
  - /usr/lib/libSystem.B.dylib
  - MACOSX_DEPLOYMENT_TARGET=10.10

----

### Detail Overview: Enter Darwin Stdenv

#### An augment symfony in six stages

1. Boostrap Tools: sh, bzip2, mkdir, cpio
2. Stage 0: Bintools, coreuitls, gnugrep
3. Stage 1: LibSystem, Libc++, Libc++-abi
4. Stage 2: dyld, xnu, libdispatch, icu, launchd
5. Stage 3: bash, libiconv, locale
6. Stage 4: llvm (clang, etc.)
7. Stage 5: stdenv + cc-wrapper

----

### Detail Overview: Enter stage.nix

#### A function composition down to a computed fix point

```nix
toFix = lib.foldl' (lib.flip lib.extends) (self: {}) ([
    stdenvBootstappingAndPlatforms
    platformCompat
    stdenvAdapters
    trivialBuilders
    splice
    allPackages
    aliases
    configOverrides
  ] ++ overlays ++ [ stdenvOverrides ]);
in # Return the complete set of packages.
lib.fix toFix
```
```
# Remember:
fix = f: let x = f x; in x;

extends = f: rattrs: self:
  let super = rattrs self; in super // f self super;
```

```nix
# Every function above has the same structure:
f = self: super: { ... }
```

---

## Usage Hints

----

### Override stdenv only in derivation argument list

Wrong:
```nix
self: super: {
  stdenv = super.stdenv.overrideCC super.stdenv self.clang;
}

# error: infinite recursion encountered, at undefined position
```
Correct:
```nix
self: super: {
  hello = super.hello.override {
    stdenv = super.overrideCC super.stdenv self.gcc;
  };
}
```

----

### Select override mechanism

| Method                | Purpose                                                      |
| --------------------  | -------------------------------------------------------      |
| `override`            | Override arguments passed to derivations                     |
| `overrideAttrs`       | Override only attrs passed to `mkDerivation`                 |
| `overrideDerivation`  | Override final attrs passed to raw `derivation` builtin      |
| `lib.makeOverridable` | Provide `override` and/or default args to your own functions |


----

### Select override mechanism with care

```nix
self: super: {
  hello = super.hello.override {
    withI18n = true;
    withMusl = self.musl123;
  };

  helloWithDebug = super.hello.ovverideAttrs (oldAttrs: rec {
    separateDebugInfo = true;
  });

  helloGithub = super.hello.overrideDerivation (finalAttrs: rec {
    version = "1.2.3";
    src = super.fetchFromGithub {...};
  });
}
```

---

### So long and thanks for the fish!

Periklis Tsirakidis

Github: [github.com/periklis](https://github.com/periklis)

Slides: [periklis.github.io/talks](https://periklis.github.io/talks/)
