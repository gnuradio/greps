# GREP [XXXX] -- Upstreaming gr-correctiq

- Original Author: Michael Piscopo <ghostop14@gmail.com>
- Champion: Michael Piscopo <ghostop14@gmail.com>
- Status: Draft

History:
- 07-Jan-2020: Initial Draft

## Abstract

The DC spike created by the IQ sampling process has long been an issue with the sampling 
process. Software such as GQRX and SDRSharp have built-in algorithms that can be enabled 
to remove this artifact.  However, GNU Radio has no built-in blocks to address 
this feature.  A second related feature common in receiver software is the ability 
to correct for inverted spectrum.  The gr-correctiq OOT module provides blocks that 
address both of these issues.  Therefore, this GREP is to propose that gr-correctiq 
be merged into GNU Radio core to make these solutions immediately available to users 
without the need to install the OOT modules.

## Copyright / License

[I could use a suggestion here.  I'd like to retain the right to use the code as I 
may need to, but other than that CC-BY-ND sounds fine.]

## Motivation

When I first stared with SDR, the DC spike inherent in my first few projects with 
GNU Radio actually turned me away because I didn't know how to fix it correctly. 
I'm sure many others, experienced and new, run into the same issue with DC spikes 
and inverted spectra.  Having drag-and-drop native blocks to address it would make it 
a trivial problem to address even for novice users.

## Description

There are 4 key modules in gr-correctiq:

1. CorrectIQ - This block behaves like the algorithms in tools such as GQRX and SDRSharp. 
It continuously evaluates the stream (basically an IIR filter) that removes the DC spike.
2. CorrectIQ AutoSync Offset - This block behaves like the core CorrectIQ block, however 
a learning period limits the time impact of the IIR effect.  So it will "train" for n 
seconds then go into a static offset mode.  If the frequency or gain change, it will 
re-train for the specified period since changing either could impact the offset.
3. CorrectIQ Manual Offset - This gives the user the ability to manually specify the I 
and Q static offsets.
4. SwapIQ - Literally as it sounds.  Swaps the I<->Q values.  While inverting the real 
or inverting the complex would invert the spectrum, a true swap seemed the most preserving 
of the original data.

### Notes

1. There are no other dependencies other than GNU Radio to build these blocks.
2. Currently they're in a CORRECTIQ group.  If integrated, they should be moved into an 
appropriate standard group as well.



