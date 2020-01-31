# GREP 0001 -- Coding Guidelines

- Original Author: Martin Braun <martin@gnuradio.org>,
                   Marcus Müller <mmueller@gnuradio.org>
- Champion: Marcus Müller <mmueller@gnuradio.org>
- Status: Active

History:
- 12-Feb-2018: Initial Draft
- 6-May-2018: Cleanup, turned active

## Abstract

This document establishes coding guidelines for the GNU Radio main source tree.

As a recommendation, these coding guidelines should also be adopted by
out-of-tree modules.

## Copyright / License

This GREP is licensed as CC-BY-ND.
Copyright 2018 The GNU Radio Foundation.

## Motivation

Common coding guidelines are the grease that makes a software project's
cogwheels spin freely. They remove friction, and make certain unproductive
discussions unnecessary. In particular because coding styles are subjective, it
helps a great deal to simply declare one coding style as the one.

New code should follow these rules, but there are always exceptions, and it is
up to the individual contributor to decide what works best. That said, certain
things such as indentation or whitespace rules can be taken care of by one's
editor, and there is no reason to not adopt these settings. This makes it
easier for multiple people to contribute to the same codebase, and will keep
commits free of non-functional changes.

To summarize, consistency is extremely useful, and coding guidelines help ensure
consistency. We assume that all contributors are mature enough to at least try
and follow them.

## Description

### General Coding Guidelines

* Code is read more often than it's written. Code readability is thus something
  worth optimizing for. All guidelines thus become secondary to better
  readability.

### General Formatting Guidelines

* Never submit code with trailing whitespace.
* Try and keep line lengths short, unless readability suffers. We often have to
  do fun things like SSH into machines and edit code in a terminal, and do
  side-by-side views of code on small-ish screens, so this is actually pretty
  helpful. Maximum line length is 100 characters, but keeping lines below 80
  characters where sensible is a good guideline.
* Always end a file with a newline.

### C++-specific Guidelines

#### Implementation Guidelines

* If in doubt, consult the [C++ Core Guidelines](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md).
  If the guidelines have an answer, and it works for you, just pick that.
  Rationale: Many things are subjective and it's a waste of energy discussing
  those.
* All things equal, prefer standard C++ constructs over Boost constructs (see
  also Boost guidelines). Rationale: Boost is a 3rd party dependency, and we
  want to keep those at a minimum. Also, Boost regularly makes
  backward-incompatible changes, making supporting a wide range of Boost
  versions hard.
* Given the option, prefer C++ lambdas over std::bind, and just don't use
  boost::bind if you can. Rationale: C++ lambdas are a compiler construct, and
  can thus be optimized better at compile time. In many cases, lambdas are
  also more readable than the `bind` calls, which have add their own
  syntactical idiosyncrasies (such as the `_1` argument etc.).
* Feel free to use modern C/C++ features even if they were not used before.
  Make sure they work with the compilers and dependencies which are set for the
  version of GNU Radio the commit will be made upon. The GNU Radio CI system
  will be able to confirm this. (Note: C++11 features are available starting
  with GNU Radio version 3.8).

#### Code Formatting Guidelines

* Use the `.clang-format` file provided with the source tree. The following
  items describe some of the major formatting rules.
* Do not indent namespaces
* Use 4 spaces indentation width for new code.
* Use spaces instead of tabs. If extending an existing file stick with the given
  indentation level. Do _not_ reindent a whole file and indent code consistently.
* Before checking in code use tools available in the source tree to sanitize
  your patches
* Use Doxygen doc-blocks copiously. This makes it easier to maintain
  auto-generated documentation.
* Include include files in the following order: Local headers, other GNU Radio
  headers, 3rd-party library headers, Boost headers, standard headers.
  The rationale is to include from most to least specific. This is the best way
  to catch missing includes (if you were to include the standard header first,
  it would be available to all include files that come later. If they need that
  standard header too, they should be including it themselves).
  Example:

```cpp
#include "module_specific.hpp"
#include <gnuradio/block.h>
#include <fftw3.h>
#include <boost/shared_ptr.hpp>
#include <mutex>
```

* Macros and constant values (e.g., enumerated values, static const int FOO=23)
  shall be in `UPPER_CASE`.
* All other identifiers are in STL style, using underscores (e.g., `get_x_value()`).
  Avoid using `CamelCase()`.

#### Classes

- All class data members shall begin with d_`foo`

The big benefit is when you're staring at a block of code it's obvious
which of the things being assigned to persist outside of the block.

- All class static data members shall begin with s_`foo`.
- Each significant class shall be contained in its own file.
- Prefer non-static data member initialization (NSDMI) where possible


#### Boost-specific Guidelines

* Avoid Boost where possible (see C++ guidelines for rationale).
* Don't use Boost's sleep functions. Prefer `std::chrono` functions.

#### On the usage of clang-format

`clang-format` is a command line tool that can directly be applied to any file
by running

    clang-format -i $filename

However, the recommended way of using clang-format is either to integrate it
into your editor of choice (most popular editors have clang-format integration,
see also https://clang.llvm.org/docs/ClangFormat.html), or to enable a git
pre-commit hook (See https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks
for git hooks; there are different ways to configure a pre-commit hook for
clang-format and examples can be found all over the internet) which will verify
that all code that is being committed has been correctly formatted, or both.
By correctly formatting code before submitting it upstream as a patch or pull
request, you will save both yourself and the maintainers of GNU Radio some time,
so please use these tools to decrease the workload on everyone involved.

## Python-specific Guidelines

* Follow the suggestions in PEP8 (https://www.python.org/dev/peps/pep-0008/)
  and PEP257 (https://www.python.org/dev/peps/pep-0257/). The former is about
  code layout in general, the latter about docstrings.
* Keep Python code compatible with Py2k and Py3k, where possible. This allows
  code to be reused more easily in environments that run different versions of
  Python.
* Pylint is good tool for helping with following code guidelines. It's very
  fussy though, so don't get too worked up about following its suggestions.

### Python modules

* Only use non-core Python modules if they are already listed as dependencies
  for GNU Radio. Mostly, this implies to only rely on `NumPy` and core Python
  libraries. Otherwise, issues related to missing modules pop up repeatedly on
  the issue tracker.
* Always keep in mind that modules like `SciPy` and `matplotlib` are optional.
  If such a module is used without checking availability first, this will break
  things for at least some users.

## CMake Files

* The formatting guidelines also apply to CMakeLists.txt files
* CMake commands are kept lowercase, while custom GNU Radio macros are kept in
  uppercase. We treat CMake as if it were case-sensitive.

## Revision Control Guidelines

* In this repository, we almost always use fast-forward merges, and no merge
  commits. Fast-forward commits ensure that multiple commits belonging to one
  development streak stay together, making it easier to follow the development
  history and do git bisects.
* Before opening a request for merging your code, make sure to rebase your
  changes against the current state of the branch you forked off your own
  branch (typically, that's master, so that procedure would amount to `git fetch
  origin; git rebase origin/master`).
  That way, you keep control of how your code integrates into the common code
  base! If everything goes well, rebasing requires absolutely no interaction on
  your side. If something goes wrong, fix it like you'd fix a merge conflict.
  It's the same effort as fixing a merge conflict, and avoids one of those
  later on, but puts you in command over the order of lines, and enables you
  (who's the most knowledgeable expert on your own code) to spot problems that
  through naive merging by a maintainer would have become a bug. It also
  reduces the workload on the maintainer, and thus expedites merging of your
  code and other's.
* Prefix all commit message subject lines with the section of code they apply
  to, and use the imperative mood (Example: "blocks: Fix buffer overflow in
  vector sink").
  Try and keep the subject line to 50 characters, but make 72 characters a hard
  limit (see below for rationale).
* Follow up in greater detail in the body of the commit message. The body is
  separated from the subject line with one blank line. Consider the body of the
  git commit an email to the future reader of this changeset, so don't be
  sparse in the commit body, and use it to explain the *what* and *why* of this
  commit (the "how" part should be obvious from the code change). Lines should
  be limited to 72 characters in length.
  Rationale: This is a standard way of formatting git commits, other projects
  have this as a hard requirement, which makes switching between projects
  easier. See also the discussion on code line lengths (see general formatting
  guidelines). Finally, keeping lines shorter than 75 characters is generally
  considered to be more readable.
* Avoid refactoring, whitespace cleanup, or fixing code to match coding
  guidelines in the same commit as modifying behaviour. Prefer dedicated
  cleanup commits.
* Remember that we create git repositories, not just code. This means every
  commit is part of our project and should be treated as such.


## Comments

* When working in code, there may be additions or things you might not know how
  to do just yet. It is appropriate in these instances to make a comment about
  it. The convention for this is to use the following keywords to indicate where
  you have left something for later:
  * Use TODO comments for low-priority fixes
  * Use FIXME comments for high-priority fixes
