# GRIPE 0000 -- The GRIPE Process

- Original Author: Martin Braun <martin@gnuradio.org>
- Champion: Martin Braun <martin@gnuradio.org>
- Status: Draft

History:
- 15-Oct-2016: Initial Draft

(may have multiple original authors)

## Abstract

As of 2016, the only ways to discuss changes within GNU Radio (either the
codebase, or any other aspect of the organization) were the bug/issue tracker,
or by simply emailing the project leads (or contacting them otherwise).
For anything large-scale, The GNU Radio organization needs a way to formally
changes publicly. A great example is the "Python Enhancement Proposal" (PEP)
process. This document borrows heavy from the PEP documentation.

The GRIPE system shall be GNU Radio's version of such an enhancement proposal
process.

(200 words)

## Copyright / License

This GRIPE is licensed as CC-BY-ND.
Copyright 2016 The GNU Radio Foundation.

## Motivation

As the GNU Radio organization grows, we need better ways to discuss and bring
forward changes. Bug trackers, mailing list threads, IRC discussions, etc. are
simply not suitable for well-documented, public discussions. In the following,
our solution shall be referred to as "the process".

Python has solved this by introducing the "Python Enhancement Proposal" (PEP)
process, which was originally introduced in 2001 and has served the Python
community well (see https://www.python.org/dev/peps/pep-0001/).
The GRIPE process is modeled heavily after the PEP process.

There are a few requirements for implementing the process:
- Enhancement discussions need to be publicly visible.
- Participation in the process must be simple and available to any community
  member.
- Outcomes of discussions need to be recorded for future perusal, regardless
  of the outcome.
- The process cannot be tied to any particular service.
- The GNU Radio leadership needs to stay in control of the process.

A git-based approach, using simple text files as a basis, as well as additional
tools such as issue trackers seems a suitable way to handle this.

## Description

### Types of GRIPEs

A GRIPE can be about any enhancement for GNU Radio, including, but not limited
to:

- Technical enhancements for GNU Radio, typically about changes to the
  codebase. Depending on the scope, a small bug report or feature request to
  the GNU Radio issue tracker may suffice, and in this case, a GRIPE may be
  rejected and redirected to the issue tracker.
- Changes to the GNU Radio organization itself.
- Informational GRIPEs. This can describe a situation or issue, but does not
  propose anything new or different. If an informational GRIPE is better
  suited as an enhancement to GNU Radio's documentation, a GRIPE may be
  rejected and redirected to the manual or other components of GNU Radio.

### The GRIPE committee (GRIPE-C)

By default, the GRIPE committee (GRIPE-C) consists of the following people
(note that multiple roles may be filled by the same person):

- The GNU Radio Foundation Board members
- The GNU Radio release manager
- The GNU Radio community manager

In addition, the GNU Radio leadership may choose any other individual to be
part of the committee. For anyone not on the list above, GRIPE-C membership
can be terminated any time by GNU Radio leadership.

For any decision that pertains to the technical side of GNU Radio, the
technical lead (CTO) of GNU Radio has the final say. The technical side
includes any changes to the codebase.
For any decision that pertains to the GNU Radio organization or any other
non-technical issue, the GNU Radio organization lead (CEO) has the final say.


### Life cycle of a GRIPE

A GRIPE comes to life after it was submitted (see the following section). At
first, a GRIPE is in "Draft" status (which is also declared at the beginning of
a file). Now, the draft is discussed and changes may be proposed. See the
following sections for details on the GRIPE submission and change processes.

The purpose of the "Draft" status is to determine if a GRIPE is suitable for
actual consideration within the project. The GRIPE-C can either decide that
this is the case, which will change the status to "Accepted", or not, which
will make the status "Rejected". The author can also decide that the GRIPE is
not suitable for further consideration, and set the status to "Withdrawn".
Rejected GRIPEs still stay part of the GRIPE repository.

When "Accepted", the next goal is to flesh out all of the open questions and
details, such that the proposal can be flagged as "Final". This status means
that whatever proposal is described in the GRIPE is meant to be implemented.

A special status, "Active", is meant for GRIPEs that are never meant to be
completed.

A GRIPE can be in any of the following statuses:

- Draft
- Rejected
- Accepted
- Withdrawn
- Final
- Deferred
- Active

The following diagram illustrates the process:
```
            +----> ACCEPTED +-> [Discussions] +----> FINAL
            |                                 |
 DRAFT +----+                                 +----> ACTIVE
   ^        |
   |        +-+--> REJECTED
   |          |
   |          +--> WITHDRAWN
   v
DEFERRED
```


### GRIPE Submission Process

In order to submit a GRIPE, the following process shall be followed:

1. The author of the GRIPE needs her own version of the GRIPE repository.
   If necessary, the repository needs to be cloned first.
2. A new GRIPE is added by copying the template (`gripe-template.md`) and
   creating a new GRIPE file. The filename must follow the following naming
   convention: `gripe-XXXX-keyword.md`. `XXXX` is the next available
   four-digit decimal number (every GRIPE has a unique number, counting upward).
   The keyword can be any suitable word (or even multiple words) that
   will help identify the content, but should be kept short to keep the
   filenames manageable.
3. The author fills out the template as explained.
4. When complete, the author submits a pull request against the GRIPE
   repository, with a single commit adding the new GRIPE file.
   If multiple people submit a GRIPE, and two GRIPEs end up with the same
   number, that is treated as a merge conflict and the original author of the
   GRIPE needs to resolve by picking the next available four-digital decimal
   number.
5. If suitable, the GRIPE is merged into the main GRIPE repository by a member
   of the GRIPE committee. The following requirements apply before a GRIPE can
   be merged:
   - The four-digit number is next available one
   - The template was filled out appropriately
   - It is a single commit
   - The GRIPE is actually relevant to GNU Radio, either technically, or
     pertaining to the organization itself.
   Whether or not these requirements are met is decided by any member of the
   GRIPE committee. If they are met, the pull request is accepted and the GRIPE
   is merged into the repository. At this point, the GRIPE is flagged as
   'Draft'.
6. A GRIPE may be rejected for any reason, but there are two main causes
   for rejection: Formal reasons (in which case the author is asked to fix
   those and resubmit), and when GRIPEs are deemed unfit or unsuitable (in
   which case they will not be considered for resubmission).
   Note: The idea is that there is a very low bar of adding GRIPEs into the
   repository.

Once a GRIPE is in the repository, it is open for discussion.

### Proposing changes

Until a GRIPE is finalized, changes may be proposed. Major changes are proposed
in form of pull requests on top of the original GRIPE. The issue tracker
associated with this repository may also be used, as well as any other features
provided (such as inline commenting, etc.).

### The Champion

For every GRIPE, there is one distinct owner, in the following referred to as
the "champion". It is the champion's responsibility to drive the GRIPE to
completion. GRIPEs may be transferred to another champion. The champion is not
necessarily the one doing the actual development.

### GRIPE template

When authoring a GRIPE, the current version of the template must be used
(file `gripe-template.md` in the GRIPE repository).
The template may change over time, and anyone can propose changes to the
template by either submitting an issue or bug report, or by submitting a
pull request. Changes to the template are reviewed and accepted or rejected
by the GRIPE-C in the same was as GRIPE pull requests.

When the template is updated, GRIPEs that were already submitted to not need
to update to the new version of the template, although GRIPEs currently under
discussion ("Draft" status) are encouraged to use the new template.
GRIPEs that are in closed status (either "Accepted" or "Rejected") should not
change their corresponding GRIPE files when the template is updated.

The file format is also defined by the template. As this GRIPE is being
written, the template is in Markdown format, with UTF-8 encoded text.

