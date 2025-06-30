---
title: "Niknaks updates"
author: Tristan B. Velloza Kildaire
date: 2025-03-01
draft: true
---


// TODO: Fix date

# Introduction

Just thought I would describe what the `niknaks` project is. Firstly, it
isn't an application-side library, something that relates to any one _"idea"_
but rather just a collection of various utilities and re-usable datatypes.

It orirignally started when I begun work on a project of mine where I
was seeminly requiring common routines like:

* Array element presence checking
* Endianness conversions
    * And support for widths of 1 to 8 bytes

After some time I started working on other projects of mine whereby I needed
some functional constructs like _predicates_ and _optionals_, therefore I
went ahead and implemented those as common types accessible to my applications,
all of this being templatised of course.

As time went on I had added things like a delay mechanism, debugging tools,
prompting mechanism (think of CLI usage), configuration utility comprising
of configuration entries and a registry to manage them, a cache map implementation. (TODO: Add tree type)


Therefore I was able to collect all of these commonly used routines, types
and mechanisms into a common library that I called `niknaks`, all licensed
under the LGPL 3.0.

# Updates

It is now time to talk about the various additions I have made to this library
over the last few months because I have had great fun implementing the various
routines I need for other projects of mine **and** making them available to the
greater D community.

Below are just _some_ of the pull requests (some earlier ones were left out as
I just grabbed the ones which were already open in my browser's tabs).

| Changeset                                                | Link                                      |
|----------------------------------------------------------|-------------------------------------------|
| ⚡ Feature: Generic configuration mechanism              | https://github.com/deavmi/niknaks/pull/18 |
| ⚡ Feature: More array operations                        | https://github.com/deavmi/niknaks/pull/20 |
| ⚡ Feature: Improved prompting framework                 | https://github.com/deavmi/niknaks/pull/21 |
| ⚡ Feature: Add opIndex support to CacheMap              | https://github.com/deavmi/niknaks/pull/19 |
| ⚡ Feature: Prompting framework                          | https://github.com/deavmi/niknaks/pull/16 |
| ⚡ Feature: Buffer views                                 | https://github.com/deavmi/niknaks/pull/22 |
| ⚡ Feature: Generic tree and visitation framework        | https://github.com/deavmi/niknaks/pull/17 |
| ⚡ Feature: Insert at                                    | https://github.com/deavmi/niknaks/pull/23 |
| ⚡ Feature: Result type                                  | https://github.com/deavmi/niknaks/pull/25 |

# Example usage

I have put together several blog posts on how to use each of these
new features. You can use the table of contents below to take a
look at each:

1. [Using the new `CacheMap` type](../blog/niknaks_cachemap)
2. [Writing interactive prompts, _effortlessly_](../blog/niknaks_prompter)
3. [Managing configurations with the config engine](../blog/niknaks_config)
4. [Predicates and optionals](../blog/niknaks_predicates)
5. [Bit and byte manipulation routines](../blog/niknaks_bitmanip)
6. [Delay mechanism](../blog/niknaks_delay)
7. [Using the Result type](../blog/niknaks_result)


TODO: Add links to last few posts here