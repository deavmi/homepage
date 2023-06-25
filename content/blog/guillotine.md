---
title: 💤️ Guillotine - 
author: Tristan B. V. Kildaire
date: 2023-03-19
draft: true
---

# Project Guillotine

> Check it out on [DUB](https://code.dlang.org/packages/guillotine) and [GitHub](https://github.com/deavmi/guillotine)

After having started the process of studying for my upcoming Java OCP examination I came across the section on Java's `ExecutorService`. Having already implementing something similar by hand by making use of Java's condition variables and mutexes (a.k.a. thew `synchronized` with its `notify()` and `wait()` method) I came to saw that I had indeed implemented a sort of scheduler that could have tasks submitted into it and upon submission returning a handle to them in the form of a `Future`.

## What is a future?
A future is effectively a handle on a submitted task that continues being run in the background and the future, this *"handle"*, let's you check if it is completed (i.e. what state it is in). Perhaps if it is in a state such as `RUNNING` you can continue doing other work till you come round to check it again.

However useful that is I was more enthralled with the idea of calling `await()` on the `Future` and sleeping my thread till the task completed.

## Executors? D?
There may very well be some library that provides an API similar to that of `ExecutorService` in Java but I decided I would implement it myself to have a library that was nice to user to my needs. It's things like, how does an error get reported back - *"do you re-throw the error or return it in the result when returning from the call to `await()`"* . Another question, *"what does the return value look like?"*.

One of the things that I wanted to get working right away was the ability for someone to provide a task returning any primitive values such as `int`, `double` and so on but also any object type (this was easy enough as you just use `Object` then - the super type). However, with java there is a conversion that can occur easily between primitives like `int` and the OOP-version such as `Integer`. In D, we don't really like generics modelled in the way that Java does. In fact we *can do generics with primitives*.

### Wrapping functions
I therefore went ahead and started working on a template (a form of meta-programming that is like generics but more powerful and can operate on symbols too - not just types). I started with a template which would let me take in any symbol at compile time, evaluate it to check the following:

1. Is it a function?
2. Is the arity of the function 0
	1. What would we call it with? I am not yet going to support this - so far very similar to java's requirement of passing in a `Callable`
3. Does it have a supported return type?
	1. Here I checked for the aforementioned types
	2. There is a difference here compared to Java, I support `void` - it just gets a custom `Empty` value assigned when the future's `await()` call returns

I was able to effectively write code that could generate more code at compile-time on-demand just by a singular parameter and various compile-time traits checking.

## Underlying mechanism
The underlying mechanism being used for the waiting/notification mechanism is, funnily enough, not Hoarian at all. I am actually not using something akin to a `futex` but rather a lock (in D a `Mutex`) and `libsnooze` which currently uses a UNIX pipe to do an I/O wait on (a blocking 1-byte `read()`) and then waking is done by writing to the write end (via a call to `write()` with 1-byte). There's also handling for interruptions here - something I noticed D's runtime doing and took me months to figure out.

> It's good to check `errno()` now and then after checking your return value. Sometimes `-1` is not always a bad thing, maybe you got a signal - `EINTR`

In any case it was D's runtime most likely signalling some signal like `SET-t..._LIMIT_` and a `...+1` version (I cannot recall but presumably for pausing so it could do a GC sweep).

I do plan to move to a Hoarian system in the near future and will most likely make `libsnooze` effectively wrap around D's `core.sync.condition` code which provides it. `libsnooze`  isn't bad at all but there are some interesting by products by doing the `pipe`-trick, such as multiple notifies (say `n`-many) meaning each successive `wait()` would wake `n` times when instead many notifies shouldn't do that if the notified thread was awoken from the first call. This isn't bad, depends on how you use libsnooze but it can cause unneeded cycles. Anyways, I digress.

## Providers
Once you have an `Executor` (as we call it in Guillotine) you can use it to submit tasks as shown below:

```d
// Create an executor with this provider
Executor t = new Executor(provider);

// Submit a few tasks
Future fut1 = t.submitTask!(hi);
```
But you are probably wondering about a few things?

1. Where and how does it executor?
2. And what is that object being passed to the constructor of `Executor`?

Number `2` answers number `1`. The thing being passed is known as a `Provider` and is actually a separate API completely. It defines a task submission system - very general without any knowledge of futures and all that good stuff, the API is very simple - it's shown below! This is what a `Provider` is:

```d
interface Provider
{
	public void consumeTask(Task);
	public void start();
	public void stop();
}
```

### A `Task`?
Well then, if we submit `Task`'s to a `provider`, then the next question is *"what in the hell is a task?"*. That **too** is simple, this is now akin to what Java refers to as a `Runnable` and it is as basic as:

```d
interface Task
{
	public void run();
}
```

### Sequential provider
So far there is only one provider which exists (more are to come) but the so-called `Sequential` provides us with a task submission system which dequeues submitted tasks from a queue and then executes them in serial or *"sequential order"*.

## Putting it all together
Now that we have a `Provider` which the `Executor` can submit tasks too we can look at the worked example below.

First let's import everything we need:
```d
import guillotine.providers.sequential;
import guillotine.executor;
import guillotine.future;
import guillotine.result;
import guillotine.provider;
```

Now let's create our sequential provider, we will start it now but you could start it anytime later (even after creating the `Executor`, just know that then your tasks can only start running after the call to `start()`):
```d
Provider provider = new Sequential();

// Start the provider so it can execute
// submitted tasks
provider.start();
```

Now let's create our executor:
```d
// Create an executor with this provider
Executor t = new Executor(provider);
```

We will now define some worker functions which we will submit as tasks. I have added thread sleeps to simulate work being done to properly test out the `Future`'s awaiting mechanism:
```d
public int hi()
{
    writeln("Let's go hi()!");
    // Pretend to do some work
    Thread.sleep(dur!("seconds")(2));
    return 69;
}

public float hiFloat()
{
    writeln("Let's go hiFloat()!");
    // Pretend to do some work
    Thread.sleep(dur!("seconds")(10));
    return 69.420;
}

public void hiVoid()
{
    writeln("Let's go hiVoid()!");
    // Pretend to do some work
    Thread.sleep(dur!("seconds")(10));
}
```

Now we will submit these tasks:
```d
// Submit a few tasks
Future fut1 = t.submitTask!(hi);
Future fut2 = t.submitTask!(hiFloat);
Future fut3 = t.submitTask!(hiVoid);

// Stops the internal task runner thread
provider.stop();
```

We can now await on the first future as follows:
```d
// Await on the first task
writeln("Fut1 waiting...");
Result res1 = fut1.await();
writeln("Fut1 done with: '", res1.getValue().value.integer, "'");
```

This would print out to the terminal:
```
Fut1 waiting...
Let's go hi()!
Let's go hiFloat()!
Fut1 done with: '69'
```

What happened to the void function? Ah well, recall after submission I made a call to `provider.stop()` and sequential will not accept any tasks after this, now it accepted them all because we made it past the three calls to `submitTask!(alias)()` but it never executed the third because in the time it took us to get from the call to `await()` and the call to `stop()` the sequential provider looped back and saw it was still active and dequeued the second task (the one corresponding to `fut2`) and executed it, it could have actually missed the second one too but it's unlikely.

It is useful to know how to `stop()` a provider as it aids in gracefully shutting down your program and letting any running tasks finish (that it can guarantee). If a running `Task` hangs however, so will the call to `stop()` so always ensure your tasks are coded correctly to handle errors.

---
## What's next?
There are a few things:

1. Upgraded the internal mechanism to use condition variables
	* As I said, it isn't really a huge problem but it could be nice to do things in a more modern way - better for cross platform support too (other than just UNIX)
2. Cancellable `Future`(s)
	* Some systems let you cancel a `Future`, this is done by (on UNIX) sending a signal and having that thread have a custom handler for it set
	* This handler would set the state of the `Future` to `State.CANCELLED` and wake up any `await()` calls with an exception explaining so
	* As for the underlying `Provider` implementation, well, I would have to investigate
		* Most likely a method such as `onInterrupted()` could be used or something (and registered with the process's (well the TID, the thread process) signal table)

---

## That's it!

I hope you have some fun using it as an executor service API makes coding a lot of network-based applications very much more manageable (it's actually why I created Guillotine was to deal with an upcoming project of mine where I need the concept of a `Task` and its progress `Future`).