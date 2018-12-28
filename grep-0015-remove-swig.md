# GREP 0015 -- Replace SWIG with something else

- Original Author: Martin Braun <martin@gnuradio.org>
- Champion: Martin Braun <martin@gnuradio.org> (Looking for a new champion!)
- Status: Draft

History:
- 26-Dec-2018: Initial Draft

## Abstract

**Note: This is a very raw GREP. It's a discussion we've been meaning to have
for a while. This GREP will stay here for as long as this repo will exist, even
if the out come of the discussion is to not replace SWIG.**

SWIG is a huge dependency. We might be able to do better. One option is to use
Boost.Python, since Boost is already a dependency. Potential gains:
- Faster compile times
- One fewer dependency. Our CMake would get a lot saner.
- It's possible that writing special-case wrappers would become easier

There is a major impact of this: We would no longer have the option to wrap
our C++ code into another language. However, we've never used this feature
anyway.


## Copyright / License

This GREP is licensed as CC-BY-ND.
Copyright 2018 The GNU Radio Foundation.

## Motivation

A lot of us have cursed SWIG programming. If we can get rid of it... well,
wouldn't that be great. But maybe we can't. Let's find out.

## Description

**To be written**. GREP14 (blocktool) would be a help.

Options for a replacement:
- Boost.Python with auto-generated files (blocktool could be useful here)
- PyBind11 is similar to Boost.Python. It's header-only and could be shipped
  with GNU Radio instead of making it a dependency. It would still require
  auto-generated files as with Boost.Python.
