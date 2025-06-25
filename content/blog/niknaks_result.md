---
title: "Addition to Niknaks: A handy Result type"
author: Tristan B. Velloza Kildaire
date: 2025-03-01
draft: true
---

## Result type

I also decided on adding some more functional types to this release,
some of which I would realise later I _really_ needed in many of my
programs.

I therefore decided to add a **result** type, which is easier to
show through examples. Basically something that stores either
the result of a successful operation of the error thereof.

### Example code

We can create a `Result` with an _okay_ state (meaning
that it was a success) as follows:

```d
auto a = Result!(string, Exception).ok("A successful result");
```

Notice that we specified the type for the _"happy path"_
to be `string` and that of the _"unhappy path"_ to be of
`Exception`. By default if you don't specify the type
for the error then it will be made to be a `string`.

Now, we can check if it is okay as follows:

```d
assert(a.is_okay())
```

Following this we can then be assured that are proceeding
call to `okay()` will yield a valid result:

```d
assert(a.ok() == "A successful result");
```

One thing to noteis  that the types of the **Error** and
**Okay** can never be the same - a compilation error will
result if one would attempt to construct a `Result!(string, string)`.


There are several ways to check a `Result` for if it is
_okay_ or _erroneous_:

```d
// opCast to bool
assert(cast(bool)a);

// Validity checking
assert(a.is_okay());
assert(!a.is_error());
```

The `opCast(T: bool)()` overload is quite useful
as it let's you sue your result type like this:

```d
if(a)
{
    // do something
}
```

Instead of the more verbose:

```d
if(a.is_okay())
{
    // do something
}
```