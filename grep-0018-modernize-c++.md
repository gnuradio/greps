# GREP 0018 -- Modernize C++

- Original Author: Thomas Habets <thomas@habets.se>
- Champion: Thomas Habets <thomas@habets.se>
- Status: Draft

History:
- 20-Jan-2020: Initial Draft

## Abstract

As of early 2020 there is lots of pre-C++11 code in GNU Radio, and
some other pattern of non-idiomatic C++.

This GREP lays out how to get from here to there with minimal
breaking. Modernizing the code will make the code:

* more readable: standard components, less code, and explicit const
* safer: less manual memory management, prevent inherently broken copy
* faster: fewer pointer indirections, more information to the compiler
* less dependent on boost: To one day not require boost, which is sometimes a
  language on its own

All changes proposed here are aimed at at being done between major
versions (e.g. between 3.8 and 3.9), so minor API changes (e.g. adding
`const`, scoping `enum class` values) are OK.

## Copyright / License

This GREP is licensed as CC-BY-ND. Copyright 2020 Google Inc.

## Motivation

Cleaning up whole patterns of the code.

The legacy patterns this GREP lays out a plan for making the code base:

* more readable: standard components, less code, and explicit const
* safer: less manual memory management, prevent inherently broken copy
* faster: fewer pointer indirections, more information to the compiler
* less dependent on boost: To one day not require boost, which is sometimes a
  language on its own

## Description

### PR granularity

Most changes to be made should be scoped to one type of change in one
section. E.g. a PR with one commit saying:

* `digital: Make all non-changing member variables const`
* `dtv: De-pointer to remove manual memory management`
* `audio: Replace all raw pointers with smart pointers`

If the change is inherently multi-section, part of wide-spread
interfaces, or a very minor change repeated many times, then instead
expect PRs/commits like:

* `Replace boost smart pointers with std ones`
* `Replace assert with static_assert, where knowable at compile time`

If a change is trickier (but still worth it), then use a finer granularity such
as:

* `blocks: Simplify foo processing in file_sink`

These are guidelines for PR sizes for these changes. Common sense when a commit
is "too big" or "could change the other thing too" still applies.

### Add `const` where appropriate

This is not new with C++11, but `const` use could be better in the GNU Radio
code base.

Benefits:
* more readable; even for local variables a reader can see that a name has a
  final value
* avoids accidents
* possibly allows compiler optimizations

Recommendation:
* add `const` to large scope variables (e.g. member variables) when appropriate
* new code can have `const` for small scope (temporary) variables, but don't go
  back and change old code
* "large" and "small" scope border line is subjective, but 3 lines is small,
  member variable (indefinite time scope) is large

Risks: can cause very minor OOT break between major versions

### More `constexpr`

`constexpr` guarantees compile time evaluation.

Benefits over `const`:
* no risk of initialization order breakage
* can be used in some places where const can't
* enables compiler and link-time optimizations

Recommendation: Same as for `const`, see above.

Risks: can cause very minor OOT break between major versions

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

Another class of raw pointers is [arrays that should be changed to
`std::vector<>`][raw-pointer-to-vector]. If they require volk-specific
allocations they should use `volk::vector`.

Recommendation:
* switch all uses of needless pointers to pure objects
* for arrays, switch to `std::vector<>` or `volk::vector<>`.
* where it has to be a pointer, switch to `std` smart pointers

Risks: none

### Delete copy constructor / copy assignment

Where classes are not copyable (e.g. contain a pointer), the copy
constructor and copy assignment operator should be deleted.

```c++
    foo(const foo&) = delete;
    foo& operator(const foo&) = delete;
```

Recommendation:
* delete copy constructor & copy assignment when copying object is always a
  mistake (e.g. object is a singleton, or very expensive to copy and never
  should be)
* make object copyable/movable if copying the object is fine. E.g. replace all
  member pointers by smart pointers (converting raw pointer to `std::unique_ptr`
  correctly automatically prevents copy, but allows move)
* if making the object copyable is not feasible at this time, delete the copy
  constructor/assignment to prevent accidental copies

Risks: none. This is between major versions, and if it breaks OOT then
it was broken already.

### Boost smart pointers

Boost smart pointers currently provide no extra benefit over C++11 smart
pointers. The only thing missing in C++11 (which arrived in C++14) is
`std::make_unique()`. Until C++14 is required for GNU Radio that gap can be
filled by `gr::make_unique` which can start off being an alias for
`boost::make_unique()`.

Benefits of switching to `std`:
* readability for a larger population of developers
* reduction of dependency on Boost.
* more likely to we interoperable with third party libraries and OOT
  blocks

Recommendation:
* Merge a change like [PR 2974][PR2974], breaking ABI before 3.9 release.
* Use `boost::make_unique` to create new unique pointers. When C++14
  is allowed a global change to `std::make_unique` is trivial and
  breaks no API

Risks: can cause very minor OOT break between major versions

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
* use `std` locks and threads in new code
* leave existing boost ones ones alone, to be addressed by a future GREP

Risks: none

### `volk_malloc`

Likely all of these calls can be replaced by `volk::vector`, reducing
manual memory management, improving readability, and reducing chances
for mistakes like copy-paste ` * sizeof(float)` where ` *
sizeof(gr_complex)` was intended.

Recommendation:
* Replace all feasible uses of `volk_malloc` et al with `volk::vector`

Risks: master branch already depends on `volk::vector`, so none.

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

Risks: none

### `static_assert`

There are many uses of `assert()` (runtime check) that should be
`static_assert` (compile time check).

Example:

```c++
assert(sizeof(fftwf_complex) == sizeof(gr_complex));
```

Recommendation:
* Replace `assert` with `static_assert`, where possible

Risks: none

### Default member initialization

Pre-C++11 there were only two ways to initialize values, both in the
constructor:

```c++
class obj
{
private:
    int d_a, d_b, d_c;
    std::string d_s;

public:
    obj(int a) : d_a(a), d_b(0), d_s() // d_s() init is completely unnecessary
    {
        d_c = 3;
    }
    obj(int a, int b) : d_a(a), d_b(b) // Deplication of d_a(a).
    {
        d_c = 3; // Repeated init of d_c;
    }
};
```

This causes code duplication.

In C++11 default member values can be set at declaration time
(normally in header files):

```c++
class obj
{
private:
    int d_a = 0; // Or int d_a{0};
    int d_b = 0;
    int d_c = 3;

public:
    obj(int a) : d_a(a) {}
    obj(int a, int b) : d_a(a), d_b(b) // Duplication of d_a(a).
    {
    }
};
```

To completely avoid duplication one constructor can call another:

```c++
class obj
{
private:
    int d_a = 0; // Or int d_a{0};
    int d_b = 0;
    int d_c = 3;

public:
    obj(int a) : d_a(a) {}
    obj(int a, int b) : obj(a) // Call the other constructor.
    {
        d_b = b; // Can't be put in construct initializer when
                 // construction delegation is used.
    }
};
```

Recommendation:

* don't use constructor delegation with a non-empty body. It's
  semantically unclear what it means when object is constructed, but a
  constructor is still running.
  * no cases of this should exist in old code, as it's a C++11 feature
* don't needlessly list member variables for their default constructor
  (`d_s`, above)
  * remove existing cases of this, as they bloat the code
* for future code use default member initialization for default values. It's the
  least repetition in the face of multiple constructors, which the code may get
  one day if not already
  * don't change old code that uses constructor initialization, unless already
    changing that bit of the code
* for future code avoid initialization inside the constructor body,
  especially when it prevents a member being const
  * for old code change to constructor initialization only if doing other
    changes, or in order to make a member variable const

Risk: none

### `enum class`

Convert all `enum` to type-safe `enum class`. This is type safe and enables
better compiler warnings for `switch` cases with missing states.

Recommendation:
* internal enums: change them
* enums part of API: new enums should all be `enum class`, but leave
  existing ones alone

Risks: can cause very minor OOT break between major versions

## Loops

Use the appropriate algorithm, when possible, instead of a raw for
loop (not C++11-specific).

If not possible, use range based for loops, with `auto&` or `const
auto&`.

```c++
std::copy(from_vector.begin(), from_vector.end(),
          std::back_inserter(to_vector));
for (const auto& t : container) {
    […]
}
```

These loops are easier to get right, and easier to read, than the manual
init-check-increment loops, and avoids working with tricky iterators directly.

Recommendation:
* change these in old code where it doesn't cause too much refactoring, unless
  that refactoring is an improvement overall

Risks: none

### `override`

Add `override` when a virtual is overridden. This prevents surprising
bugs.

Recommendation:
* batch fix old code

Risks: none

### Pointer values `NULL`, `0`, `nullptr`

Use `nullptr`. Improves readability and some type safety.

Recommendation:
* batch fix old code

Risks: none

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
* API-breaking changes are fine before releasing a new major version,
  as long as they are switched all the way.

Risks: can cause very minor OOT break between major versions if part of API

[trigger-points]: https://www.boost.org/doc/libs/1_67_0/doc/html/thread/thread_management.html#thread.thread_management.tutorial.interruption.predefined_interruption_points
[raw-pointer-to-obj]: https://github.com/gnuradio/gnuradio/pull/2970/commits/d76c2ffdf20e1076eafd5aba83728548c59bfc69
[raw-pointer-to-std]: https://github.com/gnuradio/gnuradio/pull/2970/commits/9cb9ba9b62494a422c238bb387db0c038e6119bb
[raw-pointer-to-vector]: https://github.com/gnuradio/gnuradio/pull/2970/commits/e51bde49e83b0535106e7a8f11605a81584b1ba3
[PR2974]: https://github.com/gnuradio/gnuradio/pull/2974
