---
title: "Addition to Niknaks: Optional types and predicates"
author: Tristan B. Velloza Kildaire
date: 2025-08-06
draft: true
---

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