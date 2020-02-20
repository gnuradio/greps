# GREP 0015 -- Replace SWIG with something else

- Original Author: Martin Braun <martin@gnuradio.org>
- Champion: Martin Braun <martin@gnuradio.org> / Josh Morman <mormjb@gmail.com>
- Status: Draft

History:
- 26-Dec-2018: Initial Draft
- 20-Feb-2020: Update with current design assumptions toward pybind11

## Abstract

**Note: This is a very raw GREP. It's a discussion we've been meaning to have
for a while. This GREP will stay here for as long as this repo will exist, even
if the out come of the discussion is to not replace SWIG.**

SWIG is a huge dependency. We might be able to do better.  The leading candidate is pybind11, which is a header only library to produce Python bindings from c++. Potential gains:

- Faster compile times
- Less memory usage during compile
- One fewer dependency. Our CMake would get a lot saner.
- It's possible that writing special-case wrappers would become easier

There is a major impact of this: We would no longer have the option to wrap
our C++ code into another language. However, we've never used this feature
anyway.


## Copyright / License

This GREP is licensed as CC-BY-ND.
Copyright 2020 Free Software Foundation??

## Motivation

A lot of us have cursed SWIG programming. If we can get rid of it... well,
wouldn't that be great. But maybe we can't. Let's find out.

Since SWIG takes in the entire public header file, many methods and class members that are not necessarily intended to be exposed through the python API gets automatically bound.  Pybind11 requires deliberate binding of methods, and the binding code does not get regenerated on every compile.  

## Description

### Block Binding Code Structure

In each gr module, create a `python/bindings` directory

`python/bindings`
-  `python_bindings.cpp` <-- c++ code to build up the pybind11 module block by block
   -  alternatively this can be broken up into a per-block cpp file
-  `generated/` <-- folder to contain automatically generated binding code with [blockname]_python.hpp
-  `custom/` <-- folder to contain manually generated binding code

Let's take as an example the public header of char_to_float.h in gr-blocks/include/gnuradio/blocks

```c++
class BLOCKS_API char_to_float : virtual public sync_block
{
public:
    typedef std::shared_ptr<char_to_float> sptr;
    static sptr make(size_t vlen = 1, float scale = 1.0);
    virtual float scale() const = 0;
    virtual void set_scale(float scale) = 0;
};
```

In order to bind this, we write/generate `char_to_float_python.hpp`:

```c++
void bind_char_to_float(py::module& m)
{
    using char_to_float    = gr::blocks::char_to_float;

    py::class_<char_to_float,gr::sync_block,
        std::shared_ptr<char_to_float>>(m, "char_to_float")

        .def_static(py::init(&char_to_float::make),
           py::arg("vlen") = 1,
           py::arg("scale") = 1.0
        )

        .def("scale",&char_to_float::scale)
        .def("set_scale",&char_to_float::set_scale,
            py::arg("scale")
        )
        ;
}
```

`python_bindings.cpp` will include the above hpp file, and make the call to `bind_...`:

```c++
#include <pybind11/pybind11.h>
namespace py = pybind11;
...
#include "generated/char_to_float_python.hpp"
...

PYBIND11_MODULE(blocks_python, m)
{
    ...
    bind_char_to_float(m);
    ...
}
```

### Binding Automated Tools

The only way in which pybind11 is a usability improvement over SWIG is if the proper automated tools exist to have a smooth workflow, and generate the bindings and related code automatically.  

#### Workflow

##### GR In-Tree Modules

- run bindtool (tool to generate bindings) on the in-tree module
  - generates _python.hpp, python_bindings.cpp, snippets of CMakeLists.txt
  - output directory is specified so it doesn't smash the code tree
- manually copy/paste and move files into the working codebase

##### OOT Modules

- gr_modtool add
  - creates blank binding templates and appropriate directory structure
- gr_modtool pybind
  - same behavior as gr_modtool makeyaml
  - calls bindtool under the hood
  - parses the header files in the module
  - updates the python/CMakeLists.txt
  - generates block_python.hpp for each block
  - updates python_bindings.cpp
- alternatively, run bindtool and output the bindings to some external/temp dir

#### Assumptions

- We will try to reuse the block header parser tool for parsing header files and then using the parsed information to generate the bindings.  
  - This tool relies on pygccxml, which is rather slow, and adds some extra dependencies [need to evaluate]
  - If speed at this stage is necessary we can revert back to homegrown header parser
- Bindings will **NOT** be generated at compile time, but by calling a separate tool such as gr_modtool similar to generating the yaml for grc blocks
  - If the binding generation is sufficiently sped up, then bindings can be generated at compile time
  - In which case, it is necessary to make sure previously manually edited bindings are not overwritten, hence the above generated/custom directory structure

### Python-only flowgraphs

The current implementation of Python-only flowgraphs relies on the Swig Director functionality to evaluate python calls from the C++ gateway block wrappers.  Pybind11 also includes the ability for c++ to call back into Python, though this performance needs to be evaluated, but could make for a very clean interface from c++ --> python --> c++

A proposed approach for replacing the current block_gateway is the following:

- block_gateway.h is exposed through pybind11 so that python blocks can inherit from block_gateway (same as SWIG)
- block_gateway constructor takes and stores a handle to a python object

```c++
    static sptr make(const py::object& py_handle,
                     const std::string& name,
                     gr::io_signature::sptr in_sig,
                     gr::io_signature::sptr out_sig);
```

- block_gateway only has `general_work()` and is broken up into work vs general_work in the python code.  The `general_work()` function that gets called from the scheduler relies on the python code to do everything:

```c++
    py::object ret = _py_handle.attr("handle_general_work")(noutput_items, ninput_items, input_items, output_items);

    return ret.cast<int>();;
```

- same for forecast:

```c++
    py::object ret_ninput_items_required = _py_handle.attr("handle_forecast")(noutput_items, ninput_items_required.size());
    ninput_items_required =  ret_ninput_items_required.cast<std::vector<int>>();
```

- python blocks inherit from {sync_block, decim_block, interp_block, basic_block} in gateway.py
- gateway.handle_general_work calls the appropriate work() function on the derived block
  
### Issues and Limitations

- though pybind11 supports custom container classes including boost::shared_ptr, there are some issues with proper downcasting, so std::shared_ptr should be used globally
- overloaded functions do not get handled automatically require more verbose function pointer casting in the binding definitions - this can be added to the automated tools


### Options for a replacement:
- Boost.Python with auto-generated files (blocktool could be useful here)
- PyBind11 is similar to Boost.Python. It's header-only and could be shipped
  with GNU Radio instead of making it a dependency. It would still require
  auto-generated files as with Boost.Python.
