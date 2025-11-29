---
layout: post
published: false
title:  "RISC-V programming with qemu"
date:   2025-11-29 10:33:15 -0600
categories: projects linux kernel
excerpt: "How to cross-compile and run a RISC-V program on x86-64 Ubuntu"
---

[Qemu][Qemu Docs] is a machine virtualizer and emulator that can be used to simulate entire computing systems. If I wanted to, say, write an OS from scratch for a RISC-V machine I would use `qemu` to give me that environment, because my laptop is not RISC-V. 

But another cool thing `qemu` does is provide an emulated userspace environment to run regular programs in. This means that I can write a RISC-V program and run it on my laptop.

## Freestanding assembly program

In the simplest case, we don't need dynamic linking or anything fancy. In fact I'm not even going to use a programming language. We are just creating a static binary that gets run in an emulated RISC-V userspace.


## C program with glibc

This example shows how to create a dynamically linked RISC-V executable and run it on x86-64


## Assembly program with glibc

Sometimes you want to write assembly, but also not have to invent your own `printf`. Okay, maybe that's just something I'm doing, but here's how to link against glibc and call functions in it from assembly.



[Qemu Docs]: https://www.qemu.org/docs/master/