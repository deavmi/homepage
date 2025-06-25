---
title: "Addition to Niknaks: Bits and byte manipulation"
author: Tristan B. Velloza Kildaire
date: 2025-03-01
draft: true
---

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