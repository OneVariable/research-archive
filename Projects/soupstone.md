# Soupstone

* [On Github](https://github.com/jamesmunns/soupstone)

## Summary

A fun little electronic fidget toy (eventually). The goal is to make a small, battery powered device that has a bunch of fun inputs and outputs, like clicky switches and a piezo speaker, which responds to your inputs.

Ideally: It should be very easy to hack on, and write fun random-ish behaviors, to make it more interesting to play with.

I am currently using the [Seeed Studio XIAO nRF52840 Sense] as the main MCU/SOM, and [Embassy] for software and driver support.

The name comes from the [Stone Soup] folk story, as well as [worry stones].

[Embassy]: https://github.com/embassy-rs/embassy
[Seeed Studio XIAO nRF52840 Sense]: https://wiki.seeedstudio.com/XIAO_BLE/
[Stone Soup]: https://en.wikipedia.org/wiki/Stone_Soup
[worry stone]: https://en.wikipedia.org/wiki/Worry_stone

## Goals

* Make it quick and easy to iterate on
* Make it robust enough to carry around and use continuously
* Don't make it too complicated, it's a toy!
* Build a few units to share with friends and family, and as a neat gift
* Maybe make it something nice to use later for teaching embedded Rust

## Current State

As of March 2023, still pretty early! I've been starting with the [nRF52840-DK], because I had one of these boards lying around already, and it has a built-in debugging probe onboard and has a USB port that is connected to the on-chip USB device capabilities.

The initial work has been focused on writing a very simple "[stage zero]" USB RAM loader (see below). As the XIAO board does not have easily-accessible SWD debugging pins for reprogramming, it will generally be easier to reprogram and debug the device via the main USB port.

I have quite a few parts like toggle/momentary switches, analog potentiometers, rotary encoders, and LEDs in my parts drawers. The next step will be building some basic proto boards to decide what is fun, move them to PCBs, and make some initial rough units to play with.

[nRF52840-DK]: https://www.nordicsemi.com/Products/Development-hardware/nrf52840-dk
[stage zero]: https://github.com/jamesmunns/soupstone/pull/2

## Areas of research

### No Debugger

In Rust, it is very common to use an external debugging probe via SWD, like a [J-Link] or [DAPLink]. This makes it really easy to reprogram the device, and even get logs via something like [probe-run] and [defmt]. These are made possible using [probe-rs]'s wide support for different chips, debug probes, and protocols like RTT for getting logs via the debugger link.

On one hand, this is super powerful. It makes a lot of the initial board bringup painless, and every Cortex-M board supports it.

On the other hand, not every chip has as good of a debugger story. ESP32 has a couple of different debugging link types. RISC-V processors don't typically have a standard debug interface yet.

On top of that, boards like the XIAO don't have the debugging pins on the header, which means you need pogo pins or solder wires onto the bottom of the board. For people without a soldering iron or the skills to solder to a board or design their own board, this might not be an option at all.

In short: I want to play around with different ways of iterating on firmware, without using a debugger. I hope this opens up more opportunities when you don't have a chip, or a board, or the materials to be able to lean on the debugger to make magic happen.

I plan on using the built-in USB support of the nRF52840, but a similar approach should be possible over UART, or whatever other interfaces are available.

[J-Link]: https://www.segger.com/products/debug-probes/j-link/
[DAPLink]: https://daplink.io/
[probe-run]: https://github.com/knurling-rs/probe-run
[defmt]: https://github.com/knurling-rs/defmt/
[probe-rs]: https://probe.rs/

### RAM Loader

I got really in my head about having "the perfect bootloader". It can be really hard to make a "bullet proof" bootloader, and flexibily support flashing to both internal flash, and external flash.

To break the "[analysis paralysis]", I decided to make the dumbest persistent loader I could.

This meant:

* It could only write to RAM, not flash
* A very simple, USB-serial link, using [postcard]
* Nothing I write with the loader will survive a reboot

In general when you are writing early code for microcontrollers, or rapidly iterating on whatever embedded project you are working on, you don't really care about the code running when it isn't attached to your computer. If you have enough RAM to fit both the application and the RAM used at runtime by the application, that means you can skip the slowest part of "flashing" a device: erasing and writing the flash memory.

Luckily with the nRF52840, you have 256KiB of RAM, which is usually plenty for both application and running RAM. For reference, I often work with [STM32G0] chips, which only have 16KiB-64KiB of Flash, and 8KiB of RAM, so the nRF has more space than I'd ever see on that entire device!

At the moment, I have a working [stage zero] RAM loader working, which makes it pretty quick to reboot, flash, reboot, and run brand new code.

[analysis paralysis]: https://en.wikipedia.org/wiki/Analysis_paralysis
[postcard]: ./postcard.md
[STM32G0]: https://www.st.com/en/microcontrollers-microprocessors/stm32g0-series.html

### Standard USB communication "app framework"

Once we have the image flashed, we still want to have the "nice things" from probe-run, namely getting text logging out, as well as communicating with the device to do things like tell it to reboot.

I'm working on that "[app framework]" functionality now, and you can check out a little demo here:

[![asciicast](https://asciinema.org/a/566643.svg)](https://asciinema.org/a/566643)

Some of the parts are still "duct taped" together, but the plan is to make a single PC-side application that handles both the [stage zero] communication, as well as the [app framework] code, to make it easy to have your device update with new firmware as soon as you hit save on your editor.

[app framework]: https://github.com/jamesmunns/soupstone/pull/4

## Note logs

None yet.
