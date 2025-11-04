---
title: How Go's `errors.Is(error, error) bool` works?
author: Tristan B. V. Kildaire
date: 2025-11-04
draft: false
---

## Why?

I have only been learning go for about the last 3 weeks or so
but I have a relatively good grasp on programming by now and
also enjoy reading language runtime and standard library
internals as they're the only real way to figure things
out when the documentation is a little bit too hazy.

## How it works

If you want to get a good idea about writing some very concice
runtime-type checking code (i.e. checking if a given object
implements a certain interface) then give this a look:

Source: https://cs.opensource.google/go/go/+/refs/tags/go1.25.3:src/errors/wrap.go;l=44

This is the implementation of `errors.Is(error, error) bool` which
takes a left-hand side error and sees if it matches the right-hand
side error. The entry point is a relatively basic check for nullity
but then it moves onto a private function which does the runtime
type checking for two things.

If they don't match via `==`, a check only done when the types
are comparable (the argument passed to the private function let's
us know whether or not we can do so). Remember, you could have
a slice that is a kind-of `error` as it could implement the `Error() string`
method; so you must have a check for that else a lovely runtime
error will occur.

Okay, great, maybe they don't match by `==` (could be possible,
maybe your basic `error` implementation had a `msg string` field
and those didn't match).

Then it moves onto checking "Do you support `interface { Is(error) bool }`?"
which is whatever you implemented it to be. I personally like
using a good ol' `reflect` to get the runtime type information
and get the type of it from that; so I can replicate Java's
`instanceof`-based matching for `try+catch`. But it's up to you!

Okay, if a direct "this error does not match via `Is(error)` then
it checks; do you have support for either an `interface {Unwrap() error}`
or `interface{Unwrap() []error}` and in the former case it
recurses and in the latter it iteratively recurses (bubbling back
up and exiting the for-loop if found).


If all else fails, then no match.
