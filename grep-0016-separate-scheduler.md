# GREP 0016 -- Separate GNU Radio Base Code from the TPB Scheduler

- Original Author: Martin Braun <martin@gnuradio.org>
- Champion: Martin Braun <martin@gnuradio.org> (Looking for a new champion!)
- Status: Draft

History:
- 26-Dec-2018: Initial Draft

## Abstract

**Note: This is a very raw GREP. But we really want to do this, and here's the
GREP so we have a platform for a discussion.**

The GNU Radio scheduler could use some attention. It hasn't significantly
changed in a long time. However, how do we touch such a major part of GNU Radio?

The answer is to first make the current scheduler a swappable module. That way,
other schedulers could become developed the same way out-of-tree modules get
developed, and there could be a healthy competition between schedulers.

Besides enabling a Battle Royale of schedulers, this could also enable custom
schedulers for specific hardware.

## Copyright / License

This GREP is licensed as CC-BY-ND.
Copyright 2018 The GNU Radio Foundation.

## Motivation

In the last years, a lot of code for GNU Radio was developed. However, the core
of GNU Radio itself, the scheduler, has received very little attention. One of
the reasons for this is that changing the scheduler is both difficult, since it
is an under-documented, but still complicated piece of machinery, but also it is
so central to GNU Radio that changing it could have all sorts of ramifications.
Many attempts to improve the scheduler in GNU Radio have fizzled out, in some
cases simply because going into that part of GNU Radio is a tough nut to crack.

Since we all know there are improvements to be made to the current scheduler
(e.g., make messages a first-class citizen, enable more heterogeneous hardware
architectures, etc.), we first need to enable this kind of change. Step zero of
creating the next great scheduler thus must be: Refactor GNU Radio itself to
enable this kind of development.

Splitting the actual scheduler out of the base component of GNU Radio would
most likely require some changes to other base classes, such as the block
itself, the block details, and certain relationships between blocks. Thinking of
the scheduler and the GNU Radio blocks as more separate entities might even
force us as the maintainers to re-think and re-design interfaces and
architectures. Cleaner, more separate block interfaces would certainly
facilitate the integration of blocks into more and different pieces of other
software, further increasing the value and the shareability of GNU Radio blocks.


## Description

**To be written**. Some early thoughts:

- Whatever we do, the basic use case of clicking around in GRC and clicking
  'run' must not change
- We could write a very simple scheduler (maybe a version of the old STS) for
  demo purposes, unit testing... etc.
- We'll still need a core library, but `gnuradio-core` is a burnt name. Maybe
  `gr-base` or `gnuradio-base` (for stuff like basic block parent classes,
  types, etc.). Then we could have one component per scheduler: gr-tpb, gr-sts,
  etc.
- Other things that belong into the base:
  - A definition / interface of a `top_block`
  - PMTs
  - Concepts of blocks (e.g. sync block, interpolator/decimator blocks)
