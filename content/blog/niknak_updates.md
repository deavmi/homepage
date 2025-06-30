---
title: "Niknaks updates"
author: Tristan B. Velloza Kildaire
date: 2025-03-01
draft: true
---


// TODO: Fix date

# Introduction

Just thought I would describe what the `niknaks` project is. Firstly, it
isn't an application-side library, something that relates to any one _"idea"_
but rather just a collection of various utilities and re-usable datatypes.

It orirignally started when I begun work on a project of mine where I
was seeminly requiring common routines like:

* Array element presence checking
* Endianness conversions
    * And support for widths of 1 to 8 bytes

After some time I started working on other projects of mine whereby I needed
some functional constructs like _predicates_ and _optionals_, therefore I
went ahead and implemented those as common types accessible to my applications,
all of this being templatised of course.

As time went on I had added things like a delay mechanism, debugging tools,
prompting mechanism (think of CLI usage), configuration utility comprising
of configuration entries and a registry to manage them, a cache map implementation. (TODO: Add tree type)


Therefore I was able to collect all of these commonly used routines, types
and mechanisms into a common library that I called `niknaks`, all licensed
under the LGPL 3.0.

# Updates

It is now time to talk about the various additions I have made to this library
over the last few months because I have had great fun implementing the various
routines I need for other projects of mine **and** making them available to the
greater D community.

Below are just _some_ of the pull requests (some earlier ones were left out as
I just grabbed the ones which were already open in my browser's tabs).

| Changeset                                                | Link                                      |
|----------------------------------------------------------|-------------------------------------------|
| ⚡ Feature: Generic configuration mechanism              | https://github.com/deavmi/niknaks/pull/18 |
| ⚡ Feature: More array operations                        | https://github.com/deavmi/niknaks/pull/20 |
| ⚡ Feature: Improved prompting framework                 | https://github.com/deavmi/niknaks/pull/21 |
| ⚡ Feature: Add opIndex support to CacheMap              | https://github.com/deavmi/niknaks/pull/19 |
| ⚡ Feature: Prompting framework                          | https://github.com/deavmi/niknaks/pull/16 |
| ⚡ Feature: Buffer views                                 | https://github.com/deavmi/niknaks/pull/22 |
| ⚡ Feature: Generic tree and visitation framework        | https://github.com/deavmi/niknaks/pull/17 |
| ⚡ Feature: Insert at                                    | https://github.com/deavmi/niknaks/pull/23 |
| ⚡ Feature: Result type                                  | https://github.com/deavmi/niknaks/pull/25 |

# Example usage

I have put together several blog posts on how to use each of these
new features. You can use the table of contents below to take a
look at each:

1. [Using the new `CacheMap` type](../blog/niknaks_cachemap)
2. [Writing interactive prompts, _effortlessly_](../blog/niknaks_prompter)
3. [Managing configurations with the config engine](../blog/niknaks_config)
4. [Predicates and optionals](../blog/niknaks_predicates)
5. [Bit and byte manipulation routines](../blog/niknaks_bitmanip)
6. [Delay mechanism](../blog/niknaks_delay)
7. [Using the Result type](../blog/niknaks_result)


## Array tooling 

TODO: Add this

When programming you tend to work with arrays a lot and depending on what you
are doing you also may have to run a lot of algoritmns over said arrays, to
either check them for elements, manipulate them and so forth. Therefore as
time went on and I saw myself continuously re-implementing specific array routines
I decided I should just put them all in one place; thus `niknaks.arrays` was born.

Some of these methods of interest are:

| Method signature                        | Description                                                            |
|-----------------------------------------|------------------------------------------------------------------------|
| `bool isPresent(T)(T[] array, T value)` | Given an array of element type `T` check if `value` is present therein |
| `bool findNextFree(T)(T[] used, ref T found)` | Finds the next free **integral** element that is not present in the array, setting the `found` variable to said value if found (and returning `true`), `false` returned in case one not found. `T` must na an _integral_ type |
| `filter(T)(T[] filterIn, Predicate!(T) predicate, ref T[] filterOut)` | Runs a predicate over `filterIn`, and stores the matched elements into an array, passed by reference, called `filterOut` |
| `T[] shiftInto(T)(T[] array, size_t position, bool rightwards = false, bool shrink = false, T filler = T.init)` | Shifts a subset of the elements of the given array to a given position either from the left or right. _Optionally_ allowing the shrinking of the array after the process, otherwise the last element shifted's previous value will be set to the value specified. |
| `removeResize(T)(T[] array, size_t position)` | Removes the element at the provided position in the given array |
| `insertAt(T)(T[] array, size_t position, T value)` | Inserts the given value into the array at the provided index |
| `T[] unique(T)(T[] array)` | Returns a version of the input array with only unique elements. |

Let's look at one of these examples where I do right-wards shifting using a variant of the `shiftInto(...)` function:

```d
int[] numbas = [1, 5, 2];
numbas = numbas.shiftIntoRightwards(1);

// should now be [0, 1, 2]
writeln(numbas);
assert(numbas == [0, 1, 2]);

numbas = [1, 5, 2];
numbas = numbas.shiftIntoRightwards(0);

// should now be [1, 5, 2]
writeln(numbas);
assert(numbas == [1, 5, 2]);
```

The other functions are relatively self-explanatory and the documentation on Dub should show
you how they can be used with example code snippets.

## Debugging

Being able to easily generate debug messages for various structures
and doing so in a relatively generic way is quite helpful. All these
trinkets - sort of little functions that I use here and there, are
available from within the `niknaks.debugging` module.

| Method signature                        | Description                                                            |
|-----------------------------------------|------------------------------------------------------------------------|
| `dumpArray

TODO: Add examples
TODO: Finish vardump work


## Buffer views

One of the things I wanted to implement was what I called "buffer views".
These are effectively my own implementation of a _jump buffer_. This is
basically one big buffer which provides array-like access to its array-like
data _however_ the weay it is built is by composing various arrays into
it and smartly knowing which sub-array to index into when a request comes
in from the outside.

TODO: Finish this up
TODO: Add usage

### Example code

Creating a new `View` is as easy as follows. We can
also call `opDollar()` to get the length of the view
currently, notice indexing would fail as there is nothing
currently in the view:

```d
View!(int) view;
assert(view.opDollar() == 0);

try
{
    view[1];
    assert(false);
}
catch(ArrayIndexError e)
{
    assert(e.index == 1);
    assert(e.length == 0);
}
```

You can use the append operator `~=` in order
to add new data into the view. An important thing
to note about this is that no deep-copying of
the array `[1,3,45]` occurs:

```d
view ~= [1,3,45];
assert(view.opDollar() == 3);
assert(view.length == 3);

    view ~= 2;
    assert(view.opDollar() == 4);
    assert(view.length == 4);

    assert(view[0] == 1);
    assert(view[1] == 3);
    assert(view[2] == 45);
    assert(view[3] == 2);
    assert(view[0..2] == [1,3]);
    assert(view[0..4] == [1,3,45,2]);

    // Update elements
    view[0] = 71;
    view[3] = 50;

    // Set size to same size
    view.length = view.length;

    // Check that update is present
    // and size unchanged
    int[] all = view[];
    assert(all == [71,3,45,50]);

    // Truncate by 1 element
    view.length = view.length-1;
    all = view[];
    assert(all == [71,3,45]);

    // This should fail
    try
    {
        view[3] = 3;
        assert(false);
    }
    catch(RangeError e)
    {
    }

    // This should fail
    try
    {
        int j = view[3];
        assert(false);
    }
    catch(RangeError e)
    {
    }

    // Up-sizing past real size should not be allowed
    try
    {
        view.length =  view.length+1;
        assert(false);
    }
    catch(RangeError e)
    {
    }

    // Size to zero
    view.length = 0;
    assert(view.length == 0);
    assert(view[] == []);
}
```