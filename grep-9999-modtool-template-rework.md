# GREP [XXXX] -- [Rework gr_modtool's templating]

- Original Author: Marcus Müller <mmueller@gnuradio.org>
- Champion: Marcus Müller <mmueller@gnuradio>
- Status: Draft

History:
- 2021-11-29: Initial Draft

## Abstract

Untangle modtool's templates from strings embedded in Python code, rendering of
old-style format strings (`"Copyright %s" % datetime.now().year`), Mako
templates and manual replacements. Get rid of cruft.

## Copyright / License

 GREP "Rework gr_modtool's templating" by Marcus Müller is licensed under CC BY
 4.0. To view a copy of this license, visit
 http://creativecommons.org/licenses/by/4.0/
 
## Motivation

GNU Radio's modtool has quite some history and has come from the need to
automate the copying of a 3.5/3.6-era "square_ff" howto OOT to a comprehensive
tool that does much more, including functionality to parse and generate program
code.

It also had to go through a Python version change, be migrated away from
Cheetah, and grew alongside with its developers.

As a result, many provisionary solutions became state of the art. Over time,
this introduced a couple subtle and not-so-subtle bugs.

A cleaner structure for templates should not only simplify development, but also
reduce the likelihood of mistakes.

## Description

### Cruft Removal

Remove all pre-3.9 templating that would need to be reworked.

### Templating unification

Code for blocks, headers, python files etc is mostly found under
`modtool/templates/templates.py` as strings.

* Make a new directory `modtool/templates/templates`
* Write the strings from `templates.py` into individual `something.mako` files
  in that directory
* Install that directory alongside with `gr-newmod`
* Use Mako's `TemplateLookup` functionality to directly look up templates in
  that directory
* To enable in-tree testing, make the lookup order such that `module source
  directory/templates/templates` is visited first, then the installed location
  (`$GRDATA/templates`)

Furthermore, these templates currently are structured to contain the "common"
parts, and then a lot of `if/else` clauses on the individual block types.

* Replace the convoluted `if/else` structure with Mako `<%block
  name="snippet_name">` in the "master[^1] template", which the actual block
  types specialize through Mako's `<%inherit file="master.mako">`
  [mechanism](https://docs.makotemplates.org/en/latest/inheritance.html).


[^1]: "master" as in "slide master", "original mold"
