---
title: "Addition to Niknaks: Buffer views"
author: Tristan B. Velloza Kildaire
date: 2025-03-01
draft: true
---

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