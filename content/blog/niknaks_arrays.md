---
title: "Addition to Niknaks: Tooling for arrays"
author: Tristan B. Velloza Kildaire
date: 2025-03-01
draft: true
---

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