# An Easier Embedded Rust

## Problem Statement

**How could Rust make it easier for people just getting started with Rust and/or Embedded?**

Rust has made embedded development better (for me) in a number of ways:

* Leveraging what Rust already gives you:
    * Tools like Cargo for build + package management
    * Ability to run `no_std` programs easily
    * Modern language features (modules, iterators, type system)
    * Making it "harder to get things wrong" - use the compiler to detect errors statically
* Building good tools:
    * Way better debugger tools via probe-rs, and tools built on top like probe-run and probe-rs-cli
    * Tooling for logging, like defmt
* Reasonably good docs, and teaching materials

However, Rust does not have as good of a "getting started (new to embedded and/or new to rust" story, or a story for "just want things to work (artists/hobbyists)".

For whatever shortcomings (performance, reliability, licensing, "under the hood complexity") exist (or are perceived to exist) for ecosystems like Arduino and CircuitPython/MicroPython: they make getting started much easier than Rust does today.

## Concepts to consider

Balancing "simple" and "easy". How can we make things *easier*, without hiding all of the complexity under the hood, or making things more complex just so we can hide them under the hood?

Hidden complexity makes it very hard if you ever fall off the "happy path". Requiring "extra" tools not typically used by "production" embedded rust users makes it harder to find help. Special systems means effort is split.

How can this be done efficiently? Arduino and CircuitPython both have corporate backing in addition to their open source efforts. While there is a growing open source embedded rust community, any new approaches would have to leverage existing work or start from scratch.

How can this be done in a way that has a "path" to more traditional Rust usage? When users become more comfortable with embedded Rust, and would like to make lower level modifications, how can they do this without abandoning everything they've learned, or without experiencing a steep learning curve?

## Potential Research Areas

### Better enumeration of Why and How other approaches are easier

It would be good to do an overview of today's Arduino and CircuitPython environments, and see what they provide (or don't) to make the experience easier. It would also be useful to capture where their approaches "fall over", or what limitations typical users run into.

### Using a small OS/Runtime

This approach is somewhat similar to how CircuitPython approaches the problem: Have a small, portable runtime that provides base functionality. This core is compiled, and is often target (or CPU family) specific.

This base is extended by statically or dynamically included modules. In CircuitPython this can be pre-compiled modules, or textual python code. In most or all cases, this can be done without "flashing" the device (in the traditional sense), often achieved by "copying files" to a virtual flash drive interface for portability.

This approach may limit application to targets with enough resources (typically RAM and Flash, including external flash) to handle both an "OS"/"Host Environment", as well as the user application.

### Provide a "common enough" base library

This approach is somewhat similar to how Arduino approaches the problem: making code "seem" the same to the end user, by forcing libraries to use a common, high level set of interfaces.

User code is compiled together with selected libraries to produce a single, static code image which is flashed to the device (either via debugger or bootloader).

Arduino does some extra work here by providing it's own IDE and package manager ("board" and "library" managers) to manage dependencies, that may be already available out of the box in Rust.

Although Rust does have language level support for standard interfaces via Traits (rather than 'common header/api' approach used in C++/Arduino) - leveraged heavily by embedded-hal, the "OSS zeitgeist" of Rust is to provide MUCH more control over implementation details that are important to "power users", but less so to "hobbyist/artist" users (at least for initial usage). This is often confusing or even overwhelming for newcomers. This is very commonly seen on "early main code" - setting up clocks, peripherals, and other resources.

If the library approach is taken, it may be interesting to see how far we could get by taking existing HAL code (or a more unified HAL, like Embassy's), and being "more opinionated", willing to take "more portable" but "less theoretically efficient" approaches, including allocating buffers, or handling things like interrupts behind the scenes.
