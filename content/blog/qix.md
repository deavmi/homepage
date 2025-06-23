---
title: "Qix: Simple waitable-queue management"
description: "An introduction to my new waitable-queue management library called Qix"
author: Tristan B. V. Kildaire
date: 2025-06-23
draft: false
---

## Introduction

As part of an still-undisclosed project of mine that I have been working on for the
past 2 months or so is the requirement for some sort of protocol-agnostic request-response
matching system.

As such a system is normally rather generic it could potentially be used in other projects
of mine in the future, where a similar use case is required. _Or_ you could even use it! It
is for this reason that I decided to redesign what was already ongoing as a redesign within
my **older** [tristanable](../../projects/tristanable/) project.

This entailed stripping out any of the network aspects of tristanable, namely:

1. The `Socket` references
2. The message encoding/decoding routines

Removing 1 and 2 was done so as to result in a library solely focused on the actual
matching engine. What would be left to the user would be to design a wire format for
the actual messages that might be sent over something like a `Socket` - just for example.

## Usage

Let's first define the `Message` type which will describe what it is
that we will be enqueuing onto our queue:

```d
// item type
struct Message
{
	private string _t;
	this(string t)
	{
		this._t = t;
	}

	public string t()
	{
		return this._t;
	}
}
```

Now let's create a manager which will let us create and manage
our queues:

```d
// queue manager for queues that hold messages
auto m = new Manager!(Message);
```

Now I will create two new queues. Their queue IDs will be randomly
generated for us:

```d
// create two new queues
Result!(Queue!(Message)*, string) q1_r = m.newQueue();
Result!(Queue!(Message)*, string) q2_r = m.newQueue();
```

When generating a new queue, there is a theoretical limit to
the number we can have and technically if reached you would
have an error. There's also an internal iteration limit to
how many times a roll-of-the-dice random number generation
can fail. For these two reasons we need to check for such 
errors on our two result objects `q1_r1` and `q2_r`:

```d
assert(q1_r.is_okay());
assert(q2_r.is_okay());
auto q1 = q1_r.ok();
auto q2 = q2_r.ok();
```

Let's create two new messages:

```d
Message m1_in = Message("First message");
Message m2_in = Message("Second message");
```

Now, a _seperate thread_ (for the purpose of demonstration)
we would be able to enqueue our messages onto each of the
queues. Also note that if the enqueuing was successful
then the `receive(T)` method will return `true`, else
`false`. This is dependent on the `AdmitPolicy`, which
if not specified, would always evaluate to `true`:

```d
// enqueue two messages, one per queue
assert(q1.receive(m1_in)); // should not be rejected
assert(q2.receive(m2_in)); // should not be rejected
```

You can also, indirectly, do so via the `Manager` we
instantiated earlier:

```d
// via the manager, note the queue id usage
Result!(bool, QixException) eq_1 = m.receive(q1.id(), m1_in);
Result!(bool, QixException) eq_2 = m.receive(q2.id(), m2_in);

// the queues are known to be registered with this
// manager hence this should pass
assert(eq_1);
assert(eq_2);

// extracting the "okay" value we can check that the
// admit policy should pass (be true)
auto eq_1 = eq_1.ok();
auto eq_2 = eq_2.ok();
```

Now on _**another** set of threads_ we could be waiting
for the message to arrive on the first queue:

```d
assert(q1.wait() == m1_in); // should be the same message we sent in
```

And another thread waiting for the message to arrive
on the second queue:

```d
assert(q2.wait() == m2_in); // should be the same message we sent in
```

> Methods exists for doing the `wait()` via the `Manager` **and**
for doing a timed-wait as well