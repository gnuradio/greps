# GREP 0025 -- Move GRC Out of Tree

- Original Author: Josh Morman <jmorman@gnuradio.org>
- Champion: Josh Morman <jmorman@gnuradio.org>
- Status: Draft 

History:
- 22-Nov-2021: Initial Draft
- 24-Dec-2021: Made active
## Abstract

GNU Radio Companion has proved itself to be the most convenient and powerful access mechanism
to the GNU Radio framework, but many have eyed using the tool beyond just GNU Radio, including:

- GNU Radio 4.0 (newsched) flowgraphs
- FPGA toolchain workflows
- Modeling tool for non-SDR components of radio system

This motivates the idea of "detaching" GRC from the GNU Radio codebase and setting up the workflows
in a modular fashion, so that different workflows can be accomodated without upstreaming to the GNU 
Radio codebase

## Copyright / License

CC-BY-ND

## Motivation

GRC has much potential beyond just rendering python and c++ flowgraphs for GNU Radio.  The application
of connecting blocks together and based on a set of blocks, connections, parameters and then rendering
some programmatic output is general enough where it doesn't need to be tied directly to GNU Radio.  In 
addition, more features can be added that benefit GNU Radio that are difficult to maintain in-tree.

Take for instance gr-bokehgui.  In order to render flowgraphs for bokehgui, it is necessary to have 
the hooks built into the templates that will generate the python file.  If bokehgui was able to install
a "workflow" somewhere that GRC would read, then the parts of the rendering process could be abstracted
out of tree and GRC wouldn't have to know what a bokeh is.

In terms of modularity, GRC already succeeds at having blocks defined modularly which is the basis for
our out of tree (OOT) modules.  Apart from blocks, the missing piece is defining how the graph gets
rendered to a script, or c++ application, or potentially something else. 

## Description

### Workflows

The workflow should define a subset of things that are currently in the `Options` block, which is 
overloaded to handle all possible combinations.  

The workflow can be defined as a `.workflow.yml` file that specifies the parameters that are part of the workflow.

In our current usage of GRC, the following workflows could be proposed:
- gnuradio-python
- gnuradio-python-qtgui
- gnuradio-cpp
- gnuradio-cpp-qtgui
- gnuradio-python-bokehgui

For example, the `gnuradio-python.workflow.yml` file would specify the minimum amount of parameters to render a 
python flowgraph without qtgui.  This might look like:

```yaml
id: gnuradio-python
label: GNU Radio 3.x Python Workflow
flags: ['python']

parameters:
-   id: title
    label: Title
    dtype: string
    hide: ${ ('none' if title else 'part') }
-   id: author
    label: Author
    dtype: string
    hide: ${ ('none' if author else 'part') }
-   id: copyright
    label: Copyright
    dtype: string
    hide: ${ ('none' if copyright else 'part') }
-   id: description
    label: Description
    dtype: string
    hide: ${ ('none' if description else 'part') }
-   id: run_options
    label: Run Options
    dtype: enum
    default: prompt
    options: [run, prompt]
    option_labels: [Run to Completion, Prompt for Exit]
-   id: run
    label: Run
    dtype: bool
    default: 'True'
    options: ['True', 'False']
    option_labels: [Autostart, 'Off']
-   id: realtime_scheduling
    label: Realtime Scheduling
    dtype: enum
    options: ['', '1']
    option_labels: ['Off', 'On']
    hide: ${ ('all' if generate_options.startswith('hb') else ('none' if realtime_scheduling
        else 'part')) }
-   id: catch_exceptions
    label: Catch Block Exceptions
    category: Advanced
    dtype: enum
    options: ['False', 'True']
    option_labels: ['Off', 'On']
    default: 'True'
    hide: part
-   id: run_command
    label: Run Command
    category: Advanced
    dtype: string
    default: '{python} -u {filename}'


templates:
    imports: |-
        from gnuradio import gr
        from gnuradio.filter import firdes
        from gnuradio.fft import window
        import sys
        import signal
        from argparse import ArgumentParser
        from gnuradio.eng_arg import eng_float, intx
        from gnuradio import eng_notation
    callbacks:
    - 'if ${run}: self.start()

        else: self.stop(); self.wait()'

documentation: |-
    Document the workflow

file_format: 1
```

Which is just the options block with everything qtgui, bokeh, c++, hier block stripped out of it.

The workflow yaml should be installed into a workflows/workflow-name directory along with whatever 
mako templates are required to render the output.  The rendering commands should be specified
in the workflow yaml file.  There is no reason jinja2 could not be used as well.

### Selecting the workflow

One idea is that the Options block would have a dropdown that is dynamically populated with 
all the workflow files present at launch.  When a particular workflow is selected, the rest
of the options block would then be populated with the parameters from the workflow yaml file

There are probably other methods that would make for a better user experience, and this
should all be evaluated from a UX point of view.

### Pulling out of tree

With GRC more detached from GNU Radio itself, it should pull up a level and live at the same level as 
GNU Radio

#### Naming

Perhaps if it finds more application outside of GNU Radio, should be renamed to something else, such
as `Graphical Companion` or something more creative.
