# GREP [0024] -- OOTs at same level as modules

[Note: All parts in square brackets are meant to be replaced as part of
authoring a new GREP]

- Original Author: Josh Morman <jmorman@gnuradio.org>
- Status: Draft 

History:
- 09-Nov-2021: Initial Draft

## Abstract

Based off of [Issue #3006](https://github.com/gnuradio/gnuradio/issues/3006)

New OOTs should taken on a structure that more closely resembles the in-tree 
modules.  This allows them to be more coupled with gnuradio, rather than living 
in their own non-GR associated directories and python modules.  We consider 3
aspects to improving the OOT structure

1) Python module under gnuradio
2) C++ header under gnuradio
3) CMake naming involving gnuradio

## Copyright / License

CC-BY-ND

## Motivation

When creating a GNU Radio OOT module using `gr-modtool`, this creates a python 
module and a set of headers/libs which in naming bear no association with GNU Radio.

If a module is a GNU Radio wrapping of some other well known module, there can
be naming conflicts, or at the least confusion.  Take for example cuda: If I want
to use gr-modtool and create a new module gr-cuda - now I have in my prefix

```python
import cuda
```
and
```c++
#include <cuda/my_cuda_block.h>
```

Both of these use the broad naming `cuda` for something that is intended to be 
GNU Radio's wrapping of some cuda functionality - a broad overreach

Another motivation is simply for commonality between the in-tree modules and OOTs
and encourage OOT developers to easily deploy robust CMake module files that 
name their modules similar to other GNU Radio components.  For example instead of
`find_package(cuda)` - which is problematic, a user that is integrating elements
of an OOT into their OOT should call `find_package(gnuradio-cuda)` or even possibly:
`find_package(gnuradio COMPONENTS cuda)` as a stretch


## Description

### Required Changes
1) Python module structure
2) Header structure
3) CMake structure

Refer to example OOT in these examples as howto

#### Python module structure
- Python source goes from `gr-howto/python/` to `gr-howto/python/howto/`
- Likewise for bindings directory
- Python module installation path goes from `PYTHON_DIR/howto` to `PYTHON_DIR/gnuradio/howto`

The net result of this is `from gnuradio import howto`

##### Prefix Considerations
It might be the case that OOT modules and GNU Radio are installed to different prefixes, but are
in the same PYTHONPATH.  Using `pkgutils` should be able to resolve the distributed nature of 
the package - for example, in the `__init__.py` of the top level GNU Radio
```python
from pkgutil import extend_path
__path__ = extend_path(__path__, __name__)
```

#### C++ header structure
- Source include path goes from `gr-howto/include/howto` to `gr-howto/include/gnuradio/howto`
- Header installation path goes from `PREFIX/include/howto` to `PREFIX/include/gnuradio/howto`

The net result of this is `#include <gnuradio/howto/myblock.h>`

##### Prefix Considerations
When e.g. Debian packages are installed, GNU Radio gets installed to `/usr/`, but OOT modules
installed from source get installed to `/usr/local`.  This requires that another OOT that references
the installed OOT is able to use both `#include <gnuradio/anotheroot/header.h>` and 
`#include <gnuradio/filter/fir_filter.h>`, even though gnuradio in this case refers to both locations


#### CMake Structure

Changes to `howtoConfig.cmake` will be necessary to get the proper calling structure
in consuming OOTs.

The net result of this should be (when using elements of an OOT in another OOT):

`find_package(gnuradio-howto)` or

`find_package(gnuradio COMPONENTS howto)`

and then 
`target_link_libraries(my-other-oot gnuradio::howto)`

### Compatibility
Currently, gr-modtool on 3.9/3.10/master is only compatible with modules created 
with the modtool from 3.9/3.10/master.  We don't need to consider backwards compatibility
of modtool to access 3.7 or 3.8 compatible OOT modules

If a module was created with 3.9 modtool, and thus has the global level module structure
modtool should still be able to do add/rm/bind/rename, etc. just as it did with the 3.9 
modtool.

A module created with the 3.10 modtool should have the new structure, but this will not
be backwards compatible with 3.9 modtool (with careful backporting, could be made to)