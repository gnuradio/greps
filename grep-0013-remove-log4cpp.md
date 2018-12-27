# GREP 0013 -- Replace log4cpp with something else and improve logging infrastructure

- Original Author: Martin Braun <martin@gnuradio.org>
- Champion: Martin Braun <martin@gnuradio.org>
- Status: Draft

History:
- 26-Dec-2018: Initial Draft

## Abstract

There are two main goals to this GREP:

- Ideally, remove a dependency (in this case log4cpp)
- Simplify the logging subsystem

Dependencies are the bane of our existence as GNU Radio maintainers. log4cpp in
particular is a dependency that we could eradicate easily.

There are two main options for replacements:
- Boost.Log
- Hand-roll something

Other options may also exist.

As for the logging subsystem itself, there are designs decisions that could be
revisited when touching the logging code, such as the way loggers are accessed
in blocks.

## Copyright / License

This GREP is licensed as CC-BY-ND.
Copyright 2018 The GNU Radio Foundation.

## Motivation

Fewer dependencies are always better, unless we take away functionality. In this
case, we could replace one dependency (log4cpp) with one that we already have
(Boost) without losing any functionality. Or, we find a better replacement than
log4cpp, although that would reduce the value of this GREP. An alternative to
Boost.Log is to do a UHD-like approach, where a only a simple logging system is
used, but is hand-written from scratch and has no dependencies.

Such a removal might also spark a discussion on the architecture of the logging
subsystem itself, and it makes sense to only touch the logging subsystem once.
During an update to the logging subsystem, the following questions should be
answered:
- Do we really need a separate logging path for debugging and regular operations
  given that we already have log levels?
- Does it provide all the features we need in various environments (e.g., in
  embedded applications, remote logging...)?

## Description

tbw

