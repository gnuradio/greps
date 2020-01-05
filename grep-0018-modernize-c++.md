# GREP 0018 -- Modernize C++

- Original Author: Thomas Habets <thomas@habets.se>
- Champion: Thomas Habets <thomas@habets.se>
- Status: Draft

History:
- 05-Jan-2020: Initial Draft

## Abstract

As of early 2020 there is lots of pre-C++11 code in GNU Radio, and
some other pattern of non-idiomatic C++.

This GREP lays out how to get from here to there with minimal
breaking.

## Copyright / License

This GREP is licensed as CC-BY-ND. Copyright 2020 Google Inc.

## Motivation

Cleaning up whole patterns of the code 

The legacy patterns this GREP lays out a plan for making the code base:

* more readable: standard components, less code, and explicit const
* safer: less manual memory management, prevent inherently broken copy
* faster: fewer pointer indirections, more information to the compiler

## Description

### More const

Benefits:
* more readable; even in a function reader can see that a name has a
  final value
* avoids accidents
* possibly allows compiler optimizations

Recommendation:
* private members and other `_impl.xx` code should be free to change
  at any time to add const
* adding const to exposed interfaces can be done before a minor
  release

### More constexpr

`constexpr` guarantees compile time evaluation.

Benefits over const:
* no risk of initialization order breakage
* can be used in some places where const can't
* enables compiler optimizations

Recommendation: See "More const", above

### Depointerization

GNU Radio has a lot of member variables that are raw pointers. In many
cases ([example][raw-pointer-to-obj]) they don't have to be pointers
at all, which:
* clarifies for the reader what the lifetime of the object is
* removes pointer dereference at runtime
* possibly enables compiler to make smarter optimizations
* reduces need for manual memory management, and in many cases the
  whole destructor

Other raw pointers do need to remain dynamic since changing them
requires instantiating a new object
([example][raw-pointer-to-std]). They should be changed to `std` smart
pointers, or even more preferably the objects should be movable and be
changed that way. For objects where copies are cheap it's enough to be
able to copy.

Another class of raw pointers is arrays, which sometimes require
volk-specific allocation. Now that `volk::vector` exists these can all
be changed to `std::vector` or `volk::vector`.

Recommendation:
* Allow 3.9 release to break ABI by changing raw and Boost pointers to
  `std` and non-pointers even in public interfaces.

### Delete copy constructor / copy assignment

Where classes are not copyable (e.g. contain a pointer), the copy
constructor and copy assignment operator should be deleted.

```c++
  foo(const foo&) = delete;
  foo& operator(const foo&) = delete;
```

This should be added even if the class has a `std::unique_ptr` member
variable, since it may be changed to `std::shared_ptr` in the future.
(TODO: opinions? how likely is that?)

Recommendation:
* Before minor version release, do this as much as possible
* Where uncopyability is exposed in a released version and can be
  fixed: make the object copyable. It's less efficient, but fixes
  runtime behavior without breaking OOT builds.
* Where uncopyability is inherent in a released version to the exposed
  interface: break OOT builds by deleting copy constructor & copy
  assignment. It's better to break at build time than runtime.

### Boost smart pointers

Boost smart pointers currently provide no extra benefit over C++11
smart pointers. The only thing missing in C++11 (which arrived in
C++14) is `std::make_unique()`. Until C++14 is required for GNU Radio
that gap can be filled by `boost::make_unique()`.

Benefits of switching to `std`:
* readability for a larger population of developers
* reduction of dependency on Boost.
* more likely to we interoperable with third party libraries and OOT
  blocks

Recommendation:
* Merge [PR 2974][PR2974], breaking ABI before 3.9 release.

### Boost locks & threads

The scheduler depends on thread cancellation in a few places, which
depends on `condition_variable`s checking for cancellation at every
[trigger point][trigger-points].

```
$ grep -r 'interrupt()' .
./gr-blocks/lib/socket_pdu_impl.cc:        d_thread.interrupt();
./gr-blocks/lib/message_strobe_impl.cc:    d_thread->interrupt();
./gr-blocks/lib/stream_pdu_base.cc:        d_thread.interrupt();
./gr-blocks/lib/message_strobe_random_impl.cc:    d_thread->interrupt();
./gnuradio-runtime/lib/thread/thread_group.cc:        (*it)->interrupt();
./gnuradio-runtime/lib/logger.cc:        instance.watch_thread->interrupt();
./gnuradio-runtime/lib/controlport/thrift/thrift-codebase-shutdown-patch.diff:+    serverTransport_->interrupt();
./gnuradio-runtime/lib/controlport/thrift/thrift-codebase-shutdown-patch.diff:-    serverTransport_->interrupt();
```

There are 26 `condition_variable`s, and possibly other trigger points
that matter.

The work of changing the scheduler and the method of thread
cancellation is big enough to be out of scope for this GREP, and will
require a separate one.

Recommendation:
* Leave these for a future GREP

### `volk_malloc`

Likely all of these calls can be replaced by `volk::vector`, reducing
manual memory management, improving readability, and reducing chances
for mistakes like copy-paste ` * sizeof(float)` where ` *
sizeof(gr_complex)` was intended.

Recommendation:
* Replace all feasible uses of `volk_malloc` et al with `volk::vector`

### `xxx_cast<>`

`static_cast<>` and friends are safer than C style casts, but more
verbose.

A typical example is:

```c++
[…]
  gr_vector_const_void_star& input_items,
[…]
  float* in = (float*)input_items[0];
```

which should be changed to use a `static_cast<>` in order to not cast
away the constness.

Recommendation:
* Change them where they make sense, but don't have as a goal to
  eliminate C style casts where they seem right

### De-boostify

There's a lot of Boost in GNU Radio. Some have replacements in C++11
(e.g. `boost::to_string`), others don't (e.g. `std::filesystem` is
C++17).

Some uses of a Boost feature may drag others in. E.g. the lock
situation described above. Others are self-contained.

Quick survey inventory of boost:

```
$ egrep -r 'boost::[a-z_A-Z0-9]+' . | sed -nr 's,.*(boost::[a-zA-Z0-9_]+).*,\1,p' | sort | uniq -c | sort -rn
    555 boost::shared_ptr
    198 boost::format
    107 boost::asio
     97 boost::bind
     34 boost::mutex
     32 boost::posix_time
     25 boost::any
     19 boost::thread
     17 boost::system
     16 boost::dynamic_pointer_cast
     13 boost::math
     12 boost::filesystem
      9 boost::uint32_t
      9 boost::this_thread
      9 boost::shared_mutex
      9 boost::condition_variable
      7 boost::lexical_cast
      7 boost::get_system_time
      7 boost::enable_shared_from_this
      7 boost::any_cast
      6 boost::make_unique
      5 boost::noncopyable
      4 boost::system_time
      4 boost::str
      4 boost::recursive_mutex
      4 boost::program_options
      4 boost::crc_optimal
      3 boost::xpressive
      3 boost::weak_ptr
      3 boost::scoped_ptr
      3 boost::make_shared
      3 boost::lockfree
      3 boost::function
      3 boost::dynamic_bitset
      2 boost::to_string
      2 boost::thread_interrupted
      2 boost::shared_array
      2 boost::scoped_array
      2 boost::interprocess
      2 boost::function0
      2 boost::assign
      1 boost::shared_sptr
      1 boost::ptr_map
      1 boost::lock_guard
      1 boost::integer
      1 boost::int64_t
      1 boost::get_deleter
      1 boost::barrier
      1 boost::bad_lexical_cast
      1 boost::bad_any_cast
      1 boost::array
```

Recommendation:
* Switch to `std` whereever and whenever possible, when it's not part
  of the API.
* Don't switch half way for API-visible changes. E.g. if one module
  needs a feature of `boost::x` not yet in `std`, but another is fine
  either way, then don't change either of them. Specifically this
  applies to mutexes, described above.
* API-breaking changes are fine before releasing a new minor version,
  as long as they are switched all the way.

[trigger-points]: https://www.boost.org/doc/libs/1_67_0/doc/html/thread/thread_management.html#thread.thread_management.tutorial.interruption.predefined_interruption_points
[raw-pointer-to-obj]: https://github.com/gnuradio/gnuradio/pull/2970/commits/d76c2ffdf20e1076eafd5aba83728548c59bfc69
[raw-pointer-to-std]: https://github.com/gnuradio/gnuradio/pull/2970/commits/9cb9ba9b62494a422c238bb387db0c038e6119bb
[PR2974]: https://github.com/gnuradio/gnuradio/pull/2974
