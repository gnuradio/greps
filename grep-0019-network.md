# GREP 19 -- Upstreaming gr-grnets TCP and UDP Source/Sink Blocks

- Original Author: Michael Piscopo <ghostop14@gmail.com>
- Champion: Michael Piscopo <ghostop14@gmail.com>
- Status: Draft

History:
- 07-Jan-2020: Initial Draft
- 05-Mar-2020: Updated by Martin, locked in number

## Abstract

While ZMQ provides a very efficient way to move data when the developer owns both sides 
of the link, legacy systems and systems outside a developer's control still employ TCP 
and UDP data streams.  The gr-grnet TCP and UDP blocks have been completely refactored 
over the past year and written entirely in boost for the best performance. UDP source and 
sink blocks support raw or additional header formats including 64-bit sequence numbers 
that can detect dropped packets, and other formats including those used at the Hat Creek 
Radio Observatory.

The idea is to create a new in-tree component (gr-network) which imports the gr-grnet
blocks and makes them available in GNU Radio. Eventually, they shall replace the gr-blocks
network blocks.

## Copyright / License

This work is licensed under a Creative Commons Attribution-ShareAlike 4.0
International License.

## Motivation

Motivation behind these blocks started when the existing GNU Radio blocks had been marked 
as deprecated.  I work with various solutions where the up or down-stream solutions are 
outside my control and need either a UDP or TCP connection.  So in writing the blocks, 
the goal was to have the best reliable performance possible.

## Description

There are a number of blocks in gr-grnet, however the core blocks I suggest be folded into 
GNU Radio are:

1. TCP Sink
2. UDP Source
3. UDP Sink

In testing, the TCP source relying on boost dropped packets.  This wasn't noticeable until 
attempts to pump compressed blocks through it.  As a result, the 4th block (TCP Source) is 
recommended to maintain the existing TCP Source block for the most reliable performance.

The UDP blocks have been tested at full-speed on the Allen Telescope Array data streams with 
very good performance.  The blocks also support jumbo frames and a variety of header formats
(or straight raw data).

The existing blocks in gr-blocks that serve the same purpose are slightly different, and
thus shall stay in gr-blocks. They will be marked as deprecated, and will be removed in
future versions of GNU Radio.
