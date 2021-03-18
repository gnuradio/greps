# GREP [0023] -- [Re-license VOLK under LGPL]

- Original Author: [Johannes Demel <demel@uni-bremen.de>]
- Champion: [Johannes Demel <demel@uni-bremen.de>]
- Status: Draft

History:

- 18-Mar-2021: Initial Draft
- 28-Aug-2021: Final Draft
- 29-Sep-2021: Final

## Abstract

We want to remove usage entry barriers from VOLK.
As a result, we expect greater adoption and a growing user and contributor base of VOLK.
This move helps to spread the value of Free and Open Source Software in the SIMD community, which so far is dominated by non-FOSS software.
The proposed new license is [LGPLv3+](https://www.gnu.org/licenses/lgpl-3.0.html).
Historically, VOLK was part of GNU Radio. Thus, it made total sense to use GPLv3+ just like GNU Radio.
Since then, VOLK became its own project with its own repository and leadership.
While it is still a core dependency of GNU Radio and considers GNU Radio as its main user, others may want to use it too.
Especially, long term VOLK contributors expressed their desire to be able to use VOLK in a broader set of projects.

### State of the GREP

This GREP received a lot of feedback and incorporates this feedback.
Now, we consider it ready for submission.


## Copyright / License

This GREP is licensed as CC-BY-ND.
Copyright 2021 Johannes Demel and any co-author listed above.


## Motivation

We recognize the desire of our long term contributors to be able to use VOLK with their contributions in their projects.
This may explicitly include proprietary projects.
We want to enable all contributors to be able to use VOLK wherever they want.
At the same time, we want to make sure that improvements to VOLK itself are easily obtained by everyone, i.e. strike a balance between permissiveness and strict copyleft.
With a formal process and a clear vision laid out, we hope to be able to get everyone on board.

Since VOLK is a library it should choose a fitting license.
If we see greater adoption of VOLK in more projects, we are confident that we will receive more contributions.
May it be bug fixes, new kernels or even contributions to core features.


## Description

In order to facilitate this relicensing effort, we need to go through several steps.


### Notification

We consider it to be very important to let every contributor know that the wish to relicense by some contributors exists.
Based on the information we can gather through git commit names and email addresses as well as GitHub accounts, we notify every contributor of this GREP and ask them to resubmit their contributions.


### The new license

We propose [LGPLv3+](https://www.gnu.org/licenses/lgpl-3.0.html) for the new license.
LGPLv3+ strikes a balance between permissiveness and obligation to provide modified source code.
All modifications to VOLK must be made available under the same terms under which everyone can use VOLK.
This should not require users to share code that uses VOLK unaltered.


### All contributors need to re-submit their code under the new license

Whether one submitted code under a CLA or with a DCO, every contributor must re-submit their code under the new license.
A contributor may submit their code to any project, including VOLK under a new license, even if that code was submitted under a CLA.

As noted below and here for completion: In reality contributors add their name to a list of those who approve of moving to the new license, and by adding themself to the list, a contributor "resubmits" their code to VOLK under the new license. There is no formal or technical (e.g., a PR) resubmission of code.

### The process

Currently, we have around 65 contributors to VOLK.
In order to relicense VOLK, every contributor must re-submit their code under the new license.
During the transition period, we want to be able to accept new contributions.

1. We notify every contributor that we will relicense VOLK.
2. New contributions are required to be submitted under the new license.
3. Contributors are kindly asked to resubmit their contributions.
4. After a vast majority, preferably all, contributors have resubmitted their contributions, we have a new major release and continue VOLK development under the new license.

The new major release updates all license headers to state the new license.
The transition process is executed in-place, i.e. in the current VOLK repository, to not add any confusion.
In case we do not receive resubmissions from all contributors, we would be forced to remove contributions that are not resubmitted before the major release under LGPLv3+.

#### How to resubmit contributions
We decided to adopt the following process to resubmit contributions.

1. We collect all resubmissions in a file inside the VOLK repository.
2. All contributors are kindly asked to add themselves to this list.
3. By adding themself to the list, a contributor resubmits their code to VOLK under the new license.
4. Contributors have several options to add themselves to the list
    - Open a PR against the VOLK repository and add themself to the list.
    - Write an email to the current VOLK maintainers that clearly states the required author information.
    - Add a line to a public VOLK GitHub issue that clearly states their wish to resubmit their code.
    - Sign a paper form and make it available to us. This is generally cumbersome, but can be a useful method at events where people are available in-person.


##### Suggested wording wording
I, {contributor name}, hereby resubmit all my contributions to the VOLK project and repository under the terms of the LGPLv3+. My GitHub handle is {github handle}, my email addresses used for contributions are: {email address}, ... .

The maintainers add the contributor to the list together with their statement in the git commit message.
We add this statement in a git commit message for long term documentation purposes.

##### Required information and purpose

We ask for the following information to be able to identify all contributions of individual contributors.

- Date of resubmission
- Name of contributor
- GitHub handle (if applicable) to ease identification of individual contributions.
- git email addresses used for contributions.

These data points are used in git commits as well.


### Transition period

It will take time until every contributor re-licenses their code.
We want to be able to continue VOLK development while we are in this license transition period.

The latest VOLK release is 2.5.
We start to collect resubmissions.
After we collected all resubmissions, we will have a new major release VOLK 3.0.
We consider the license change a breaking change.
Further API updates may manifest themselves during the transition period.

In the meantime, we use the following approach.
First, we add a statement to the VOLK repository.
```
We are in the process to relicense VOLK under [LGPLv3+](https://spdx.org/licenses/LGPL-3.0-or-later.html).
We can only accept contributions under the new license.
```
This is accompanied by the addition of a file that collects the information of all contributors that resubmitted their code.
Further, we add the new license and corresponding information.
We notify every open PR about that new requirement.

At the end of the transition period, we change license file headers in all files to
```
Copyright {YEAR} {CONTRIBUTOR}

This file is part of VOLK

SPDX-License-Identifier: LGPL-3.0-or-later
```
cf. [SPDX identifier for LGPLv3+](https://spdx.org/licenses/LGPL-3.0-or-later.html).



### The aftermath

Assume all contributors resubmitted their code, then we continue with a new major release.

Assume some major contributors disagree with the license change, then we need to abandon this effort. However, we feel confident that this is a very unlikely outcome.

Assume most contributors re-submitted their VOLK contributions but some did not.
We kindly ask again for resubmissions and extend the transition period.
The whole effort will start without a definitive end date but we will set an end date as soon as we are able to estimate how long it might take for everyone to resubmit their code.
After we set an end date for the transition period and this period expired, we must remove all contributions that were not relicensed.
We can re-add them if they are resubmitted later.

A contributor who does not want or cannot re-license their work, cannot be forced or pushed to do so.
We really hope to avoid this situation but need to be aware that we might face it anyways.
We must remove these contributions before the first release under the new license.
Otherwise, we risk to stay in relicensing limbo.

#### The first relicensed VOLK version

The license change is considered to be an API breaking change.
Thus, we will have a new major release VOLK v3.0 and continue development on that branch.


### The result
VOLK is available under a new license, LGPLv3+, that better meets the needs of its community.
