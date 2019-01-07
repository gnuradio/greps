# GREP 0013 -- Replace log4cpp with something else and improve logging infrastructure

- Original Author: Martin Braun <martin@gnuradio.org>
- Champion: Marcus Müller <marcus@hostalia.de>
- Status: Active

History:
- 26-Dec-2018: Initial Draft
- 7-Jan-2018: Updated from various mailing list and chat discussions
- 11-Jan-2018: Made Active, made Marcus champion

## Abstract

There are two main goals to this GREP:

- Remove a dependency (in this case log4cpp)
- Simplify the logging subsystem

Dependencies are the bane of our existence as GNU Radio maintainers. log4cpp in
particular is a dependency that we could eradicate easily.

As for a replacement, we could use a third-party library, or hand-roll a GNU
Radio-specific logger.

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

### Requirements

- Any log message must have a severity and a "category" (or "source ID"). We
  already have this, and want to keep it.
    - The source ID for a block should be its block ID.
    - Furthermore, it might be nice to have the source ID follow the block
      hierarchy.
- Blocks will have a single logger going forward, as opposed to the debug vs.
  regular logger of the current implementation. Developers can use the DEBUG and
  TRACE log levels for their debugging.
- We absolutely want to make sure that log messages are properly interleaved.
  This requires some amount of serialization/thread handling in the logging
  subsystem.
- The I/O happening during a logging call (e.g., writing to stderr, writing to
  a file, writing to journald, etc.) should not cause a slowdown at the call
  site. Otherwise, we couldn't reliably log from within `work()` or
  `general_work()`
- By default, there should be log files and logging to stderr. However, we also
  want to enable (or at least, not disallow) other logging paths, such as
  journald, or network logging. This might be as simple as providing generic
  hooks for other loggers.
- If possible, we would also like to pull in log messages from dependencies, so
  they all go through the same logging interface. For example, UHD log messages
  could be redirected to the GNU Radio logger when running gr-uhd applications,
  so UHD and GNU Radio don't clobber each other with different formats of their
  logging messages.
- On top of severity, category, and log message, a logger should have access to
  call site information (line number, file, function), thread ID, and timestamp.
- Logging to stderr needs to have optional colours, because without colours,
  logging is lame.

Note: As of Jan 2019, we would like to drop any kind of dependency and hand-roll
our own logger.

### Detailed Design

The requirements drive some design decisions directly:

- There will be one single "root logger". All other logger objects (such as a
  logger object attached to a block) will be *childs* of the root logger.
- A child logger has a fixed source ID.
- Child loggers can have child loggers of their own.
- The root logger must implement a thread-safe multi-writer queue for log infos.
  It will serially handle the actual logging, which means the logging is not
  fully synchronous -- but it also means that the publishing of the log messages
  is not handled by the thread that is producing it, which takes I/O waits out
  of the log call sites.
    - We will most likely start with the dumbest form of such a queue, probably
      a `std::queue` with a mutex and a notification variable. We can optimize
      later, if need be. Lockfree data structures tend to be worse for best-case
      and better for worst-case latency, but only some research would have that
      kind of answer
- Every child logger can filter log messages by severity, allowing different
  log levels for different blocks (e.g., only those blocks currently being
  debugged).
    - The root logger stores a default log level, but does it filter based on
      the log level? That's up for debate.


### Motivation for not using a third-party logging framework

Logging is sometimes cited as a prime example for something one shouldn't write
themselves, so the question remains, why not use an existing framework?
The reasons are manifold:

- It is our general desire to reduce the number of dependencies. The current
  logging library (log4cpp) has not been updated in a while, and we run risk of
  it breaking at some point with no upstream maintainers available.
- Logging implementations always have specific quirks for the project they're
  running in. This means there's always a lot of code to configure the
  third-party library, which in some cases exceeds the amount of code it takes
  to write one's own logging subsystem.
- The requirements for the GNU Radio logging subsystem are independent of the
  implementation, including the choice of the logging library. Out of the
  available third-party logging libraries, we would have to find one that
  satisfies all of our requirements, as well as being actively maintained.

A list of C++ logging frameworks is available here: https://github.com/fffaraz/awesome-cpp#logging


