# GREP [00XX] -- [Relicense VOLK]

- Original Author: [Johannes Demel <demel@uni-bremen.de>]
- Champion: [Johannes Demel <demel@uni-bremen.de>]
- Status: Draft

History:
- 18-Mar-2021: Initial Draft

## Abstract

We have several community requests to relicense VOLK under a more permissive license.
The proposed new license is [LGPLv3+](https://www.gnu.org/licenses/lgpl-3.0.html).
Historically, VOLK was part of GNU Radio. Thus, it made total sense to use GPLv3+ just like GNU Radio.
Since then, VOLK became its own project with its own repository and leadership.
While it is still a core dependency of GNU Radio and considers GNU Radio as its main user, others may want to use it too.
Especially, long term VOLK contributors expressed their desire to be able to use VOLK in a broader set of projects.

### State of the GREP
This GREP does currently serve as a platform to collect all the necessary information and ideas to relicense VOLK.
Everyone is invited to contribute to this GREP with ideas, pull requests, comments etc.

## Copyright / License

This GREP is licensed as CC-BY-ND.
Copyright 2021 Johannes Demel and any co-author listed above.


## Motivation

We recognize the desire of our long term contributors to be able to use VOLK with their contributions in their projects.
This may explicitly include proprietary projects.
We want to enable all contributors to be able to use VOLK wherever they want.
At the same time, we want to make sure that improvements to VOLK itself are easily obtained by everyone, i.e. strike a balance between permissiveness and strict copyleft.
Previously, several attemps were made to relicense VOLK without success.
With a formal process and a clear vision laid out, we hope to be able to get everyone on board.

Since VOLK is a library it should choose a fitting license.
If we see greater adoption of VOLK in more project, we hope to also see more contributions.
May it be bug fixes, new kernels or even contributions to core features.

## Description

In order to facilitate this relicensing effort, we need to got through several steps.

### Notification
We consider it to be very important to let every contributor know that the wish to relicense by some contributors exists.
A first step would be to notify every contributor of this GREP.

### Another license discussion
Currently, the proposed license is LGPLv3+.
Though, other licenses, possibly more permissive are available too.
This includes [3-Clause BSD](https://opensource.org/licenses/BSD-3-Clause), [MIT](https://opensource.org/licenses/MIT) and [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0).
Prelimenary discussion on the subject matter revealed that Apache-2.0 is out of favor by some.
We assume that LGPLv3+ strikes a balance between permissiveness and obligation to provide modified source code.

### All contributors need to sign off for the new license

Before we can adopt a new license, we need approval by every contributor to basically resubmit their contributions under the new license.
[There is more to this discussion and how this evolved but others are more qualified to add to that discussion]

### The process
We need to figure out how we collect all the contributor approvals.
This might be a file within the VOLK repo where everyone adds a signed-off commit to approve the new license.
Currently, we have around 65 contributors to VOLK.

### The meantime
It might take time to gain approval by every contributor.
We want to be able to continue VOLK development while we are in this license transition period.
We need to work out a clear path how to deal with contributions in the mean time.
This will probably prior include relicensing approval by a majority of contributors.
Then, we will probably state that new contributions will only be accepted under the terms of the new license.
The details need to be worked out.

### The aftermath
Assume most contributors approved to relicense VOLK but some did not.
How do we deal with that situation?
A contributor who does not approve the new license, cannot be forced or pushed to do so.
Either, we decide to give up on this relicensing effort or, we are required to remove all contributions by that contributor.
Though, we hope to avoid this situation.
Yet, we need to define a final date by which we will end the transition period.
Otherwise, we risk to stay in relicensing limbo.


### The result
VOLK is available under a new license that better meets the needs of its contributors.
