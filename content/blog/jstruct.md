---
title: 🏗️ jstruct - Struct JSON serializer/deserializer for D!
author: Tristan B. V. Kildaire
date: 2023-06-29
draft: false
---

{{<bruh>}}
<img src="/projects/jstruct/logo.png" width=40% height=40% style="float:right;gap;margin-left:20px">
{{</bruh>}}

# Introducing `jstruct`

I decided to delve a little bit deeper into D's meta-programming templates and wanted to explore what one could do with it - this is by far nowhere as close as the cool things I did with [Project Guillotine](/blog/guillotine) but it did result in a library that does what I needed - **Struct to JSON serialisation** and **JSON to struct deserialization**.

You can checkout the [jstruct homepage](https://deavmi.assigned.network/projects/jstruct/), the source code [on GitHub](https://github.com/Hax-io/jstruct) or if you want to use it in your project checkout the [DUB page](https://code.dlang.org/packages/jstruct).

I wanted to be able to easily define some generic struct such as the following:

```d
struct Person
{
	public string firstname, lastname;
	public int age;
	public string[] list;
	public JSONValue extraJSON;
	public EnumType eType;
}
```

With say now values such as:
```d
Person p1;

// Set some values therein
p1.firstname = "Tristan";
p1.lastname = "Kildaire";
p1.age = 23;
p1.list = ["1", "2", "3"];
p1.extraJSON = parseJSON(`{"item":1, "items":[1,2,3]}`);
p1.eType = EnumType.CAT;
```

And at request turn that into a JSON string that would look something akin to:
```json
{
    "age": 23,
    "eType": "CAT",
    "extraJSON": {
        "item": 1,
        "items": [
            1,
            2,
            3
        ]
    },
    "firstname": "Tristan",
    "lastname": "Kildaire",
    "list": [
        "1",
        "2",
        "3"
    ]
}
```

---

## What's the big problem?

Well, the above may be very simple in a language that has all primitive types actually boxed to well, non-primitive *object-based* types. In such a situation, like in Java, one can query the runtime-type information (as one could in D, just for the sake of clarity) in order to fetch the type.

One could then easily map this to the correct JSON type and voila. Likewise, the reverse could also be done by querying annotated fields and using that as a way to set them and so forth.

**The problem is**, D *has primitive types* and we cannot query them at runtime in such a manner nor use reflection (even if we could or were using boxed types) as runtime reflection isn't a thing in D. Therefore I needed a way to use compile-time function execution to actually build custom code depending on the `struct` that was being passed in.

## Serialization

Serialization using jstruct is very easy, you can serialise the above struct `Person` as follows:
```d
JSONValue serialized = serializeRecord(p1);
```
The resulting JSON is then:
```json
{
    "age": 23,
    "eType": "CAT",
    "extraJSON": {
        "item": 1,
        "items": [
            1,
            2,
            3
        ]
    },
    "firstname": "Tristan",
    "lastname": "Kildaire",
    "list": [
        "1",
        "2",
        "3"
    ]
}
```
The [inner workings](https://github.com/Hax-io/jstruct/blob/master/source/jstruct/serializer.d#L18) of this are quite weird looking but effectively I am telling the D compiler to repeat code for each struct member of `Person` and depending on their type generate different code for each member and set an exact (known at compile time) member of the blank JSON object (created at runtime).

## Deserialization
I really wish I could say the reverse was as easy as the forward way but there are some caveats. One can easily rely on `to!(T)(K)` in order to aid with array conversions and enum conversions but when working with what we have we need to do a lot of checks for it. 

[*Internals aside*](https://github.com/Hax-io/jstruct/blob/master/source/jstruct/deserializer.d#L21) this is how deserialization is done. If we have been given the following JSON (and I am now using a different example because why not):

```json
{
	"firstname" : "Tristan",
	"lastname": "Kildaire",
	"age": 23,
	"obj" : {"bruh":1},
	"isMale": true,
	"list": [1,2,3],
	"list2": [true, false],
	"list3": [1.5, 1.4],
	"list4": [1.5, 1.4],
	"list5": ["baba", "booey"]
}
```
And we want to deserialize it to the following struct:
```d
struct Person
{
	public string firstname, lastname;
	public int age;
	public bool isMale;
	public JSONValue obj;
	public int[] list;
	public bool[] list2;
	public float[] list3;
	public double[] list4;
	public string[] list5;
}
```
Then we simply have to call the following:
```d
Person person = fromJSON!(Person)(json);
```

The result of which is:
```
Person("Tristan", "Kildaire", 23, true, {"bruh":1}, [1, 2, 3], [true, false], [1.5, 1.4], [1.5, 1.4], ["baba", "booey"])
```

---

## Using it in your next D project

In order to use jstruct in your project simply run:

```bash
dub add jstruct
```

And then in your D program import as follows:

```d
import jstruct;
```