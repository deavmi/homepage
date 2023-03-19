---
title: libsnooze - a wait/notify mechanism for D
author: Tristan B. V. Kildaire
date: 2023-03-19
draft: false
---

# What is libsnooze?

I decided I wanted a mechanism similiar to that of Java's wait/notify mechanism whereby a thread can be put to sleep and then another can wake it up when it sees fit. The waking up is normally done on some condition on the calling thread - it's a rather common need in multi-threaded programs that are cooperating on shared data structures.

Now, one may think:

> "Why not just have a thread spin in a while loop till a boolean is set to true?"

Asking this is not a dumb question at all actually and is what is known as the "spinning" implementation - it's simple and it works! **But there's a catch...**

## An example of spinclass 🚴‍♂️️

The problem can be illustrated in a simple manner, but first let's setup an example. What I first will define is a `Thread` object (using D) which will have a `worker()` function which, when called will loop forever until the variable `ready` (which both the _main thread_ and the thread `worker()` is running on, have access to).

```d
bool ready = false;

public class MyThread : Thread
{
    this()
    {
        super(&worker);
    }

    private void worker()
    {
        while(!ready)
        {
            // Spin till ready
        }

        writeln("I am now ready!");
    }
}
```

Now, I will spawn the second thread which will start to loop indefinately in `worker()`:

```d
MyThread thread2 = new MyThread();
thread2.start();
```

The next step is now is to update the value of `ready` from `false` to `true` such that we can get the other thread to "wake up" and print the _"I am now ready!"_ message. let's say we wait 5 seconds and then we wake it up:

```d
// This sleeps the CALLING thread - so the main thread in our case
Thread.sleep(dur!("seconds")(5));
```

### Results

Well, after 5 seconds it is obvious the thread will wake up and we'll get the output:

```
I am now ready!
```

## Issues with joining spinclass

Barr the obvious fact that I don't want to be caught in 4K wearing tights for a spinclass there is a problem with this implementation. But don't fret it isn't one to do with the implementation's _idea_ so to speak. It is a logical implementation and should work (and does), the problem is with _performance_ of the _rest_ of your machine's processes and heat generation and energy usage (the latter of which I only became aware of because certain CPUs can actually powersave in certain conditions).

Woah, that's a lot - let's break this down first of all and start off with a description of a few things which one must understand about hardware and operrating systemn software.

### Scheduling and the run-queue

The kernel is what schedules processes. Every certain number of milliseconds an interrupt fires (due to a hardware timer, like an eggtimer on repeat), this is then picked up at the end of processing a single instruction, if the interrupt register is non-zero (for explanations sake), then the number is used as an offset into a table of function handlers for that interrupt type. The specif interrupt here is the system timer which tyhe kernel has a handler for.

This handler is used to save the context of the current process, its open files, memory map, instruction pinter (where we were in the program) etc.. The kernel then moves this process to the back of the run queue and looks at the hwead of the queue and switches the context for that process in. This process repeats.

Now, a program that never sleeps or does any I/O will remain in the runqueue and is what I call _"highly schedulable"_ as it can always be immediately run. If we look at the second thread (which in UNIX is just a process with certain flags that make it considred what is seen as a _"thread"_) it cannot sleep - why is that? Well, there is no I/O system calls such as those which would place it to sleep for perhaps reading a file of whioch bytes have not been made avialbale yet (the I/o request yet to be fulfilled). It also never calls `sleep(5)` like our main thread does. **It can _only_ remain in the run queue** (unless it crashed which it won't).

#### Problem 1: Time quantum starving

If we have say now 4 processes and give each process 1 seconds of run time and then perform a context switch then in the period of 4 seconds we would have scheduled each of the 4 processes once. If each process completes its job within 2 seconds then it means we could need two periods of 4 (so 8 seconds) of time to do so.

Now imagine we add a 5 process here, this will mean we need about (worst case) 10 seconds to get the real work of the 4 other processes done. _"real work"_ - what do I mean? Well, let's say process 5 _was_ our second thread we made with the spinning in it. It would be scheuled twice with a time slice of 1 second but you would agree it is _working_ but it isn't getting its intended job really done (which is to print the the comamnd line "I am now ready") but rather it is "waiting" for a condition till it can do it's _"real work"_.

The idea I am trying to drive home here is its meaningless work is making the other provesses take lomnger to complete their full job (`10` seconds in total is greater than `8` secondsd in total).

_What if we could change that?_ so it didn;'t stay in the runqueue till it was ready to do work?

In fact, you will see most programs tend to not cap your run queue on a normal desktop if you open up `htop` for example. The times it will would be strenuous tasks like compilations etc, those however, are doing _"real work"_ - they just have hardly any I/O (and they definately wouldn't sleep - _why would a compiler sleep_).

#### Problem 2: Drawing higher current

I'm no engineer but I know the more components you have turned on the more a single conductor connecting all of them (the power supply) will draw more. It's like the ~~welfare state the more you give the more they want back~~ idea of having a pressized pipe, the more taps you open the more water can flow into it - or atleast that is the analogy used - I don't know I never took physics. This section only applies to CPUs that actually support this feature so _Problem 1_ is enough to justify why _spinnning_ is a bad idea but this is another reason too.

CPUs such as those that are `x86` support an instruction known as `HLT` which, if called, will disable certain components in the CPU till the next interrupt occurs (waking it from this state), this can save a lot of power in the long run. Now, the way the Linux deals with this is that if there isn't anything at this moment that can be scheduled then it runs a kernwel task that calls `HLT` in a loop. THis will momentrarily power save until an interrupt goes off - one of which is the task switching interrupt, the kernel can wake up for a burst (the CPU hardware turns out and resumes drawing more current) and the kernel can check _"Are there any tasks availabl;e to schedule?"_, if not it sleeps again. This means you do more sleeping than waking and you save power doing this. 

So you can see why keeping the user process runqueue as empty as possible - when it makes sense to, has more than one benefit.