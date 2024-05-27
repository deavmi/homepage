---
title: "Niknaks updates"
author: Tristan B. Velloza Kildaire
date: 2024-04-01
draft: true
---


// TODO: Fix date

# Introduction

Just thought I would describe what the `niknaks` project is. Firstly, it
isn't an application-side library, something that relates to any one _"idea"_
but rather just a collection of various utilities and re-usable datatypes.

It orirignally started when I begun work on a project of mine where I
was seeminly requiring common routines like:

* Array element prescence checking
* Endianness conversions
    * And support for widths of 1 to 8 bytes

After some time I started working on other projects of mine whereby I needed
some functional constructs like _predicates_ and _optionals_, therefore I
went ahead and implemented those as comon types accessible to my applications,
all of this being templatised of course.

As time went on I had added things like a delay mechanism, debugging tools,
prompting mechanism (think of CLI usage), configuration utility comprising
of configuration entries and a registry to manage them, a cache map imlementation. (TODO: Add tree type)


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

## CacheMap

One of the additions I made earlier this year was that of a cache map. This is
effectively a type which provides a map-like interface where you have a key of
type `K` and it should return a value of type `V`. That should be obvious. Where
the difference comes in is that the mapping is not permanent and only lasts for
a given lifetime (there is a thread for checking for expiration). With this in mind
everytime you lookup a value `K` then if the entry is not there (either exxpired or
never created) then a _"filler"_  function (which returns a value of type `V`)
will be called with the value of your key and fill the entry and then return it.

Now, accesses made within the timeout period will not call that filler function
again and you will ba accessing a cached value, that's the idea.

### Example code

Below is an example usage of the `CacheMap` which is actually part of the test
suite:

```d
int i = 0;
int getVal(string)
{
    i++;
    return i;
}

// Create a CacheMap with 10 second expiration and 10 second sweeping interval
CacheMap!(string, int) map = new CacheMap!(string, int)(&getVal, dur!("seconds")(10));

// Get the value
int tValue = map["Tristan"];
assert(tValue == 1);

// Get the value (should still be cached)
tValue = map["Tristan"];
assert(tValue == 1);

// Wait for expiry (by sweeping thread)
Thread.sleep(dur!("seconds")(11));

// Should call replacement function
tValue = map["Tristan"];
assert(tValue == 2);

// Wait for expiry (by sweeping thread)
writeln("Sleeping now 11 secs");
Thread.sleep(dur!("seconds")(11));

// Destroy the map (such that it ends the sweeper)
destroy(map);
```

## Prompter and prompts

When working on the Tristan's Programming Language's package manager `tpkg` I
realised the quick need for a mechanism that would let me build up custom
questions and then have the answers be provided by the means of some `File`.

I wanted to basically be able to prompt the user, on the command-line, for
answers to several questions such as _"What is the name of the new module?"_
or _"What description do you want to have set?"_ and then have it do the
prompting for me and storing of the respective answers.

Well, that's what this basically does.

### Example code

A nice example of the prompting is a snippet of code out of the `tpkg`
code base itself. Here I prompt for a few things, notably I require the
answers for the prompts to the _package name_ and  _package description_
to be non-empty (that is the second `false` given to the `Prompt` constructor;
the first `false` means to not request a multiple-value answer).

```d
Prompter prompter = new Prompter(stdin);
prompter.addPrompt(Prompt("Project name: ", false, false));
prompter.addPrompt(Prompt("Project description: ", false, false));


Prompt[] answers = prompter.prompt();

Project proj;
string projName;
if(answers[0].getValue(projName))
{
    proj.setName(projName);
}
else
{
    ERROR("Could not get a valid project name");
    return;
}
string projDescription;
if(answers[1].getValue(projDescription))
{
    proj.setDescription(projDescription);
}
else
{
    ERROR("Could not get a valid project description");
    return;
}
```

## Configuration engine

The configuration engine is something I put together out of
a need for it in several projects, one of which was my `tpkg`
and the TLang compiler itself. Originally beginning as a simple
abstraction over a union of types and a tag (to indicate the type
and provide safety during runtime) I made use of it as part
of the configuration engine for my compiler. I then realised
I could spin out that logic into a generic configuration
mechanism. (TODO: TLang to use it)

### Entry types

There are a few entry types that are supported, these can
be found within the `ConfigType` enum:

| Type name | D's equivalent type |
|-----------|---------------------|
| `TEXT`    | `string`            |
| `NUMERIC` | `int`               |
| `FLAG`    | `bool`              |
| `ARRAY`   | `string[]`          |


### The configuration entry

The entry itself is easy enough to construct using
one of the static factories at your disposal via
the `ConfigEntry` struct type. I have constructed
a few example use cases of this type below to show
the proper usage thereof.

Below we test out using the configuration entry and
its various operator overloads:

```d
ConfigEntry entry = ConfigEntry.ofArray(["hello", "world"]);
assert(entry[] == ["hello", "world"]);

entry = ConfigEntry.ofNumeric(1);
assert(entry.numeric() == 1);

entry = ConfigEntry.ofText("hello");
assert(cast(string)entry == "hello");

entry = ConfigEntry.ofFlag(true);
assert(entry);
```

Tests out the erroneous usage of a
configuration entry. In this case it
is because we are attempting to extract
an `ARRAY` value out of an entry which is
of type `TEXT`:

```d
ConfigEntry entry = ConfigEntry.ofText("hello");

try
{
    entry[];
    assert(false);
}
catch(ConfigException e)
{
        
}
```

Tests out the erroneous usage of a
configuration entry which, in this
case, is becuase we are trying to
extract _any_ value out of it (here
I just attempted an `ARRAY` value
for the purpose of demonstration)
**without** having set any value
in this entry:

```d
ConfigEntry entry;

try
{
    entry[];
    assert(false);
}
catch(ConfigException e)
{
    
}
```

### The `Registry` and its entries

The `ConfigEntry`(s) can be used just by themselves
and whatever custom map type you would want to write
to potentially acommodate them. However, if you are
looking for a ready-made batteries-included solution
then look no further than the `Registry` and its
`RegistryEntry`(s), the latter of which wraps the
`ConfigEntry`(s) of yours, easily. The registry
provides the ability to associate names with
entries and various rules on updating them and
so forth, a true mapping facility to hold all
the dense attributes for properties one would
need to store (and potentially modify) during
application runtime.

Here is some initial usage of the `Registry`
whereby we will:

* Create a new `Registry` with the overwriting
of existing registries via calls to `newEntry(...)`
for the same entry will **FAIL** (hence the `false`
passed in)
* We add a new entry and then check it is there
* We then try adding it again which then **FAILS**

```d
Registry reg = Registry(false);

// Add an entry
reg.newEntry("name", ConfigEntry.ofText("Tristan"));

// Check it exists
assert(reg.hasEntry("name"));

// Adding it again should fail
try
{
    reg.newEntry("name", ConfigEntry.ofText("Tristan2"));
    assert(false);
}
catch(RegistryException e)
{

}
```

We also have some useful overloads such as the `opCast(T)`
which means that if you cast to a type which maps to a
supported `ConfigType` then you can effectively extract
the value like that:

```d
// Check that the entry still has the right value
assert(cast(string)reg["name"] == "Tristan");
```

---

Now talking about adding new entries. You can use the
`newEntry(...)` method _or_ you can make use of the
indexing operator as follows:

```d
// Add a new entry and test its prescence
reg["age"] = 24;
assert(cast(int)reg["age"] == 24);

// Update it
reg["age"] = 25;
assert(cast(int)reg["age"] == 25);
```

When using the above operator overload (the index
operator) then it will create an entry if it
doesn't exist and overwrite it if it does (**irrespective**
of whether or not the overwriting policy).

This is in comparison to `setEntry(...)` which
always allows the over-writing of entries (unlike
the `newEntry(...)` counterpart) **however**
unlike its indexing-operator counterpart it **cannot**
create (and then set) entries if they did not
already exist. This is shown below:

```d
// Should not be able to set entry it not yet existent
try
{
    reg.setEntry("male", ConfigEntry.ofFlag(true));
    assert(false);
}
catch(RegistryException e)
{

}
```

---

We also provide access to the memory space that 
the `ConfigEntry`, at a given key, occupies via:

```d
// Obtain a handle on the configuration
// entry, then update it and read it back
// to confirm
ConfigEntry* ageEntry = "age" in reg;
*ageEntry = ConfigEntry.ofNumeric(69_420);
assert(cast(int)reg["age"] == 69_420);
```

Wanted to get all the entries in the registry?
Well, then you can as follows:

```d
// All entries
RegistryEntry[] all = reg[];
assert(all.length == 2);
writeln(all);
```

The `RegistryEntry` type has a `getName()`
which returns a `string` of the entry's
name. Then it also has a `getValue()`
which returns its associated `ConfigEntry`.

## Predicates and Optionals

One of the things that come sup quite a lot when programming
and dealing with arrays of data (items of the same data type)
is the ability to programatically filter such collections of
data by some form of reusable component - enter _the predicate_.

The predicate is not any sort of new wrapper type I came up
with, but rather it is a templatised (type-parameterized) alias
which when used, expands into a `bool delegate(T)` - some delegate
which takes in a single argument of type `T` and returns a `bool`,
a _verdict_.

The definition is as follows:

```d
/** 
 * Predicate for testing an input type
 * against a condition and returning either
 * `true` or `false`
 *
 * Params:
 *    T = the input type
 */
template Predicate(T)
{
	/**
	 * Parameterized delegate pointer
	 * taking in `T` and returning
	 * either `true` or `false`
	 */
	alias Predicate = bool delegate(T);
}
```

There is also a handy `predicateOf(alias)` template used for
constructing predicates from some symbol (either a function
or a delegate).

---

Now onto _optionals_. An optional is basically a safe way to
pass around a value which _may or may **not**_ be there. So
it is "null-safe" (or whatever the functoid people say).

You can construct an `Optional!(T)`, potentially fill it
with a value (depends on your code) and then return it.
The receiving code will check if a value is present _and only
if so_ then it will extract said value out.

### Example code

#### Predicate example

Suppose we have some function as shown below:

```d
private bool isEven(int number)
{
    return number%2==0;
}
```

We can then construct a predicate out of it
and test it against some input values:

```d
/**
 * Uses a `Predicate` which tests
 * an integer input for evenness
 *
 * We create the predicate by
 * passing in the symbol of the
 * function or delegate we wish
 * to use for testing truthiness
 * to a template function
 * `predicateOf!(alias)`
 */
unittest
{
	Predicate!(int) pred = predicateOf!(isEven);

	assert(pred(0) == true);
	assert(pred(1) == false);

	bool delegate(int) isEvenDel = toDelegate(&isEven);
	pred = predicateOf!(isEvenDel);

	assert(pred(0) == true);
	assert(pred(1) == false);
}
```

#### Optional example

Below we create an `Optional` that is to
potentially hold a `byte` value. In this
case you can construct it from the get go
with a value, but if not then it will be 
empty (it is a `struct` type).

```d
/**
 * Creating an `Optional!(T)` with a
 * value present and then trying to
 * get the value, which results in
 * said value being returned
 */
unittest
{
	Optional!(byte) f = Optional!(byte)(1);
	assert(f.isPresent() == true);

	try
	{
		assert(1 == f.get());
	}
	catch(OptionalException)
	{
		assert(false);
	}
}
```

What you can then do is call `isPresent()` which
will return `true` if a value is present, otherwise
`false`. Any attempt to then call `get()` will throw
an exception **if and only if** the call to `isPresent()`
prior returned `false`, otherwise the byte value
is returned.

## Array tooling 

TODO: Add this

When progranming you tend to work with arrays a lot and depending on what you
are doing you also may have to run a lot of algoritmns over said arrays, to
either check them for elements, manipulate them and so forth. Therefore as
time went on and I saw myself continuously re-implemting specific array routines
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

## Bits

There are some byte flipping utilities I required for a library of mine (which, seeing how
busy I am, is still in the works - so it isn't something I am talking about yet).

| Method signature                        | Description                                                            |
|-----------------------------------------|------------------------------------------------------------------------|
| `T flip(T)(T bytesIn)`                  | Flips the given integral value (note, that `T` has to be an integral type) |
| `T order(T)(T bytesIn, Order order)`    | Swaps the bytes to the given ordering but does a no-op if the ordering requested is the same as that of the system's |
| `ubyte[] toBytes(T)(T integral)`        | Converts the given integral value to its byte encoding |
| `bytesToIntegral(T)(ubyte[] bytes)`     | Takes an array of bytes and dereferences then to an integral of your choosing |

Some examples of these, below I show the flipping of bytes:

```d
version(BigEndian)
{
    ushort i = 1;
    ushort flipped = flip(i);
    assert(flipped == 256);
}
else version(LittleEndian)
{
    ushort i = 1;
    ushort flipped = flip(i);
    assert(flipped == 256);
}
```

Then I show a byte-order _platform-sensitive_ flipper which will only flip if the requested ordering
is not equal to that of the platform's byte order:

```d
version(LittleEndian)
{
    ushort i = 1;
    writeln("Pre-order: ", i);
    ushort ordered = order(i, Order.BE);
    writeln("Post-order: ", ordered);
    assert(ordered == 256);
}
else version(BigEndian)
{
    ushort i = 1;
    writeln("Pre-order: ", i);
    ushort ordered = order(i, Order.BE);
    writeln("Post-order: ", ordered);
    assert(ordered == i);
}
```

## Debugging

Being able to easily generate debug messages for various structures
and doing so in a relatively generic way is quite helpful. All these
trinkets - sort of little functions that I use here and there, are
available from within the `niknaks.debugging` module.

| Method signature                        | Description                                                            |
|-----------------------------------------|------------------------------------------------------------------------|


TODO: Add examples
TODO: Finish vardump work

## Delay mechanism

One of the things I decided to put together was a programming
structure that I thought would be usable in a scenario as follows:

> You have run some task on some remote server and now you want to
regularly check in with it and see when a reply is sent back that
satisfies some requirement. You _also_ can't do this forever so you
want to set an interval for how frequently you re-check and then also
a total timeout time which will be your deadline - when you exceed
it then you stop checking.

This mechanism is called the `Delay` and can be found in the
`niknaks.mechanisms` package.

The constructor appears as follows:

```d
/** 
 * Constructs a new delay mechanism
 * with the given delegate to call
 * in order to determine the verdict,
 * an interval to call it at and the
 * total timeout
 *
 * Params:
 *   verdictProvider = the provider of the verdicts
 *   interval = thje interval to retry at
 *   timeout = the timeout
 */
this(VerdictProviderDelegate verdictProvider, Duration interval, Duration timeout)
{
    this.verdictProvider = verdictProvider;
    this.interval = interval;
    this.timeout = timeout;
}
```

Therefore you can set those two parameters mentioned earlier. The first
parameter is the "check" that is to be periodically called. The delay
system will return successfully if at some point this `VerdictProviderDelegate`
returns a `true` _within_ the timeout window of `timeout`. If you
keep getting `false` returned and go over the `timeout` period then
a `DelayTimeoutException` is thrown.

### Example code

Here is an example where my verdict function (technically a delegate in
the case of a D unittest) which refers to a variable `cnt`. This example
illustrates the multiple retry calls to said function due to it only
returning a `true` verdict on the _second_ call.


```d
/**
 * Tests out the delay mechanism
 * with a verdict provider (as a
 * delegate) which is only true
 * on the second call
 */
unittest
{
    int cnt = 0;
    bool happensLater()
    {
        cnt++;
        if(cnt == 2)
        {
            return true;
        }
        else
        {
            return false;
        }
    }

    Delay delay = new Delay(&happensLater, dur!("seconds")(1), dur!("seconds")(1));

    try
    {
        delay.go();
        assert(true);
    }
    catch(DelayTimeoutException e)
    {
        assert(false);
    }
}
```