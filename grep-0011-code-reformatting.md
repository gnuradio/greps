# GREP 0011 -- Code Reformatting

- Original Author: Andrej Rode <mail@andrejro.de>
- Champion: Andrej Rode <mail@andrejro.de>
- Status: Active

History:
- 02-Mar-2018: Initial Draft
- 06-May-2018: Cleanup and made active
- 27-Dec-2018: Added decision that was made to do a mega-commit before 3.8

## Abstract

Once GREP 0001 is in effect the question arises how to proceed with regard to
code formatting.
This GREP will present multiple ways going forward with code formatting from
which the most feasible way is picked and proposed.

Some research about FOSS projects which have completed codebase reformatting
highlights two big projects which successfully finished a big code reformatting:

1. [MongoDB](https://engineering.mongodb.com/post/succeeding-with-clangformat-part-1-pitfalls-and-planning/) which also has a conclusive writeup with a rationale and how formatting was
implemented.

2. [OpenSSL](https://www.openssl.org/blog/blog/2015/02/11/code-reformat-finished/).

Both projects reformatted the full source tree at once to reduce cleanup noise.

**Addendum:** The GNU Radio leadership has decided to apply one reformatting
mega-commit before the tagging of release 3.8.0.0.

## Copyright / License

This work is licensed under a Creative Commons Attribution-ShareAlike 4.0
International License.

## Motivation

Cleaning up the GNU Radio codebase will help current and future contributors to understand code in the
GNU Radio source tree more quickly and will help speed up development.
Also, having a clean formatted codebase will allow maintainers and developers
to use automated tools for reviewing changes.

## Description

Several ways to clean up the codebase according to the Code Formatting agreed
upon in GREP 0001.

### 1. Don't clean up at all

Apply code formatting rules only to new development. Changes to existing source
code have to be formatted according to the formatting done in its section or
file. This will result in inconsistently formatted existing source code as-is.

### 2. Clean up files that are touched during development

Everytime changes are introduced to a file during bug fixing or feature
development add a separate cleanup commit reformatting the file. This will
result in files being consistent after they have been edited.

### 3. Clean up the full source tree at once

During a time of low development (low count of pull requests and feature
branches) apply a reformatting across the full source tree.
This will result in a fully cleaned source tree.

**Addendum:** The GNU Radio leadership team has decided to go ahead with method
3 (i.e., apply a reformatting mega-commit). As of December 2018, the GNU Radio
repository has tools in place to ease the rebasing of previously written pull
requests.

### Comparison

Each method of cleaning the source tree has pros and cons. Method 1 represents
current state of development. Method 2 will incrementally clean up the tree
until every file was touched once, but also will result in huge commit noise
with a separate commit for each bugfix and new feature introduced while breaking
history as well.

Method 3 will result in a history reset across the full source tree while having
the least commit noise. Both Method 2 and Method 3 will break history by
reformatting source files as whole.

To ease maintenance and further development in GNU Radio not cleaning up the
source tree is not an option.

### Conclusion

This GREP proposes to perform a full codebase reformatting (Method 3) according
to standards set in GREP 0001 and by specifiying and documenting further
formatting decisions in a `.clang-format` in the source tree. Until the
reformatting can be implemented several major issues have to be resolved:
- Provide sufficient tools for contributors and developers to format new patches
  and additions according to the GNU Radio coding standards
- Prepare code reformatting carefully by writing wrapper around tools to format
  all parts of the source tree without destroying intentionally formatted parts
  (Flow charts in comments etc.).
- Pick a time of low development activity to push formatting changes on all
  development branches

To have sufficient diversity in the discussion about crafting a `.clang-format`
at least 3 developers should be part of creating it and further tools to format
the GNU Radio source tree.

Further on each pull-request and patch has to be formatted using the tools in
the GNU Radio source tree before it can be merged.

**Addendum:** GREP 0001 has been finalized, and Method 3 will indeed be the
weapon of choice.

### Timeline

As of the first draft of this GREP, there were still multiple branches of GNU
Radio (`next`, `python3`, etc.) which all had to be merged. In September 2018,
all branches were finally merged into the `master` branch, enabling the
finalization of the reformatting effort.

In December 2018, the final touches to the clang-format file and accompanying
tools were applied. In a separate effort, the path towards a 3.8 release was
decided. The GNU Radio leadership team thus decided to apply one big
reformatting mega-commit to all C++ code shortly before the finalization of the
3.8.0.0 release of GNU Radio. The exact point in time will be determined once
the first release candidate of GNU Radio is ready to be tagged by the
maintainer.

The champion of this GREP will then apply the changes.

