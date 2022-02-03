# GREP 0000 -- The GREP Process

- Original Author: Martin Braun <martin@gnuradio.org>
- Champion: Martin Braun <martin@gnuradio.org>
- Status: Active

History:
- 15-Oct-2016: Initial Draft
- 31-Jan-2018: Updated and renamed to GREP, changed numbering assignment
- 3-Feb-2022: Updated language to reflect organization updates

(may have multiple original authors)

## Abstract

As of 2016, the only ways to discuss changes within GNU Radio (either the
codebase, or any other aspect of the organization) were the bug/issue tracker,
or by simply emailing the project leads (or contacting them otherwise).
For anything large-scale, The GNU Radio organization needs a way to formally
changes publicly. A great example is the "Python Enhancement Proposal" (PEP)
process. This document borrows heavy from the PEP documentation.

The GREP system shall be GNU Radio's version of such an enhancement proposal
process.

## Copyright / License

This GREP is licensed as CC-BY-ND.
Copyright 2016 The GNU Radio Project.

## Motivation

As the GNU Radio organization grows, we need better ways to discuss and bring
forward changes. Bug trackers, mailing list threads, chat discussions, etc., are
simply not suitable for well-documented, public discussions. In the following,
our solution shall be referred to as "the process".

Python has solved this by introducing the "Python Enhancement Proposal" (PEP)
process, which was originally introduced in 2001 and has served the Python
community well (see https://www.python.org/dev/peps/pep-0001/).
The GREP process is modeled heavily after the PEP process.

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

### Types of GREPs

A GREP can be about any enhancement for GNU Radio, including, but not limited
to:

- Technical enhancements for GNU Radio, typically about changes to the
  codebase. Depending on the scope, a small bug report or feature request to
  the GNU Radio issue tracker may suffice, and in this case, a GREP may be
  rejected and redirected to the issue tracker.
- Changes to the GNU Radio organization itself.
- Informational GREPs. This can describe a situation or issue, but does not
  propose anything new or different. If an informational GREP is better
  suited as an enhancement to GNU Radio's documentation, a GREP may be
  rejected and redirected to the manual or other components of GNU Radio.

### GREP Editors

The GREP editors are individuals responsible for managing the administrative and
editorial aspects of the GREP workflow (e.g. assigning GREP numbers and changing
their status).

The list of GREP editors always include the board members, and members of the
General Assembly.
The General Assembly may choose additional individuals to be a editors.

For any decision that pertains to the technical side of GNU Radio, the
technical lead of GNU Radio has the final say. The technical side
includes any changes to the codebase.
For any decision that pertains to the GNU Radio organization or any other
non-technical issue, the GNU Radio organization lead has the final say.

### Life cycle of a GREP

Before anything is submitted, it is highly recommended to discuss ideas for
future GREPs with other GNU Radio developers, preferably on the mailing list.

A GREP comes to life after it was submitted (see the following section). At
first, a GREP is in "Draft" status (which is also declared at the beginning of
a file). Now, the draft is discussed and changes may be proposed. See the
following sections for details on the GREP submission and change processes.

The purpose of the "Draft" status is to determine if a GREP is suitable for
actual consideration within the project. The editors can either decide that
this is the case, which will change the status to "Accepted", or not, which
will make the status "Rejected". The author can also decide that the GREP is
not suitable for further consideration, and set the status to "Withdrawn".
Rejected GREPs still stay part of the GREP repository.

When "Accepted", the next goal is to flesh out all of the open questions and
details, such that the proposal can be flagged as "Final". This status means
that whatever proposal is described in the GREP is meant to be implemented.

A special status, "Active", is meant for GREPs that are never meant to be
completed.

A GREP can be in any of the following statuses:

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
   |        +-|--> REJECTED
   |          |
   |          +--> WITHDRAWN
   v
DEFERRED
```


### GREP Submission Process

In order to submit a GREP, the following process shall be followed:

1. The author of the GREP needs her own version of the GREP repository.
   If necessary, the repository needs to be cloned first.
2. A new GREP is added by copying the template (`grep-9999-template.md`) and
   creating a new GREP file. The filename must follow the following naming
   convention: `grep-9999-keyword.md`. `9999` will be converted into a
   four-digit decimal number (every GREP has a unique number, counting upward)
   by an editor.
   The keyword can be any suitable word (or even multiple words) that
   will help identify the content, but should be kept short to keep the
   filenames manageable.
3. The author fills out the template as explained.
4. When complete, the author submits a pull request against the GREP
   repository, adding the new GREP file.
5. While the pull request is open, the editors, the author, and anyone else
   interested in the GREP cooperate to fix formal, orthographical, and
   stylistic shortcomings, as well as open issues with the actual content.
6. If suitable, and after all major discussion items are resolved, the GREP may
   be merged into the main GREP repository by a GREP editor. The following
   requirements apply before a GREP can be merged:
   - The four-digit number is the next available one
   - The template was filled out appropriately
   - It is a single commit
   - The GREP is actually relevant to GNU Radio, either technically, or
     pertaining to the organization itself.
   Whether or not these requirements are met is decided by any GREP editor.
   If they are met, the pull request is accepted and the GREP is merged into
   the repository. At this point, the GREP is flagged as 'Draft'.
7. A GREP may be rejected for any reason, but there are two main causes
   for rejection: Formal reasons (in which case the author is asked to fix
   those and resubmit), and when GREPs are deemed unfit or unsuitable (in
   which case they will not be considered for resubmission).
   Note: The idea is that there is a very low bar of adding GREPs into the
   repository.
8. Otherwise, the editors will squash the commit and assign a GREP number.

Once a GREP is in the repository, it is open for discussion.

### Proposing changes

Until a GREP is finalized, changes may be proposed. Major changes are proposed
in form of pull requests on top of the original GREP. The issue tracker
associated with this repository may also be used, as well as any other features
provided (such as inline commenting, etc.).

### The Champion

For every GREP, there is one distinct owner, in the following referred to as
the "champion". It is the champion's responsibility to drive the GREP to
completion. GREPs may be transferred to another champion. The champion is not
necessarily the one doing the actual development.

### GREP template

When authoring a GREP, the current version of the template must be used
(file `grep-template.md` in the GREP repository).
The template may change over time, and anyone can propose changes to the
template by either submitting an issue or bug report, or by submitting a
pull request. Changes to the template are reviewed and accepted or rejected
by the editors in the same was as GREP pull requests.

When the template is updated, GREPs that were already submitted do not need
to update to the new version of the template, although GREPs currently under
discussion ("Draft" status) are encouraged to use the new template.
GREPs that are in closed status (either "Accepted" or "Rejected") should not
change their corresponding GREP files when the template is updated.

The file format is also defined by the template. The template is in Markdown
format, with UTF-8 encoded text.

