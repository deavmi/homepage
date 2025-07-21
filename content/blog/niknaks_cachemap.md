---
title: "Addition to Niknaks: A generic CacheMap implementation"
author: Tristan B. Velloza Kildaire
date: 2025-07-21
draft: false
---

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

---

You can read the documentation [here](https://niknaks.dpldocs.info/niknaks.containers.CacheMap.html).