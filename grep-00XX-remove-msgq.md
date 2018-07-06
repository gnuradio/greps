# GREP [XXXX] -- Remove legacy msgq API

- Original Author: Martin Braun <martin.braun@ettus.com>
- Champion: Martin Braun <martin.braun@ettus.com>
- Status: Draft

History:
- 29-Jun-2018: Initial Draft

## Abstract

This GREP proposes to remove the msgq API from GNU Radio, and propose users to
use either the existing Python blocks, or message passing.

## Copyright / License

This GREP is licensed as CC-BY-ND.
Copyright 2018 Martin Braun and any co-author listed above.

## Motivation

For a long while, GNU Radio has had the msgq API, a simple way to convert
streams to messages. In the past, this API was used to move data from the C++
into the Python domain, because the msgq API calls could be easily SWIG'd, but
the block connections could not.

For a few years now, msgq has been superseded by Python blocks and message
passing. The former solves the issue moving data from the C++ to the Python
domain, and the latter implements a more feature-complete message passing API.

The msgq API is thus obsolete, and can be easily replaced with those techniques.
For the upcoming 3.8 release, it is already clear that the msgq API will require
some work to keep running, and the question becomes on whether or not this is a
good time to simply remove it.

## Description

### What shall be removed?

The following files shall be removed:

- Everything in `gnuradio/messages` and `gnuradio-runtime/lib/messages`, as well
  as all related SWIG bindings
- `msg_queue.{cc,h}` and related SWIG bindings and unit tests
- gr-blocks: `bin_statistics` block.
  - Alternative: Replace this block with one that uses actual message passing
- gr-digital: `framer_sink` and `packet_sink` blocks
  - Alternative: Replace these blocks with ones that use actual message passing
- gr-digital: `pkt.py` and various Python examples
- gr-digital: Legacy OFDM blocks
- gr-uhd: Replace async message source block with one that uses actual message
  passing


### What are the consequences?

Anyone using any of the blocks mentioned above will be affected by this change.
When blocks can be migrated to the new-style message passing interface, we can
provide examples how they shall be used.



