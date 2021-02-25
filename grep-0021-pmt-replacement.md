# GREP 0019 -- Restructure GNU Radio Blocks

- Original Author: Josh Morman <mormjb@gmail.com>
- Champion: Josh Morman <mormjb@gmail.com>
- Status: Draft

History:

- 25-Feb-2021: Initial Draft

## Abstract

This GREP proposes replacing the PMT types with classes that support better 
serialization and have a cleaner API

## Copyright / License

CC-BY-ND

## Motivation

The existing PMT API is confusing, doesn't take advantage of OOP concepts, and
the nomenclature for doing simple things can be inconsistent or lacking.  Additionally,
there has been work to show that the serialization/deserialization is inherently slow
and specific to the PMT library.

The leading candidate to natively support serialization of polymorphic types is
[flatbuffers](https://google.github.io/flatbuffers/).  This library provides cross
platform serialization and deserialization and is efficient in storing the serialized
representation in a flat buffer that can be accessed directly

## Description

Create a new in-tree module - `pmtf` - that will live alongside `pmt` during a 
deprecation cycle.  The `pmtf` namespace will have a drastically updated API that
is more similar to `std::` and other libraries used inside the GNU Radio codebase.

`pmtf` will wrap flatbuffer representations of polymorphic objects (underlying
blob can map to a fixed number of enumerated types)

The work to prototype flatbuffer based pmts is being prototyped [here](https://github.com/gnuradio/newsched/tree/master/pmt)

### Proposed API (looking for feedback)

Templated classes derived from `pmt_base` represent the _container_ holding the specified
type:

- `pmt_scalar`
  - represents a single value (e.g. `pmt_scalar<uint8_t>`)
- `pmt_vector`
  - represents a uniform vector (e.g. `pmt_vector<gr_complex>`)
- `pmt_map`
  - roughly replaces `pmt::dict`, but the generality of the anything-to-anything
  dict is hard to serialize with flatbuffers.  `map` seems to be a reasonable simplification
- `pmt_string`
  - a string


Objects (same as in legacy pmts) are generated as a shared pointer to a PMT object, 
since they are meant to be passed around

```
auto pmt_val = pmt_scalar<std::complex<float>>::make(cplx_val);

std::vector<int32_t> int_vec_val{ 5, 9, 23445, 63, -25 };
auto int_pmt_vec = pmt_vector<int32_t>::make(int_vec_val);
```

Maps are created by specifying the key type, but the value is always another PMT:
```
// Create the PMT map
std::map<std::string, pmt_sptr> input_map({
  { "key1", pmt_scalar<std::complex<float>>::make(val1) },
        { "key2", pmt_vector<int32_t>::make(val2) },
  });
auto map_pmt = pmt_map<std::string>::make(input_map);
```

### Additional Dependencies

The flatbuffers library and flatc compiler will need to be included to compile the PMT schema definition into a generated header

Alternatively if we don't expect the schema to change very ofted, this can be done as a
manual step and the generated header included in-tree.  Then, only the flatbuffers headers
will need to be available at compile time

Currently, to get complex values working, the latest version of flatbuffers from the master
branch is required (beyond 1.12 which will eventually become 2.0).  

It doesn't make sense to put this in-tree until flatbuffers 2.0 is released. 