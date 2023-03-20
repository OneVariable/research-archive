# An Easier Embedded Rust

## Problem Statement

**How could Rust make it easier for people just getting started with Rust and/or embedded?**

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

However, Rust does not have as good of a "getting started (new to embedded and/or new to rust" story, or a story for "just want things to work (artists/hobbyists)").

> TODO: These claims above need better examples and citation. At the moment, they are largely "gut feelings" from James.

Ecosystems like Arduino and CircuitPything/MicroPython make getting started much easier than Rust does today, despite whatever shortcomings which exist (or are perceived to exist) with those ecosystems, such as performance, reliability, licensing or "under the hood complexity."

## Concepts to consider

Balancing "simple" and "easy". How can we make things *easier*, without hiding all of the complexity under the hood, or making things more complex just so we can hide them under the hood?

Hidden complexity makes it very hard if you ever fall off the "happy path". This means the skill required to debug or extend anything in the "hidden" parts is likely much higher, which might not be reasonable for someone to learn incrementally.

Requiring "extra" tools not typically used by "production" embedded rust users makes it harder to find help. By using "extra" tools, you can only be helped by people who know *those* tools, instead of anyone who works in the larger embedded rust ecosystem.

Special systems means effort is split between "general embedded rust" and "users of the special system".

How can this be done efficiently? Arduino and CircuitPython both have corporate backing in addition to their open source efforts. While there is a growing open source embedded rust community, any new approaches would have to leverage existing work or start from scratch.

How can this be done in a way that has a "path" to more traditional Rust usage? When users become more comfortable with embedded Rust, and would like to make lower level modifications, how can they do this without abandoning everything they've learned, or without experiencing a steep learning curve?

## Potential Research Areas

### Better enumeration of Why and How other approaches are easier

It would be good to do an overview of today's Arduino and CircuitPython environments, and see what they provide (or don't) to make the experience easier. It would also be useful to capture where their approaches "fall over", or what limitations typical users run into.

These systems make "the easy things easy to do", but at what limit of "difficulty/complexity" does a problem need to be before things are no longer easy to do?

### Using a small OS/Runtime

This approach is somewhat similar to how CircuitPython approaches the problem: Have a small, portable runtime that provides base functionality. This core is compiled, and is often target (or CPU family) specific.

This base is extended by statically or dynamically included modules. In CircuitPython this can be pre-compiled modules, or textual python code. In most or all cases, this can be done without "flashing" the device (in the traditional sense), often achieved by "copying files" to a virtual flash drive interface for portability.

This approach may limit application to targets with enough resources (typically RAM and Flash, including external flash) to handle both an "OS"/"Host Environment", as well as the user application.

### Provide a "common enough" base library

This approach is somewhat similar to how Arduino approaches the problem: making code "seem" the same to the end user, by forcing libraries to use a common, high level set of interfaces.

User code is compiled together with selected libraries to produce a single, static code image which is flashed to the device (either via debugger or bootloader).

Arduino does some extra work here by providing its own IDE and package manager ("board" and "library" managers) to manage dependencies, that may be already available out of the box in Rust.

Rust does have language level support for standard interfaces via Traits (rather than 'common header/api' approach used in C++/Arduino), which are leveraged heavily by embedded-hal.

However, the "OSS zeitgeist" of Rust is to provide MUCH more control over implementation details that are important to "power users", but less so to "hobbyist/artist" users (at least for initial usage).

This wide variety of "options" is often confusing or even overwhelming for newcomers. This is very commonly seen on "early main code" - setting up clocks, peripherals, and other resources.

If the library approach is taken, it may be interesting to see how far we could get by taking existing HAL code (or a more unified HAL, like Embassy's), and being "more opinionated", willing to take "more portable" but "less theoretically efficient" approaches, including allocating buffers, or handling things like interrupts behind the scenes.

## Notes from Friends

I asked two friends for their opinions:

* [Anatol] - as he comes from an embedded Rust background, but has recently been trying out CircuitPython for a project, and
* [Scott M] - as he is one of the maintainers of [esp-rs], which has an option to run Rust code with the standard library on top of their IDF based RTOS.

[Anatol]: https://github.com/spookyvision/
[Scott M]: https://github.com/MabezDev
[esp-rs]: https://github.com/esp-rs

Answers I got back:

### [Anatol] - regarding CircuitPython:

> * REPL (fantastic for playing around with e.g. LED values, or PWM, etc.)
> * auto-reload (exposes flash as mass storage, updating /[code.py](http://code.py/) restarts the app)
>     * this includes a fancy special case for display drivers (any kind really, hub75 as well as LCDs): as soon as you "quit the app" (via C-c), the runtime takes over the display and shows the repl (this is of somewhat limited use unless you have some kind of keyboard connected, but ... just very cool)
> * out of the box starts a ton of USB devices (uart, midi, keyboard, etc. - even has host support but only for teensy)
> * mainly though: cross-architecture HAL + "BSC" (e.g. you can do spi = board.SPI() - that's literally all you need) + "OS" (layout, font rendering, animation etc) with very, very few LOC required)
>
> I think [these examples/cheatsheet things](https://github.com/adafruit/awesome-circuitpython/blob/main/cheatsheet/CircuitPython_Cheatsheet.md) are a good showcase, or [these](https://docs.circuitpython.org/projects/displayio-layout/en/latest/examples.html) (if your specific board has a display you don't even have to initialize it, you get board.DISPLAY for free)
>
> oh, speaking of filesystems! it comes with a vfs! so you can mount an sd card on top of your flash fs
>
> ```python
> import os
>
> import board
> import sdcardio
> import storage
>
> sd = sdcardio.SDCard(board.SPI(), board.SD_CS)
> vfs = storage.VfsFat(sd)
> storage.mount(vfs, '/sd')
> os.listdir('/sd')
> ```
>
> I don't think building is such a big problem with Rust, the main advantages I see with circuitpython are just lots and lots of conveniences/helpers that make for very short programs
>
> and fast iteration
>
> I think I'll add one more bullet point, a bit more philosophical: micro/circuitpython carry the spirit of Python's "batteries included" ... so you don't have to go and find crates, add them to Cargo.toml - most stuff is just bundled in, and designed to interoperate (kinda like embedded-hal on steroids)

### [Scott M] - Regarding RTOS usage:

> Some initial thoughts of the top of my head.
>
> I think running an RTOS (that's not written in Rust) is not worth is unless you are porting the standard library too. If you do port the standard library, it's awesome! If you don't, you just end up using unsafe everywhere. The key points imo are:
>
> * Anyone who knows Rust can write code for a target that supports STD. Many beginners just want to get stuff done, and don't care about choosing alloc impls, or worrying about efficient static memory usage.
> * Access to more of the amazing Rust ecosystem (we have clients running fully fledged tokio on esp's!)
>
> The not so good, but imo everything in this list is solvable:
>
> * Usually you have to build the RTOS along with your Rust build, for many developers this is no big deal but noobs just want it to work without installing 50 million deps
>     * Easy solution to this is to precompile std with your RTOS, just like how std is distributed now
> * Sometimes STD is too high level, i.e a filesystem on an embedded system could be flash based, or a full on fat32/exfat impl
>     * In these instances, for esp-idf anyway, there is some setup to do outside of std to allow the fs parts of std to work correctly.
>     * This gap can be filled nicely with supporting crates, we have esp-idf-hal and esp-idf-svc (svc = services) to do that.
