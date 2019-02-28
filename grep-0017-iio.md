# GREP 0017 -- Add support for IIO devices

- Original Author: Travis Collins <travis.collins@analog.com>
- Champion: Travis Collins <travis.collins@analog.com>
- Status: Draft

History:
- 22-Feb-2019: Initial Draft

## Abstract

GNU Radio is an invaluable tool for data processing, especially for data sources that can provide large amounts of real-time data like sounds devices and software-defined radios. Such support today is provided by in-tree modules like gr-audio, gr-uhd, gr-comedi, and gr-video-sdl. However, a large category of sensors/DAC/ADC among others supported by the Industrial Input/Output Framework of the linux kernel is not supported in-tree today.

The goal of this GREP is to merge the existing out-of-tree (OOT) module gr-iio into GNU Radio mainline.

## Copyright / License

This GREP is licensed as CC-BY-ND.
Copyright 2019 Analog Devices, Inc.

## Motivation

The main purpose of having an IIO module in-tree would allow wider access to users wanting to access IIO devices, especially novice users, and force compatability of gr-iio with GNU Radio by getting gr-iio into mainline CI. Analog Devices, Inc will provide maintenance resources to fix future bugs and add features as necessary.

The underlying library libIIO is crossplatform and available today in many distrubtion package management systems. The current OOT builds across [Linux](https://wiki.analog.com/resources/tools-software/linux-software/gnuradio) including ARM, [macOS](https://github.com/macports/macports-ports/blob/master/science/gr-iio/Portfile), and [Windows](https://github.com/tfcollins/GNURadio_Windows_Build_Scripts/releases/tag/1.5.0).

## Description

Currently gr-iio has two categories of blocks:
 - Generic IIO:
    - IIO Attribute Sink/Source
    - IIO Device Sink/Source
 - Device Specific:
    - PlutoSDR Sink/Source
    - FMComms2/3/4 Sink/Source
    - FMComms5 Sink/Source
    
The generic IIO blocks provide access to stream data using scan elements, as well as direct interaction with all attribute types. Therefore, if a device specific block does not exist users can interact with it simply by filling out the necessary IIO device information. These blocks have already been used by the community to interact with IIO sensors and even custom SDR solutions.

The device specific blocks simplify the process of configuring the generic IIO device block, but technically inherent from those generic blocks under the hood. These drastically make it easier for users who know nothing about IIO to interact with specific devices. These device specific blocks have a single additional dependency libad9361 which is used for generating filters, performing phase alignment and other specialized functions for AD936X based devices like PlutoSDR or E310.

To simplify packaging, gr-iio could be changed to conditionally built blocks based on available the availability of libad9361 since it is not available as widely in distributation package management tools.

The core dependencies of libIIO are: `libxml2 libxml2-dev bison flex libcdk5-dev cmake`
libIIO can be conditionally built with a number of backends including: USB, network, and serial. PCIe will be added soon. These dependencies can include: `libaio-dev libusb-1.0-0-dev libserialport-dev libxml2-dev libavahi-client-dev` and for documentation `doxygen graphviz`.

### Integration

To integrate gr-iio into mainline GNU Radio a new branch has been created [here](https://github.com/analogdevicesinc/gnuradio/tree/merge-griio) based off mainline master which is in the 3.8 release transistion. As of posting gr-iio integration is building successfully and has been upgraded to include the new python changes in 3.8.

#### Todo:
 - [ ] Update all IIO examples to support 3.8
  - Existing bug currently preventing this: https://github.com/gnuradio/gnuradio/issues/2286
 - [ ] Update dependency documentation to include gr-iio dependencies
 - [ ] Add documentation to GNU Radio wiki about gr-iio blockset
 - [ ] Update CI to include gr-iio dependencies
