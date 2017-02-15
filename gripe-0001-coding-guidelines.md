# GRIPE 0001 -- Coding Guidelines

- Original Author: Marcus Müller <marcus@hostalia.de>
- Champion: Marcus Müller <marcus@hostalia.de>
- Status: Draft 

History:
- 15-Feb-2017: Initial Draft


## Abstract

This document establishes coding guidelines for the GNU Radio main source tree.

Things defined here should apply, within the boundaries of smaller projects, to
Out-Of-Tree Modules (OOTs), too.

## Copyright / License

[WTFPL](http://www.wtfpl.net/txt/copying/) Version 2 or later

## Motivation

GNU Radio's code has a medium to high quality, but it can be argued that the
lack of clear coding guidelines makes it harder for contributors to cooperate.

Also, a lot of different styles are used throughout GNU Radio.

This document strives to *unify* rather than to *discriminate*. It should give
you a feeling for what *good* code looks like, and new additions to the GNU
Radio main tree should be held against this. In the long run, we'd want to make
sure all of GNU Radio's code adheres to this.

## Description

### Coding Style

**TODO** pick and agree on a well-defined coding style.

### Code Quality

* Block operation SHALL be side-effect free aside from
  * the output buffer
  * message passing
  * the state of the block instance itself
  * the state of the superclass instances as well as interaction with the GNU Radio scheduler
  * gr-prefs reading
  * in case of sinks and sources, the Input- or Output states these are designed for
* Block operation MUST be thread-safe
  * Getters and Setters must be safe to call while (general_)work is executed in a different thread
  * The former can be, preferably, implemented by having a message passing interface
* Compilation MUST work on all platforms supported by GNU Radio, except if the block is only enabled platform-specifically
* Building SHALL be warning-free on current GCC
* Every functional unit (especially, every block) MUST come with a unit test, which all of
  * MUST test instantiation (if any)
  * MUST test logical functionality
* Every functional unit MUST come with documentation, that, if applicable all of
  * describes in- and output types
  * describes the function performed
  * if internal state is kept, define that
  * defines behavior on start()/stop()
* Blocks SHALL come with an example GRC flow graph, if applicable
* Code SHALL NOT include any magic constants
* Blocks MUST tolerate repeated start()/stop()
* If any code is included that the original author did not write, proper copyright licenses must be included

### Maintainability / Integration

* Blocks SHALL not depend on any library that GNU Radio does not already depend on
* Blocks SHALL come with a GRC file that allows usage in GRC
* Blocks SHALL have exactly one functionality (no "this block does sine/cosine/tan/exp based on what you select")
* Changes MUST NOT break existing OOTs that rely on GNU Radio's public API

### Common interfaces
* Blocks SHALL use the standard interfaces that are used throughout GNU Radio
* For streams, this SHALL only include any of
 * float32 for real-valued data / samples
  * complex float32 for complex-valued data / samples
  * floating point types MUST NOT be used to carry discrete information (e.g. time stamps, indices)
  * byte (uint8_t) for digital data as unpacked bits in byte (LSB)
  * byte (uint8_t) for digital data as packed bits in byte (all bits)
  * (u)int8_t for quantized data / samples where sufficient
  * (u)int16_t for quantized data / samples where sufficient, but (u)int8_t is not
  * (u)int32_t for quantized data / samples where sufficient, but (u)int16_t is not
  * vectors of above types, iff those are to be interpreted as single data items (else using output_multiple is preferred)
* Message port names MUST be descriptive
* Message type SHALL be a dictionary that maps descriptive names to command parameters
