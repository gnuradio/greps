# GREP [00XX] -- [Re-license VOLK under LGPL]

- Original Author: [Johannes Demel <demel@uni-bremen.de>]
- Champion: [Johannes Demel <demel@uni-bremen.de>]
- Status: Draft

History:
- 18-Mar-2021: Initial Draft

## Abstract

We want to remove usage entry barriers from VOLK.
As a result, we expect greater adoption and a growing user and contributor base of VOLK.
This move helps to spread the value of Free and Open Source Software in the SIMD community, wich so far is dominated by non-FOSS software.
The proposed new license is [LGPLv3+](https://www.gnu.org/licenses/lgpl-3.0.html).
Historically, VOLK was part of GNU Radio. Thus, it made total sense to use GPLv3+ just like GNU Radio.
Since then, VOLK became its own project with its own repository and leadership.
While it is still a core dependency of GNU Radio and considers GNU Radio as its main user, others may want to use it too.
Especially, long term VOLK contributors expressed their desire to be able to use VOLK in a broader set of projects.

### State of the GREP

This GREP currently serves as a platform to collect all the necessary information and ideas to relicense VOLK.
Everyone is invited to contribute to this GREP with ideas, pull requests, comments etc.

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

In order to facilitate this relicensing effort, we need to got through several steps.


### Notification

We consider it to be very important to let every contributor know that the wish to relicense by some contributors exists.
A first step would be to notify every contributor of this GREP.


### The proposed license

We propose [LGPLv3+](https://www.gnu.org/licenses/lgpl-3.0.html) for the new license.
LGPLv3+ strikes a balance between permissiveness and obligation to provide modified source code.
All modifications to VOLK should be made available under the same terms under which everyone can use VOLK.
This should not require users to share code that uses VOLK unaltered.


### All contributors need to re-submit their code under the new license

Whether one submitted code under a CLA or with a DCO, every contributor must re-submit their code under the new license.
A contributor may submit their code to any project, including VOLK under a new license, even if that code was submitted under a CLA.


### The process

Currently, we have around 65 contributors to VOLK.
In order to relicense VOLK, every contributor must re-submit their code under the new license.
Once all contributors re-licensed their code, we can add a new commit to VOLK that updates the license as well as all the license headers.
We consider it to be favorable to do this transition in-place, i.e. in the same repository, to not add any confusion.
This approach should keep things simple.
Most importantly, we do not introduce any new places to search for VOLK which would introduce confusion.

#### how to
We need to figure out how we collect all the contributors re-licensed code.
This might be a file within the VOLK repo where everyone adds a signed-off commit that states they re-submit their code under the new license.
Alternatively, we may collect all the contributors re-license statements in another place.
Though, this might raise questions about long term documentation of this change.


### The meantime

It will probably take time until every contributor re-licenses their code.
We want to be able to continue VOLK development while we are in this license transition period.
A possible approach may be:

- ask for informal support and start the formal process as soon as a majority of contributors confirmed their support.
- Decide on a start and end date for this process
    - This period should be between two releases.
- Collect all the re-license agreements
- Add a commit that reflects the license change in every necessary place
- Do a first release under the new license.

In the meantime, we use the following approach.
First, we add a statement to the VOLK repository.
```
Starting at {DATE}, we can only accept contributions under the new license.
```
At that point, we probably notify every open PR about that new requirement.
Then, we require
```
Copyright {YEAR} {CONTRIBUTOR}

This file is part of VOLK

SPDX-License-Identifier: LGPL-3.0-or-later
```
to be added to every modified file.
cf. [SPDX identifier for LGPLv3+](https://spdx.org/licenses/LGPL-3.0-or-later.html).
This entails a careful rework of the current license statements in all files.


### The aftermath

Assume most contributors re-licensed their VOLK contributions but some did not.
How do we deal with that situation?

- We could extend the transition period and hope contributors where just to busy to re-license their work yet.
- Decide to remove contributions. This might imply that we lose features and change our API which either requires someone else to step up and re-implement this feature or definitely do a new major release that indicates API breakage.
- Give up on the relicensing effort and revert to GPLv3+.

A contributor who does not want or cannot re-license their work, cannot be forced or pushed to do so.

We really hope to avoid this situation but need to be aware that we might face it anyways.
Otherwise, we risk to stay in relicensing limbo.

#### The first relicensed VOLK version

We need to figure out the prefered way to announce the completed relicensing effort.
We could consider the new license an API breaking change and release a new major version.
Alternatively, we could continue the current major release series and just have another release.


### The result
VOLK is available under a new license that better meets the needs of its community.
