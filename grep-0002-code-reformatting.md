# GREP 0002 -- Code reformatting

- Original Author: Andrej Rode <mail@andrejro.de>
- Status: Draft

History:
- 02-Mar-2018: Initial Draft

## Abstract

Once GREP 0001 is in effect the question arises how to proceed with regard to code formatting.
This GREP will present multiple ways going forward with code formatting from which the most feasible way is picked and proposed.

Some research about FOSS projects which have completed codebase Reformatting gives two big projects which successfully finished a big code reformatting recently.
One project being [MongoDB](https://engineering.mongodb.com/post/succeeding-with-clangformat-part-1-pitfalls-and-planning/) which
also has a conclusive writeup with a rationale and how formatting was implemented.
Another big project which now has auto-formatted code is
[OpenSSL](https://www.openssl.org/blog/blog/2015/02/11/code-reformat-finished/).

Both projects reformatted the full source tree at once to reduce cleanup noise.

## Copyright / License

This work is licensed under a Creative Commons Attribution-ShareAlike 4.0 International License.

## Motivation

Cleaning up the GNU Radio codebase will help current and future contributors to understand code in the
GNU Radio source tree more quickly and will help speeding up development.
Also having a clean formatted codebase will allow the maintainers and developers to use automated tools
for reviewing changes. The way to achieve a clean codebase might be not clear.

## Description

Several ways to clean up the codebase according to the Code Formatting agreed upon in GREP 0001.

### 1. Don't clean up at all
Apply code formatting rules only to new development. Changes to existing source code have to be formatted according to the
Code formatting done in its section or file. This will result in inconsistently formatted existing source code as-is.

### 2. Clean up files that are touched during development
Everytime changes are introduced to a file during bug fixing or feature development add a separate cleanup commit reformatting
the file. This will result in files being consistent after they have been edited.

### 3. Clean up the full source tree at once
During a time of low development (low count of pull requests and feature branches) apply a reformatting across the full source tree.
This will result in a fully cleaned source tree.

### Comparison
Each method of cleaning the source tree has pros and cons. Method 1 represents current state of development. Method 2 will
incrementally clean up the tree until every file was touched once, but also will result in huge commit noise with a separate commit for
each bugfix and new feature introduced while breaking history as well.
Method 3 will result in a history reset across the full source tree while having the least commit noise. Both Method 2 and Method 3 will break
history by reformatting source files as whole.

To ease maintenance and further development in GNU Radio not cleaning up the source tree is not an option.

### Conclusion

This GREP proposes to perform a full codebase reformatting according to standards set in GREP 0001 and by specifiying and documenting further formatting decisions in a `.clang-format` in the source tree. Until the reformatting can be implemented several major issues have to be resolved:
 - Provide sufficient tools for contributors and developers to format new patches and additions according to the GNU Radio coding standards
 - Prepare code reformatting carefully by writing wrapper around tools to format all parts of the source tree without destroying intentionally formatted parts (Flow charts in comments etc.).
 - Pick a time of low development activity to push formatting changes on all development branches

To have sufficient diversity in the discussion about crafting a `.clang-format` at least 3 developers should be part of creating it and further tools to format the GNU Radio source tree.

Further on each pull-request and patch has to be formatted using the tools in the GNU Radio source tree before it can be merged.
