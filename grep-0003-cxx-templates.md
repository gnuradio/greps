# GREP 0003 -- C++ templates in GNU Radio

- Original Author: Andrej Rode <mail@andrejro.de>
- Status: Draft

History:
- 02-Mar-2018: Initial Draft

## Abstract

As outlined in the [discussion on the mailinglist](https://lists.gnu.org/archive/html/discuss-gnuradio/2007-08/msg00361.html) C++ templates are supported in SWIG for quite some time.

Leveraging C++ templates will allow to use code checking tools to perform checks before compilation is performed.

This will result in a GNU Radio source tree containing only valid `Python` and `C++` source code.

## Copyright / License

This work is licensed under a Creative Commons Attribution-ShareAlike 4.0 International License.

## Motivation

It is cumbersome to review code generated in the process of compilation. Reducing the amount of generated code by templating methods available in `C++` will close yet another source of possible bugs. Also regeneration of custom templated code is not always performed if template code is changed, This slows down development as is more error prone compared to using `C++` templates directly.

## Description

This change is meant to be implemented by leveraging automatic tools to perform conversion across the full GNU Radio source tree.
Necessary steps to perform conversion:
 - Walk through the full source tree and identify block names and generated types
 - For each block name move implementation & header from `${block_name}_{v,vX,X,XX,}X.{c,h}.t` to `${block_name}.{h.c}`
 - Fill with appropriate values:
   - `@GUARD_NAME_IMPL@`
   - `@GUARD_NAME@`
   - `@NAME_IMPL@`
   - `@NAME@`
 - If `${block_name}` is a reserved `C++` keyword fill `@NAME@` with `${block_name}_blk<IType{, OType}>` otherwise use `${block_name}<IType{, OType}>` (Leave out <Type> for class declarations).
 - Decorate each member function with `template <class IType{, OType}>` and fill `@O_TYPE@` and `@I_TYPE@` accordingly to defined input and output types.
 - Add `typedef ${block_name}_{v,vX,X,XX}X template class ${block_name}{_blk}<IType{,OType}>` to adhere to previous block naming
 - If necessary: instatiate each currently defined block_type in `${block_name}_impl.cc`

This will be implemented in a Python tool giving opportunity to `OOT` authors to auto-convert custom templating to `C++` templates.
