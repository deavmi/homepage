---
title: New goals for Eventy
author: Tristan B. V. Kildaire
date: 2021-10-23
draft: true
---

# The current state

The goals have shifted for Eventy. What started off as what I have now figured out is referred to _"signals and slots"_ which is effectively to say that you associate a _input signal_ like `push(Event)` where `e.id` maybe is equal to `8`, and you associate that
number `8` with a _slot_ that handles anything of _"type"_ `8`. Given 5 events being pushed into such a system we then end with `5+1` threads, the `+1` accounting for the caller main thread and the other 5 being the handlers that were dispatched for the respective slot(s). This is effcetively what the v.0.4.x series of Eventy does now.

Previosuly, the inefficient mechanism of an event-loop and queue was used but this wasn't a sleeping loop but rather a yielding loop. Which meant it would run rather often even when it didn't need to. What would have been better would be the use of `sleep()` (indeffinately)
and then signalling that process to wake it up. Some precautions, as I have now read, would probably need to be taken into account. Such would be checking the received signal and if it matches the one we expect - sometimes the kernel can send us signals like when a thread leader quits (or something I believe) - I am still learning (never used `clone()` with `CLONE_THREAD` a lot, normally just used `CLONE_VM` and maybe or'd that with `CLONE_FILES` (or whichever of that of `_FD` it was)). This would have been a better solution and could have allowed us to add some priorities to events. Yes, we would slow down all execution by having no immediate dispatch but the playing field would have been evened out then, _however_ we would have gaine dthe ability the prioritise certain dispatches potentially _or_ have low priority events that could be dispatched way later on.

# What we want

The next section discusses the ideas I would like toimplement into Eventy.

## Promises

The idea of promises is rather simple. WHen you make a call to `push(Event e)` you will get a `Promise` object returned instead of the method being a `void push(Event)` as it stands today. This `Promise` object will then have the following methods which are of interest to us:

1. `await(int timeout = 0)`
    * The default timeout being `0` implies block-forever
2. `then(Event e)`

### `await()`

We could write a program as follows:

```d
// Create a promise
Promise promise = engine.push(new Event(1));
```

Then later on we could provide this _Promise_ `promise` to maybe let's say two threads **T1** and **T2** and both of them could call the method `await()` as follows:

---

###### Thread 1

```d
promise.await()
```

###### Thread 2

```d
promise.await()
```

---

This would then block on bith **T1** and **T2** until the signal handler registered for events of type `1` (recall our `new Event(1)`)
finishes. A timeout can be given, ~~in such a case we should probably return a boolean `false` if a timeout occured, `true` if not - meaning the event finished~~ and an enum is returned indicating either a timeout, unblocked from successful execution or unblcoked because there was an error in the signal handler (we will need to investigate indicating this last one well).

### `then(Event e)`

THe `then(Event)` method should provide the ability to chain another `Event`, in this case `e`, upon completion of the signal handler
for the original event. This, however, might require we do things a little differently.

We might need a method to create the promise, without executing it, such that we can perform such chaining.

```d
// Create the promise first of all
Event myEvent = new Event(1);
Promise promise1 = engine.promise(myEvent);

// Create another event
Event myEvent2 = new Event(2);

// Upon completion og `myEvent` by whatever signal handler it uses
// then execute the given `myEvent2` by its respective signal handler
promise1.then(myEvent2);
```

This should firstly, create the first `Event`, then create a `Promise` (`promise1`). Then we create the second `Event`,
and chain that to `promise1` (internally the `.then(Event)` call will also create a `Promise` for us.

The `.then(Event)` returns another promise (call it _`promise2`_).

---

One should setup their awaits before the next step (very important or else the dispatcher may not find any waiting threads in the `Engine`'s queues):

```d
// Thread 1
promise1.await();

// Thread 2
promise1.await();
```

---

But now we need to `emit(Promise)` which will actually begin execution:

```d
engine.emit(promise1);
```

This will then start the whole system.

# Implementation

This section goes into the details about the implementation of such a feature.

## Promises

### Implementation of `await()`

In order to accomplish such a mechanism with the `await()` feature we would need to track a few things. Firstly, when `push(Event e)`
is called we need to immediately create a `Promise` object that is associated with that Event `e`.

When it comes to the calls to `await()` on multiple threads, such as our example with **T1** and **T2**, we will need to, via the `Promise` class somehow access the `Engine` class. This can be accomplished by a private non-static (staic could be done but that might only allow a single Eventy instance then - I would rather keep everything self-contained) field that gets passed into the
`Promise` constructor. This also indicates we should make the constructor private, such that only Eventy's `push(Event e)` can initialize a new `Promise` object. **The reason** for this is because we need to actually queue up a tuple of `(Promise, tid)` in
the Engine such that when the signal handler completes it can send a signal to the matching promises, i.e. the ones where for-every tuple in the queue we have `tuple[0].e == dispatcher.e` (i.e. matching event), then we extratc the `tid` or _thread-id_ and we can
use something like `pthread_sigqueue()` to signal and wake-up the sleeping **T1** and **T2** in their `await()` calls.

### `DispatcherThread` notes

The `DispatcherThread` will also now need access to the Event engine but this should be relatively easy (in the same vain as with the `Promise` private instantiation) - we can simply pass the `this` (referring to the `Engine` object) in.

This dispatcher will have to lock the queue and do the search as discussed earlier.

### Implementation of `then()`

Most important thing here is that the `promise1.then(myEvent2)` refers to `promise1` which was created by `engine.promise(Event e)`, meaning `promise1` has access to the `Engine` (`this`) object. Therefore, when we construct the second _internal_ `promise2`, which
represents the chained Event `myEvent2`, it can also easily be associated.

The need for association is that upon the Dispatcher completing `promise1`, it will make a call to `engine.emit(promise2)`. This whole system then should technically work for infinitely chained events.