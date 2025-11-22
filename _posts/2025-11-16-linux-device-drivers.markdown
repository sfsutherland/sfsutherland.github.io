---
layout: post
published: false
title:  "Kernel module: hello world"
date:   2025-11-16 12:33:15 -0600
categories: projects linux kernel
excerpt: "Here's how to compile and load a simple kernel module against a running Ubuntu OS"
---

There are a few ways to get a kernel module up and running locally. The simplest is to build and link against the running kernel, if your distro supports it. I'm using Ubuntu 24.04, which does. Here's how to do it.

## Create a work directory with an `output` folder

Make a new directory and inside of it create an `output/` directory to hold the build artifacts.

```
$ mkdir -p my_module/output
$ cd my_module
```

The `output/` directory isn't strictly necessary, but running the `make` command later will automatically create a bunch of build files in whatever directory you run it from. I like to have this stuff contained in its own directory instead of cluttering up my code files, so I created an `output/` directory and added `MO=` flags to the build commands below.

## Create hello.c

```
#include <linux/init.h>
#include <linux/module.h>
MODULE_LICENSE("Dual BSD/GPL");

static int hello_init(void)
{
	printk(KERN_ALERT "Hello, world\n");
	return 0;
}

static void hello_exit(void)
{
	printk(KERN_ALERT "Goodbye, cruel world\n");
}

module_init(hello_init);
module_exit(hello_exit);

```



## Create Makefile

```
ifneq ($(KERNELRELEASE),)
# kbuild part of makefile
obj-m  := hello.o

else
# normal makefile
KDIR ?= /lib/modules/`uname -r`/build

default:
	$(MAKE) -C $(KDIR) M=$$PWD MO=$$PWD/output

clean:
	$(MAKE) -C $(KDIR) M=$$PWD MO=$$PWD/output clean
endif

```

## Compile

Run the following from the directory where both above files reside:

```
make -C /lib/modules/`uname -r`/build M=$PWD MO=$PWD/output
```

## References

- [Official kernel docs on building external modules](https://docs.kernel.org/kbuild/modules.html).
- [Linux Device Drivers, 3rd Edition](https://lwn.net/Kernel/LDD3/)


[Official docs]: https://docs.kernel.org/kbuild/modules.html