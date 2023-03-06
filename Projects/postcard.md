# Postcard

Relevant links:

* [On Github](https://github.com/jamesmunns/postcard)
* [On crates.io](https://crates.io/crates/postcard)
* [Postcard Wire Format Specification]

[Postcard Wire Format Specification]: https://postcard.jamesmunns.com/

## Summary

Postcard is a serializer and deserializer library and wire format, written in Rust. It is built on top of the [serde] framework, the most widely used way of handling serialization and deserialization in Rust.

It was originally written to be portable, resource friendly, and simple for embedded systems, but due to its efficient and performant nature, has found home in a number of other locations.

It is [not a self-describing format], meaning that both the sender and receiver must share a mutual understanding of the "schema" of the messages. This differs from formats like JSON, where it is possible to process a message with no prior knowledge of the format. This makes an explicit trade-off of flexibility for speed and efficiency.

[serde]: https://serde.rs/
[not a self-describing format]: https://postcard.jamesmunns.com/wire-format.html#non-self-describing-format

## Goals

The design goals for postcard are:

1. Design primarily for `#![no_std]` usage, in embedded or other constrained contexts
1. Support a maximal set of `serde` features, so `postcard` can be used as a drop in replacement
1. Avoid special differences in code between communication code written for a microcontroller or a desktop/server PC
1. Be resource efficient - memory usage, code size, developer time, and CPU time; in that order
1. Allow library users to customize the serialization and deserialization behavior to fit their bespoke needs

## Current State

As of June 2022, Postcard has reached [a 1.0 state]. **It is considered stable, production ready, and well maintained**.

Postcard has a published [Wire Format Specification][Postcard Wire Format Specification], meaning that the serialization format is guaranteed to not change for all `1.x.y` versions of Postcard.

It is widely used in the embedded Rust ecosystem, and has also found use in:

* The Unicode Consortium's [ICU4X] tool as a "DataProvider" format, storing internationalization metadata in a compact way.
* The [Bevy Game Engine], as a "[Binary Scene]" format, storing resources such as UIs, environmental objects, and entities for importing and exporting.
* A number of other crates on [crates.io](https://crates.io/crates/postcard/reverse_dependencies).
* A number of other projects on [github](https://github.com/jamesmunns/postcard/network/dependents).

[a 1.0 state]: https://jamesmunns.com/blog/postcard-1-0/
[ICU4X]: https://github.com/unicode-org/icu4x
[Bevy Game Engine]: https://bevyengine.org/
[Binary Scene]: https://bevyengine.org/news/bevy-0-9/#binary-scene-formats


## Areas of research

The following are open areas of research related to the Postcard library.

I am actively seeking sponsors and interested parties in any of these areas of research.

### Message Schema Export

At the moment, the only official way to specify a message's schema is through the use of serde, and specifying data types in the Rust language.

Postcard (as of March, 2023) has an experimental feature for generating a [schema description] for a given data type. At the moment, this format contains all relevant data, but there is very little "out of the box" that you can do with this data.

I would like to be able to do a few things with this generated schema data:

1. Export the format specification to a human-readable format, to be used as part of documentation of a protocol specification, or as part of an [Interface Control Document]. This export process could be automatic (when building an application), or manually performed.
1. Use the format specification for code generation purposes, allowing languages like C, Python, JavaScript, or others to serialize and deserialize postcard messages, without having to call Rust code via FFI.
1. Use the schema for automated detection when potentially breaking changes are made to a format, for example in CI tests.

[schema description]: https://docs.rs/postcard/latest/postcard/experimental/index.html#message-schema-generation
[Interface Control Document]: https://en.wikipedia.org/wiki/Interface_control_document

### Self Describing Extensions

Relevant Github Tracking issue: https://github.com/jamesmunns/postcard/issues/92

As mentioned above, postcard is NOT a self-describing format. This trade-off has been made to reduce the message size on the wire, as well as complexity when serializing and deserializing. The upsides of this are:

* The messages "on the wire" are MUCH smaller than they would be with additional schema information inline.
* The serialization and deserialization process is much simpler and more deterministic, often just copying and validating fields. This leads to far less CPU, memory, and code size usage

However, this comes with some downsides:

* Changes to a message schema must be made carefully, to avoid incompatible modifications
* Both the sender and receiver must know the exact schema, ahead of time
* It is difficult to even DETECT when message formats have been changed, leading to potentially incorrect behavior when changes are made intentionally or unintentionally

I am currently researching ways to optionally provide schema data "on the side", in a way that can be used to aid flexibility for schema changes, and handling messages with schemas that are not known ahead of time.

I hope to be able to provide this metadata "on request", for example on first connection, rather than within every message.

I have prototyped a [compact format] for this "on the side" schema, which could allow for dynamic deserialization, or cross-checking that the sender and receiver agree on the message format.

[compact format]: https://cohost.org/jamesmunns/post/960289-a-schema-in-128-byte

## Note logs

None yet.
