---
title: "Qix: Simple waitable-queue management"
description: "An introduction to my new waitable-queue management library called Qix"
author: Tristan B. V. Kildaire
date: 2025-06-07
draft: true
---

## Introduction

As part of an still-undisclosed project of mine that I have been working on for the
past 2 months or so is the requirement for some sort of protocol-agnostic request-response
matching system.

As such a system is normally rather generic it could potentially be used in other projects
of mine in the future, where a similar use case is required. _Or_ you could even use it! It
is for this reason that I decided to redesign what was already ongoing as a redesign within
my **older** [tristanable](../projects/tristanable/) project.

This entailed stripping out any of the network aspects of tristanable, namely:

1. The `Socket` references
2. The message encoding/decoding routines

Removing 1 and 2 was done so as to result in a library solely focused on the actual
matching engine. What would be left to the user would be to design a wire format for
the actual messages that might be sent over something like a `Socket` - just for example.

