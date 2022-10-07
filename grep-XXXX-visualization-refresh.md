# GREP [XXXX] -- Rework Visualization Widgets

- Original Author: Josh Morman <jmorman@gnuradio.org>
- Champion: Josh Morman <jmorman@gnuradio.org>
- Status: Draft 

History:
- 06-Oct-2022: Initial Draft

## Abstract

The QT widgets have become visually out of date and difficult to maintain for several
reasons, so refreshing the design and code structure would enable more elegant  and
performant GUI displays in GNU Radio.  This should encompass separating the data buffering
logic from the signal processing from the graphical display.  This work should also enable
custom GUI design to take place easily beyond what is able to be done in GRC, as well as
remote rendering of the flowgraph (e.g. GR running headless, display occurring over a network
connection)

## Copyright / License

CC-BY-ND 

## Motivation

Graphical widgets are a large part of the user experience in GNU Radio, and a key to being
able to demonstrate and debug the functionality of a signal processing flowgraph.  For the most
part, the GR QT Widgets have not been updated significantly for quite some time, and visually
look out of date, such as the waterfall plot.

Much of the code is hardcoded to a certain limited set of options, such as the number of ports 
or the fft size.  

The original motivation and design suggestions can be found here:

https://wiki.gnuradio.org/index.php?title=GSoCIdeas#QT_Widgets_Improvements

https://github.com/gnuradio/gnuradio/issues/6029


## Description

### Design Components

1) Universal Data Sink
2) Decoupled Visualization Widgets
3) GRC Decoupled Workflow
4) Separate GUI Designer

### Universal Data Sink

https://github.com/gnuradio/gnuradio/issues/6029#issuecomment-1196338157

Most of the current QT Plotting Widgets have a common step in which they buffer up data
and the most recent N samples is plotted when the buffer is full, up to some configured refresh
rate.  This can easily be separated into a common block that buffers N items, and either:

1) outputs a PMT message at the configured refresh rate
2) is polled whether new data is available

The buffered data could represent any dimensional data.  Some examples

```
sig_src --> data_sink ~~> (time_plot)  (N items of size 1)
sig_src --> fft+window --> log pwr --> data_sink ~~> (freq_plot)  (1 item of size fftlen)
sig_src --> fft+window --> log pwr --> data_sink ~~> (waterfall plot) (nrows of size fftlen)
sig_src --> cyberether --> data_sink ~~> (cyberether plot) (frames of nrows x ncols)
```

Inside the block is simply a circular buffer for each input.  An option for the block could
be whether the read pointer is ignored and always the last N samples are kept, or if the block
creates a larger circular buffer and uses the read pointer to ensure samples are not dropped. 
In this case, if not read quickly enough, would cause other blocks in the flowgraph to potentially
over/underrun.

#### Synchronized inputs of different sizes

The buffers could even be of different sizes and types.  Say I wanted one port to be time series data, and the second
port to be the synchronized constellation.  How to keep the buffers sync'd would be an 
interesting problem to solve.

The block should also have some remote interface so the data can be pulled into a GUI that 
is running on a remote machine.  An API over ZMQ would be a good candidate here, and if 
the interface out of the block is PMT, serialization is already supported.  Not sure
if this should be built into the block or a separate block.

#### Return Values

A PMT map or map of maps makes a strong candidate for the return value of the `data_sink`.  Could
look something like:

```json
{
    "meta": {
        // top level parameters
    },
    "data": {
        "ch0": {
            // channel specific parameters
            "samp_rate": 12345.678,
            "dtype": "rf32",
            "data": //pmt_vector<dtype>,
        },
        "ch1": {
            // same structure as ch0
        }
    }
}

When queried over the ZMQ interface, this would be serialized (`pmtf::map::serialize`) and then deserialized
within the visualization widget

```

### Decoupled Visualization Widgets

With the buffering and signal processing out of the way, the visualization widget can focus on display only.
Signal processing *could* also be incorporated in the visualization widget if the desire is to remove that 
computation from the remote node and perform it at the display terminal, if the DSP is only for visualization
purposes.

Several different GUI frameworks could be supported for plotting and variable control, either in- our out-of-tree.
We would probably want at least one solid set of widgets to include in-tree depending on what is the most maintainable

#### Updated QT Widgets

The QT widgets should be rewritten to have better structure and get the data from the universal data sink.

#### DearImGui / ImPlot

[DearImGui](https://github.com/ocornut/imgui) provides a graphical toolset for immediate mode GUI development.  The 
GUIs generated look very modern and slick.  This is also the direction used by 
[CyberEther](https://github.com/luigifcruz/CyberEther)

#### Bokeh GUI Widgets

These exist out of tree for display of plots inside a web browser (though the GRC rendering lives in-tree).  
Updates to these blocks to work with the universal data sink would be required.

### GRC Decoupled Workflow

As described in [GREP-0025](grep-0025-grc-out-of-tree.md), the desire to have workflows in GRC be modular
would allow a decoupling of the rendering of the GUI application (not GRC itself, but the QT/Bokeh or similar
interface that gets output from the flowgraph rendering) from the GRC itself.  This would allow a completely 
different set of GUI widgets and the associated workflow to live entirely out of tree and not have to update
code within GRC to integrate a new set of GUI widgets

The workflow within GRC can be used to render a python script or C++ application that 

### Separate GUI Designer

GRC provides a quick way to configure the display of widgets, but does not have the capability to do 
comprehensive GUI design.  Can the output of the flowgraph rendering (instantiate blocks, connect, run)
be decoupled from the GUI designer?  Several ideas have been proposed here

#### QT Creator

Standard tool for generating QT GUI designs.

#### DearImGui

There are some interactive GUI designers that could possibly be leveraged

https://github.com/Code-Building/ImGuiBuilder

https://github.com/Raais/ImStudio
