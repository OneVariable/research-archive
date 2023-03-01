# MnemOS

## Summary
MnemOS is a small, general purpose operating system (as opposed to "real time operating system", or RTOS), designed for constrained environments.

As an operating system, it is aimed at making it easy to write portable applications, without having to directly interact with the hardware.

## Goals
For "Why did I write it": This is a "spin off" of part of my previous project, Powerbus, which is a networked home automation project. It honestly started getting too complicated with too many "crazy ideas" (it's a network stack! and a scripting language! and an operating system!) for one single hobby project. So I split the "networking" part off to a much smaller, simple project (lovingly named "Dumb Bus" at the moment), and MnemOS, which was more focused on building a single computer system.

This split helped to better focus BOTH parts of the (former) Powerbus system, and may in the future be recombined, when the separate parts have had more time to bake and solidify on their own.

At the moment, it has the benefit of being relatively small (compared to operating system/kernel projects like Linux, or Redox), which makes it easier to change and influence aspects of the OS. I don't think it will ever be anything "serious", but I do plan to use to it to create a number of projects, including a portable text editor, a music player, and maybe even music making/sythesizer components. Some day I'd like to offer hardware kits, to make it easier for more software-minded folks to get started.

## Current State
At the moment, MnemOS is not a multitasking operating system, so only one application (and the kernel) can run at any given time.

## Open tasks
TO DO James

## Areas of research
TO DO James

## Cross links to related topics/projects
[MnemOS Guide](https://mnemos.jamesmunns.com/)

[MnemOS / Pellegrino : Github link](https://crates.io/crates/mnemos-common)

Crate.io Links

+ [MnemOS Kernel](https://crates.io/crates/mnemos)
+ [MnemOS Userspace Library](https://crates.io/crates/mnemos-userspace)
+ [MnemOS Common Components](https://crates.io/crates/mnemos-common)

## Note logs
What is Pellegrino?
