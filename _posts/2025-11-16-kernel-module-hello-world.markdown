---
layout: post
published: true
title:  "Kernel module: hello world"
date:   2025-11-16 12:33:15 -0600
categories: projects linux kernel
excerpt: "A basic guide showing how to compile and load a kernel module against a running Ubuntu OS"
---

There are a few ways to get a kernel module up and running locally. The simplest is to build and link against the running kernel. Here's how to do it.

_Note: The following steps were done on a system running Ubuntu 24.04_

## 1. New directory with an `output` folder

Make a new directory and inside of it create an `output/` directory to hold the build artifacts.

```
$ mkdir -p my_module/output
$ cd my_module
```

The `output/` directory isn't strictly necessary, but running the `make` command later will automatically create a bunch of build files in whatever directory you run it from. I like to have this stuff contained in its own directory instead of cluttering up my code files, so I created an `output/` directory and added `MO=$$PWD/output` flags to the build commands below.

## 2. Create hello.c

In your project directory create a `hello.c` file with the following contents.

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



## 3. Create Makefile

Now create a file called `Makefile` with the following contents.

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

## 4. Compile

To build, run this command.

```
$ make
```

## 5. Load the module

In your `output/` directory there should now be a `hello.ko` file. This is the loadable kernel module. Load the module into the running kernel with this command:

```
$ sudo insmod ./output/hello.ko
```

## 6. View `printk` output

The output of the `printk` shows up in the ring buffer.

```
$ sudo dmesg | tail

...
[27784.684238] hello: loading out-of-tree module taints kernel.
[27784.684245] hello: module verification failed: signature and/or required key missing - tainting kernel
[27784.685898] Hello, world

```
And there's our `printk` string. Linux doesn't like that our module isn't signed, but it still allows it. To learn how to sign modules, check out the [official docs][Signing modules].

Note: You can also view the ring buffer with `journalctl -f`

## 7. Unload module

To remove the module, you need to know its name. Ours is just `hello`. To verify this, you can view the output of `lsmod`, which lists the names of the currently loaded modules.

```
$ sudo rmmod hello
```


## Conclusion

Hopefully you found this a useful starting point for getting a module up and running. To dive deeper into modules, checkout the references below.

## References


- [Official kernel docs on building external modules](https://docs.kernel.org/kbuild/modules.html).
- [Linux Device Drivers, 3rd Edition](https://lwn.net/Kernel/LDD3/)


[Official docs]: https://docs.kernel.org/kbuild/modules.html
[Signing modules]: https://docs.kernel.org/admin-guide/module-signing.html