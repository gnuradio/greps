# GREP [XXXX] -- Upstreaming Process


- Original Author: Marcus Müller <mmueller@gnuradio.org>
- Champion: Marcus Müller <mmueller@gnuradio.org>
- Status: Draft

History:
- 15-Feb-2017: Initial Draft
- 05-Mar-2018: Conversion to GREP process

## Abstract

This GREP+ strives to establish a guideline for

* when to submit blocks for becoming part of the upstream GNU Radio tree and
* how to ensure maintenance and maintainability

## Copyright / License

[WTFPL](http://www.wtfpl.net/txt/copying/) Version 2 or later

## Motivation

GNU Radio calls itself the "Free and Open Software Radio Ecosystem". We're not
doing that bad with ecosystem size, but the code quality in Out-Of-Tree Modules
(OOTs) is very varying, and with every release, we risk serious bitrot, since
there's no "maintained" upstream for any of these -- they are 100% in the care of
their original authors.

For standalone, self-contained OOTs, this is a fine solution; for utility
modules like the popular gr-foo, this becomes problematic. Also, most authors
just prefer to write the utility blocks they need for themselves, and thus,
these are never seen by other GNU Radio users or OOT authors.

Compare the Linux Kernel: There's extensive infrastructure within the kernel,
yet code quality is high, and aside from the comparatively rare slip-up, no
kernel gets released that breaks userland or has components that ceased to work.

The Linux kernel contribution model is centralistic: There's but one kernel.
Sure, you can fork it, add your device drivers to it, and then hope it'll be
forward-portable easily in the future, but the preferred way is to upstream your
code.

The idea is that as soon as it's upstreamed, the subsystem maintainer has gotten
the responsibility to keep it functional. That way, if anything within the
kernel changes, there's someone with both knowledge and responsibility that will
make your code work with the change.

In the long run, GNU Radio should come with batteries included -- not a
ready-to-be-used implementation of all possible wireless standards and methods,
but giving its users all the tools known in the GNU Radio ecosystem to maximize
usability.

## Description

This document strives to establish a common goal of integrating new
functionality or functional enhancements into the GNU Radio upstream. This
document is not meant to describe the *only* approach on how to contribute to
the GNU Radio main tree; it revolves primarily on adding new blocks to GNU
Radio's main tree.

If a change goes deep and changes behavior of the runtime / scheduler, this
document does NOT describe the proper way of improving GNU Radio. GNU Radio is a
very community-driven project. Complex changes are best discussed on the mailing
list, and especially in the form of GREPs.

This document does not apply to bug fixes.

This document does not apply to documentation enhancements.

The last word is with the maintainers, as they will take over responsibility and
make the code work in the future. Hence, the criteria below must be understood
as necessary, but not sufficient for upstreaming change.

### Upstreaming Criteria

As said above, these are guidelines for *necessary* criteria. The maintainers
are the ones taking care of the code later on -- they can freely choose not to
upstream any changes that they feel incapable of maintaining, contradict goals
of the project, or simply do not fit their opinion of what should be part of the
main tree or not.

#### Objective

The objective is to make reusable, high-quality, useful code part of the
Runtime, and keep it there.

#### Legal Criteria

The Copyright Requirements of the GNU Radio Project have to be met. That means,
at this time, that the author of code needs to assign copyright to the FSF.

#### High Reusability

* I.e. implementations of specific standards are NOT ELIGIBLE for upstreaming
* Basic math operations are ELIGIBLE
* Widely adopted algorithms are ELIGIBLE
* Adapters to commonly used external tools are ELIGIBLE
* Additions / Modifications to the overall operation of GNU Radio and its
  runtime / scheduler are ELIGIBLE

Here, subjective judgment of the maintainers is especially relevant and can
overrule any concern given above or in the community.

Good reasons to *not* upstream change is that certain things are really better
off in an out-of-tree module.

#### Code Quality / Maintainability

GREP0001 (Coding Guidelines) applies without any restriction. Objective is to
make new code stand out in quality.

The community strives to maintain reference to an official tool to check patches
for conformity to coding style guidelines (compare checkpatch of the Linux
kernel).

### Upstreaming Process

1. The party interested in upstreaming MUST integrate their changes into a fork
   of GNU Radio on a publicly available git repository (classically: Github),
   based on the HEAD of either the current "master" or "next" branch. The branch
   name MUST be descriptive of the proposed change.
2. The party MUST write an email to the discuss-gnuradio@gnu.org mailing list
   from the email address used as author in the commits.
   If there's multiple git authors, one of these email addresses SHALL be used,
   informing the community of
  * The existence and URL of their fork,
  * the purpose of the change,
  * the name of the branch,
  * the scope of the changes.
3. If necessary, FSF copyright assignment agreements MUST be arranged
4. During the process of upstreaming, the original author(s) SHALL offer a
   reliable contact. The process consists of
  * Community and Maintainer review of the changes
  * Feedback by the Maintainer
  * Working said feedback back into the proposed changeset
5. The GNU Radio release manager, upon completion of merging the new
   functionality, SHALL add a remark describing the change to the GNU Radio
   CHANGELOG
