---
layout: post
published: true
title:  "RISC-V programming with QEMU"
date:   2025-11-29 10:33:15 -0600
categories: projects RISC-V kernel
excerpt: "How to cross-compile and run a RISC-V program on x86-64 Ubuntu"
---

[QEMU][Qemu Docs] is a machine virtualizer and emulator that can be used to simulate entire computing systems. If I wanted to, say, write an operating system from scratch for a RISC-V machine I would use QEMU to give me that environment, because my laptop is not RISC-V. 

But another cool thing QEMU does is provide an emulated userspace environment to run regular programs in. This means that I can write a RISC-V program and run it on my laptop.

## What is userspace emulation?

A standard program functions within a confined environment established by the operating system, often referred to as “userspace.” In this context, the program must depend on the operating system for various privileged operations, such as I/O or device access. This interaction occurs through system calls, which, unlike typical function calls, initiate a temporary transfer of execution to code running in a higher-privileged CPU state.

QEMU enables the execution of non-native programs in userspace by relaying these system calls from the application to the host environment. As a result, the program operates within an emulated userspace environment without the need for an additional simulated operating system layer.

## Practical Examples

I'll walk through a few demonstrations of writing RISC-V programs and executing them on my x86-64 Ubuntu 24.04 system. First I'll create and run a statically linked binary using assembly, then I'll add libc, and finally, I'll show you how to compile and run a C program.

### Dependencies
1. You'll need to download RISC-V binutils in order to assemble and link non-native RISC-V binaries. It's as easy as one command:
```
$ sudo apt install binutils-riscv64-linux-gnu
```
2. Then you'll need to install the `qemu-user` emulation package. This will update your system to handle non-native binaries automatically, so once installed you should be able to run a RISC-V program by invoking it the same way you do a native program. To install `qemu-user`:
```
$ sudo apt install qemu-user
```

### Freestanding assembly program

In the simplest case, we don't need dynamic linking or anything fancy. In fact I'm not even going to use a programming language. We are just creating a static binary that makes a couple of system calls to print a string.

Here is a RISC-V assembly file `main.s`. All it does is print "Hello, World!" to the console and exit:
```
.data
msg: .asciz "Hello, World!\n"

.text
.globl _start
_start:
    # 'write' syscall
    li a7, 64   # syscall number for write
    li a0, 1    # file descriptor 1 (stdout)
    la a1, msg  # addresss of message string
    li a2, 14   # length of message string
    ecall       # invoke syscall

    # exit syscall
    li a7, 93   # syscall number for exit
    li a0, 0    # exit status code
    ecall       # invoke syscall
```

With the code written, it can now be assembled and linked into an executable:
```
$ riscv64-linux-gnu-as -march=rv64i -mabi=lp64 main.s -o main.o
$ riscv64-linux-gnu-ld main.o
```

We can see the generated binary `a.out` is indeed a RISC-V executable:
```
$ file ./a.out 
./a.out: ELF 64-bit LSB executable, UCB RISC-V, soft-float ABI, version 1 (SYSV), statically linked, not stripped
``` 

And now it can be run in the usual way (`qemu-user` is invoked automatically):
```
$ ./a.out
Hello, World!
```

### Assembly program with glibc

Ok, so that's actually pretty easy, but making system calls can be very tedious, and you don't want to have to reinvent the wheel just to have a `printf()` function available for debugging, so next I'll show you how to link in the Gnu standard C library. 

>_Note: glibc will be dynamically linked, which adds quite a bit of size to our hello world binary. I chose to demonstrate glibc because it is more common, but for a simpler, smaller binary, check out [musl][musl-site], which is more suited for static linking and embedded applications._

To install a RISC-V glibc, run the following command.
```
$ sudo apt install libc6-riscv64-cross
```
This installs the glibc files to `/usr/riscv64-linux-gnu/lib`. QEMU will need to be told where these are so it can load glibc into the execution environment of the binary we're running.

To handle the dynamic linking, you could use the tools already installed, but the invocation commands get pretty involved. The easiest thing for me is to let gcc handle this step. To install the gcc RISC-V cross-compiler, do the following:
```
$ sudo apt install gcc-riscv64-linux-gnu
```

Next, here's another RISC-V assembly file, `main.s`

```
.data
msg: .asciz "Hello, World!\n"
msg2: .asciz "%d bytes written\n"

.text
.globl main
main:
    la a0, msg
    call printf     # we can invoke printf() directly now

    mv a1, a0       # move return value (bytes printed) into arg2
    la a0, msg2
    call printf     # printf("%d bytes written\n", arg2)

    li a0, 0        # exit return code
    call exit
```

Now, instead of assembling and linking in separate steps, we can just invoke `gcc` on the file to get our executable:

```
$ riscv64-linux-gnu-gcc main.s
```

As before, we observe the generated executable is RISC-V (with some extra dynamic linking pizzazz):

```
$ file a.out 
a.out: ELF 64-bit LSB pie executable, UCB RISC-V, RVC, double-float ABI, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux-riscv64-lp64d.so.1, BuildID[sha1]=c7c349188a2ef0647ee409c644957a9fd65a7c2d, for GNU/Linux 4.15.0, not stripped
```

This time, when we run it we need to invoke QEMU directly so we can pass in the glibc directory location:
```
$ qemu-riscv64 -L /usr/riscv64-linux-gnu/ a.out 
Hello, World!
14 bytes written
```

### C program with glibc

To write regular C code, the process is very similar.

First, write your code. I put mine in a file called `main.c`:

```
#include <stdio.h>

int main(int argc, char** argv) {
	printf("Hello, World!\n");
	return 0;
}
```

Now compile with gcc as before:
```
$ riscv64-linux-gnu-gcc main.c
```

It's RISC-V:
```
$ file a.out 
a.out: ELF 64-bit LSB pie executable, UCB RISC-V, RVC, double-float ABI, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux-riscv64-lp64d.so.1, BuildID[sha1]=f2778196306b0a96032641d012a4f8f6db7d03e3, for GNU/Linux 4.15.0, not stripped
```

...and execute:
```
$ qemu-riscv64 -L /usr/riscv64-linux-gnu/ a.out 
Hello, World!
```

## Conclusion

There you have it, userspace RISC-V programming on x86-64 using QEMU. This article was motivated by my interest in RISC-V and a desire to explore it without buying additional hardware. I hope you found it informative!

[Qemu Docs]: https://www.qemu.org/docs/master/
[musl-site]: https://musl.libc.org/