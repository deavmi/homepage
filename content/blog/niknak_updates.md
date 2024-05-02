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

TODO: Add this

## Predicates, Optionals and some tooling

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