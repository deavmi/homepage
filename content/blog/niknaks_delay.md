---
title: "Addition to Niknaks: A delay mechanism for programatic retries"
author: Tristan B. Velloza Kildaire
date: 2025-03-01
draft: true
---

## Delay mechanism

One of the things I decided to put together was a programming
structure that I thought would be usable in a scenario as follows:

> You have run some task on some remote server and now you want to
regularly check in with it and see when a reply is sent back that
satisfies some requirement. You _also_ can't do this forever so you
want to set an interval for how frequently you re-check and then also
a total timeout time which will be your deadline - when you exceed
it then you stop checking.

This mechanism is called the `Delay` and can be found in the
`niknaks.mechanisms` package.

The constructor appears as follows:

```d
/** 
 * Constructs a new delay mechanism
 * with the given delegate to call
 * in order to determine the verdict,
 * an interval to call it at and the
 * total timeout
 *
 * Params:
 *   verdictProvider = the provider of the verdicts
 *   interval = thje interval to retry at
 *   timeout = the timeout
 */
this(VerdictProviderDelegate verdictProvider, Duration interval, Duration timeout)
{
    this.verdictProvider = verdictProvider;
    this.interval = interval;
    this.timeout = timeout;
}
```

Therefore you can set those two parameters mentioned earlier. The first
parameter is the "check" that is to be periodically called. The delay
system will return successfully if at some point this `VerdictProviderDelegate`
returns a `true` _within_ the timeout window of `timeout`. If you
keep getting `false` returned and go over the `timeout` period then
a `DelayTimeoutException` is thrown.

### Example code

Here is an example where my verdict function (technically a delegate in
the case of a D unittest) which refers to a variable `cnt`. This example
illustrates the multiple retry calls to said function due to it only
returning a `true` verdict on the _second_ call.


```d
/**
 * Tests out the delay mechanism
 * with a verdict provider (as a
 * delegate) which is only true
 * on the second call
 */
unittest
{
    int cnt = 0;
    bool happensLater()
    {
        cnt++;
        if(cnt == 2)
        {
            return true;
        }
        else
        {
            return false;
        }
    }

    Delay delay = new Delay(&happensLater, dur!("seconds")(1), dur!("seconds")(1));

    try
    {
        delay.go();
        assert(true);
    }
    catch(DelayTimeoutException e)
    {
        assert(false);
    }
}
```