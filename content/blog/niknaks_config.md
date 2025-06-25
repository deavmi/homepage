---
title: "Addition to Niknaks: Using the configuration engine"
author: Tristan B. Velloza Kildaire
date: 2025-03-01
draft: true
---

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