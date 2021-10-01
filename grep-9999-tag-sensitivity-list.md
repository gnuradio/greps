# GREP [XXXX] -- Tag Sensitivity Lists

- Original Author: Marcus Müller <mmueller@gnuradio.org>
- Champion: Marcus Müller <mmueller@gnuradio.org>
- Status: Draft

History:
- 02-Oct-2021: Initial Draft

## Abstract

Reacting to tags to change properties, or more generally state, of a block is a
very common pattern in the GNU Radio code base and out-of-tree modules.

However, neither are all setters wrapped for tags, nor is the handling of tags
that is already implemented consistent.

This GREP introduces a method of delegating the handling of tags to GNU Radio
itself, allowing for zero-code handlers, consistent documentation of
capabilities and establishing of code patterns for this common task.

On the way there, we also establish a consistent method for producing message
port setters for the same properties.

## Copyright / License

This GREP is licensed as CC-BY-4.0; original author as stated above.

## Motivation

It is commonly requested feature to be able to set the property of a block,
accessible through setters, through stream tags.

The implementation of that would, under current circumstances, be not quite
straightforward. To illustrate the steps and decisions to be made in such a
situation, let's discuss how one would extend a simple block like "Interpolating
FIR Filter" block with both stream tag and message port setters accepting PMT
uniform vectors.

1. Implement thread-safe native and message handling setters. This can either be
   achieved by:
   - Having a `private` thread-unsafe native setter
     `set_taps_unsafe(std::vector<tap_t>)`; acquiring the `block::d_setlock`
     mutex in a `public` setter `set_taps(std::vector<tap_t>)` and in the
     critical part of `work` and a PMT-accepting setter `set_taps(const
     pmt_t&)`, both calling the thread-unsafe native setter. Register the
     PMT-accepting setter as message handler.
   - Implementing and registering a `private` message handler method
     `set_taps(const pmt_t&)`, which accepts a PMT, extracts the native type
     from the that and is called by GNU Radio only when `work` is not currently
     running. Implement a native setter `set_taps(std::vector<tap_t>)` method,
     which converts the `std::vector` of taps to a PMT vector and posts it as a
     message to the own message port. 

In both cases, the developer implements *both* a "native" and a "PMT" type setter,
with one just being boilerplate code invoking the other. Both options have their
potential performance downsides, with overly coarse granularity, i.e. `work`
call granularity, in case of the message handler, or a mutex acquisition
overhead if the critical section for the manual mutexing is chosen very finely.

However, after this, both the thread-safe native as well as the inherently
unaffected message handler setters are implemented.

2. Implement stream tag handling in `work`. This can also reasonably take two
   shapes. In both cases, one would acquire all key-matching tags in the current
   workload through `get_tags_in_window` first.
   - Iterating through the acquired vector in a range-based `for` loop, the
     filter processes all unconsumed samples up to the tag, consumes that number
     of items, then adjusts the filter taps by directly calling the
     PMT-accepting setter, producing the appropriate amount of items and moving
     on to the next tag, all within the same call to `work`.
   - Checking but the offset of the first tag returned by `get_tags_in_window`,
     processing and consuming all items leading up to that offset, and only then
     either calling directly into the PMT-accepting setter, or emitting the
     received tag as message to the own message handler port, before returning
     from `work`.

Both methods have their disadvantages: While in the first method, there's no
overhead in fetching the same tags multiple times, if there happens to be more
than one tag returned, but it cannot be generalized, so that a developer would
only have to "fill in" a tag-aware method in a very GNU Radio `work`-like
manner.

## Description

To ease this situation, the proposed solution is to:

Add a protected `std::unordered_set<gr::property_setter> sensitivity_list`
member to `gr::sync_block` (later extension to `gr::block` nonwithstanding).

The set contains elements of a yet to be implemented type `gr::property_setter`.
Its constructor takes a `const std::string& key` as well as a
`std::function<bool(const type_of_property&)> underlying_setter`. It has an
`operator(const type_of_property&)`, which actually calls the thread-unsafe
setter. Furthermore, it has an `operator(const pmt_t&)`, which automatically
converts the PMT to the field type prior to setting the field. The hashing and
comparison methods necessary for `unordered_set` of the type are delegated to
the PMT `pmt::symbol(key)`.

If `sensitivity_list` is not empty, the GNU Radio scheduling algorithm
(`run_one_iteration`), instead of directly calling into work, looks only for the
first tag matching one of the entries in said list. If there is none, `work` is
called as usual.

If there is one, only the first one is acquired (requiring implementation of
`get_first_tag_in_range(start, stop, keylist)` to avoid copying irrelevant
tags). The items leading up to the tag are passed to `work`. If `work` consumes
all these items, the `property_setter` is invoked with the tag value.

### Projected usage

```c++
// Block without sensitivity: simply don't change anything
class boring_blk : virtual public gr::block
{
    // ...
};
class boring_blk_impl : public boring_blk
{
    boring_blk_impl()
        : sync_block("boring_blk", io_signature::make(), io_signature::make())
    {
        // construct boring stuff
    }
};

// Block with sensitivity list: initialize the tag sensitivity list
class sensitive_blk : virtual public gr::block
{
    // ...
};
class sensitive_blk_impl : public sensitive_blk
{
    sensitive_blk_impl()
        : sync_block("sensitive_blk", io_signature::make(), io_signature::make()),
        sensitivity_list({get_specialized_setter()})
    {
        // construct exciting stuff
    }
};
```

### Proposed Implementation of Types

```c++ namespace gnuradio::gr
{
    struct property_setter {
        std::shared_ptr<gr::block> owner;
        const std::string key;
        const pmt_t symbol;
        property_setter(std::shared_ptr<gr::block> owner, const std::string& key)
            : owner(owner), key(key), symbol(pmt::symbol(symbol))
        {
            owner->message_port_register_in(symbol);
        }
    };
    class property_setter_hash
    {
        std::size_t operator(const property_setter& prop_setter) const noexcept
        {
            return std::hash<>(prop_setter.symbol);
        }
    };

    template <typename property_type>
    struct specialized_property_setter : public property_setter {
        using setter_type = std::function<bool(const property_type&)>;

        const setter_type setter;
        specialized_property_setter<property_type>(std::shared_ptr<gr::block> owner,
                                                   const std::string& key,
                                                   setter_type underlying_setter)
            : property_setter(owner, key), setter(underlying_setter)
        {
            owner->set_msg_handler([](const pmt_t& val) { this->operator()(val); });
        }

        bool operator(const property_type& value) const
        {
            if (!setter(value)) {
                owner->d_logger->warn("{} property_setter: failed setting", key);
                return false;
            }
            return true;
        }

        // specialize in template specializations for property_type=float, complex<float>,
        // int…
        // Most elegant would be to have a templated pmt_basic::to<target_type>()
        // operator?
        // Does pmtf solve this?
        /*
        bool operator(const pmt::pmt_t& polymorph) const
        {
            return setter(pmt_to_property_type(polymorph));
        }
        */
    };
}
```
