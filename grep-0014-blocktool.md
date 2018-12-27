# GREP 0014 -- Implement a block parsing library (blocktool)

- Original Author: Martin Braun <martin@gnuradio.org>
- Champion: Martin Braun <martin@gnuradio.org> (Looking for a new champion!)
- Status: Draft

History:
- 26-Dec-2018: Initial Draft

## Abstract

GNU Radio's value comes from many things, but one of them is vast library of
blocks, both in-tree and out-of-tree. Right now, it is hard to use these blocks
in a different context than GNU Radio itself, or interface with blocks
automatically or programmatically.

We therefore propose the creation of Python-based tools (blocktool) that can
interact with block headers written in C++ or Python, to automatically parse
them and extract information about them such as which getters/setters they have,
what kinds of in- and outputs, documentation, licensing, etc.

Such a tool could be useful for GNU Radio itself, since we could parse block
headers and convert the information into other formats. For example, a tool that
automatically creates GRC bindings from block headers, with full documentation,
would become trivial to implement. Third-party libraries could also
automatically generate wrappers or other kinds of interfaces, as well.

## Copyright / License

This GREP is licensed as CC-BY-ND.
Copyright 2018 The GNU Radio Foundation.

## Motivation

GNU Radio blocks have multiple descriptions: The most direct is the actual
source of the block header file (or the block itself, if it's a Python block).
The header usually contains all sorts of information: The block documentation,
the factory functions, getters and setters (also with documentation), the block
type, etc. However, there is no unified way to get all this information. The
documentation is extracted using Doxygen. gr-modtool has some rudimentary C++
parsing methods. Also, there exists a totally separate, but not totally
orthogonal other file: The GRC bindings. And of course, SWIG also parses the
block headers to auto-generate its own description of the block.
So even within GNU Radio, there are already multiple files to describe a single
block.

This unnecessary duplication is even worse for projects not within the core GNU Radio ecosystem. What if a tool would like to auto-generate wrappers for blocks,
e.g., to access them from yet another programming language? Or what if users
would like to add 3rd-party tools? And maybe GRC bindings could be more easily
written, too.

A single tool, which is part of GNU Radio, but runs independently of most of the
code would be a great addition to the set of tools.

## Description

**To be written**. Some high-level thoughts so far:

- `blocktool` should be able to run standalone, and work with existing blocks
   without having to modify them
- It might be necessary to add markup or something like that to blocks to get
  full descriptions automatically, that might be considered OK as long as it's
  not mandatory.
- We might want to return a property-tree like structure for a block description
  which can be traversed similarly to XML trees.
- Write it in Python, because we don't want to add another language to the GNU
  Radio mix, and C++ is not a great choice

