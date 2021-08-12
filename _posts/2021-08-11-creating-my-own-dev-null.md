---
title: "Creating my own /dev/null"
excerpt: "How I implemented a very simple /dev/null for learning about device drivers"
date: 2021-08-11T22:50:30-03:00
categories:
  - Blog
tags:
  - Linux
  - Kernel
---

Recently I took interest in the Linux kernel, and now I'm exploring device drivers. As a very simple learning exercise, I implemented my own `/dev/null`, which I will call `my_null`. I'll show how it is coded, how to compile it, and how to set up its device node (the `/dev/my_null` 'file').

## How `/dev/null` works

The first step was to understand how the original `/dev/null` works. The concept is really easy, as it is just a 'void' where everything that enters it is discarded. It can be tested with something like:

```terminal
$ dd if=some-big-file.iso of=/dev/null status=progress
1519230976 bytes (1,5 GB, 1,4 GiB) copied, 2 s, 760 MB/s
4284224+0 records in
4284224+0 records out
2193522688 bytes (2,2 GB, 2,0 GiB) copied, 2,97851 s, 736 MB/s
```

This could be used to test disk read speed, for example. Very simple stuff.

However, I never asked myself about what is supposed to happen if you _read_ `/dev/null`:

```terminal
$ cat /dev/null
$ # nothing happened
```

Ok, nothing happened... What about checking it with strace?

```terminal
$ strace cat /dev/null 
execve("/bin/cat", ["cat", "/dev/null"], 0x7ffdd3030048 /* 61 vars */) = 0
...
openat(AT_FDCWD, "/dev/null", O_RDONLY) = 3
fstat(3, {st_mode=S_IFCHR|0666, st_rdev=makedev(1, 3), ...}) = 0
fadvise64(3, 0, 0, POSIX_FADV_SEQUENTIAL) = 0
mmap(NULL, 139264, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f77d2052000
read(3, "", 131072)                     = 0
munmap(0x7f77d2052000, 139264)          = 0
...
+++ exited with 0 +++

```

It says it read the contents of the file descriptor associated with `/dev/null`, and it returned 0. From the `man read` pages: "On success, the number of bytes read is returned (zero indicates the end of file)". So, it just returns nothing. I think it is kinda obvious that something that is supposed to be 'null' would return absolutely nothing.

## How to implement my own `/dev/null`

First things first, modules need to be registered in the kernel. This is done via an entry point the kernel calls when a module is loaded. This entry point is specified by `module_init()`, which receives a function that will be called when loading the module. This function should return an integer, indicating if it was successful (return 0), or an error occurred (return a negative error value defined in `errno.h`).

```c
int my_null_major = 0;
int my_null_minor = 0;

static int __init my_null_init(void)
{
    int result = 0;
    dev_t devno = 0;

    if (my_null_major)
    {
        devno = MKDEV(my_null_major, my_null_minor);
        result = register_chrdev_region(devno, 1, "my_null");
    }
    else
    {
        result = alloc_chrdev_region(&devno, my_null_minor, 1, "my_null");
        my_null_major = MAJOR(devno);
    }

    if (result < 0)
    {
        printk(KERN_ERR "my_null: can't get major %d\n", my_null_major);
        goto out;
    }

    result = my_null_setup_device();
    if (result < 0)
    {
        printk(KERN_WARNING "my_null: can't setup device\n");
        goto out;
    }

    printk(KERN_INFO "my_null: setup with major %d and minor %d\n", my_null_major, my_null_minor);

out:
    return result;
}

module_init(my_null_init);
```

The `__init` flag is optional and is used by the compiler to do some optimizations. What this initialization function does is register the (character) device, giving it dynamic major/minor numbers, or using one that was specified. Then, it calls `my_null_setup_device()`:

```c
struct cdev cdev;

static int my_null_setup_device(void)
{
    int err = 0, devno = MKDEV(my_null_major, my_null_minor);

    cdev_init(&cdev, &my_null_fops);
    cdev.owner = THIS_MODULE;
    err = cdev_add(&cdev, devno, 1);

    return err;
}
```

This piece of code set up the cdev structure, which is used by the kernel to track the module, including the operations it is able to do. `cdev_init` registers the operations, which are saved in `cdev`. `cdev_add` tells the kernel the driver is ready to handle operations.

Talking about operations, here is the `file_operations` structure:

```c
static struct file_operations my_null_fops = {
    .owner = THIS_MODULE,
    .read = my_null_read,
    .write = my_null_write,
    .open = my_null_open,
    .release = my_null_release,
};
```

All available file operations and their interface are available in the [kernel source code](https://elixir.bootlin.com/linux/latest/source/include/linux/fs.h). A `null` device driver is simple enough to just have the most basic operations. Also, notice that `owner` is not an operation, but a structure that the kernel uses, and is normally set to `THIS_MODULE`.

Now, to implement the operations. Taking a look at their definitions, we have:

```c
ssize_t (*read) (struct file *, char __user *, size_t, loff_t *);
ssize_t (*write) (struct file *, const char __user *, size_t, loff_t *);
int (*open) (struct inode *, struct file *);
int (*release) (struct inode *, struct file *);
```

If interested, you should read the [Linux Device Drivers book](https://lwn.net/Kernel/LDD3/) to better understand what all these parameters mean. Go to chapter 3 if you are in a hurry, but the whole book is awesome.

The first operation that we will look at is `open`. It is called once when a program wants to take a file descriptor to communicate with the device. So everything that is supposed to be unique to each opener, or any cleaning necessary, should be done here. `my_null` is very simple and does not need to do anything in this case.

`release` is what we know as `close` in userland. It is supposed to undo everything it was done in `open`. Again, does nothing.

In the case of device drivers, `read` is the operation of receiving data from the device, while `write` is the process of sending data to the device. So, when we `cat /dev/null`, we are opening it.

```c
ssize_t my_null_write(struct file *filp, const char __user *buf, size_t count,
                      loff_t *f_pos)
{
    // dev_write returns the number of bytes read.
    // Just say to the user you read everything and return
    *f_pos += count;
    return count;
}
```

The operation is supposed to update the `f_pos` pointer, indicating the new position in the buffer, and return the amount of data read. It might be weird, why not always return the count? But sometimes the operation is done in parts, and only a part of the data is read at one time. This is normally transparent to programs in userland because libraries implement the read operation for the users and take care of guaranteeing the entire operation is completed before returning.

```c
ssize_t my_null_read(struct file *filp, char __user *buf, size_t count,
                     loff_t *f_pos)
{
    // dev_read returns the number of bytes read. 0 indicates the end
    return 0;
}
```

We also saw that reading from `/dev/null` returns 0, indicating nothing more to read. So we just replicate it!

Finally, we need to specify an interface for the module to be unloaded. It is supposed to clean everything `init` did:

```c
static void __exit my_null_exit(void)
{
    dev_t devno = MKDEV(my_null_major, my_null_minor);
    cdev_del(&cdev);
    unregister_chrdev_region(devno, 1);
}

module_exit(my_null_exit);
```

One important thing is that `exit` is only called if `init` ended successfully. 

The complete code can be seen below.

```c
#include <linux/init.h>
#include <linux/module.h>

#include <linux/kdev_t.h> // MAJOR, MKDEV
#include <linux/fs.h>     // register_chrdev_region, alloc_chrdev_region
#include <linux/cdev.h>   // cdev

MODULE_AUTHOR("Victor Colombo");
MODULE_LICENSE("MIT");

int my_null_major = 0;
int my_null_minor = 0;

struct cdev cdev;

ssize_t my_null_read(struct file *filp, char __user *buf, size_t count,
                     loff_t *f_pos)
{
    // dev_read returns the number of bytes read. 0 indicates the end
    return 0;
}

ssize_t my_null_write(struct file *filp, const char __user *buf, size_t count,
                      loff_t *f_pos)
{
    // dev_write returns the number of bytes read.
    // Just say to the user you read everything and return
    *f_pos += count;
    return count;
}

int my_null_open(struct inode *inode, struct file *filp)
{
    return 0;
}

int my_null_release(struct inode *inode, struct file *filp)
{
    return 0;
}

static struct file_operations my_null_fops = {
    .owner = THIS_MODULE,
    .read = my_null_read,
    .write = my_null_write,
    .open = my_null_open,
    .release = my_null_release,
};

static int my_null_setup_device(void)
{
    int err = 0, devno = MKDEV(my_null_major, my_null_minor);

    cdev_init(&cdev, &my_null_fops);
    cdev.owner = THIS_MODULE;
    err = cdev_add(&cdev, devno, 1);

    return err;
}

static int __init my_null_init(void)
{
    int result = 0;
    dev_t devno = 0;

    if (my_null_major)
    {
        devno = MKDEV(my_null_major, my_null_minor);
        result = register_chrdev_region(devno, 1, "my_null");
    }
    else
    {
        result = alloc_chrdev_region(&devno, my_null_minor, 1, "my_null");
        my_null_major = MAJOR(devno);
    }

    if (unlikely(result < 0))
    {
        printk(KERN_ERR "my_null: can't get major %d\n", my_null_major);
        goto out;
    }

    result = my_null_setup_device();
    if (unlikely(result < 0))
    {
        printk(KERN_WARNING "my_null: can't setup device\n");
        goto out;
    }

    printk(KERN_INFO "my_null: setup with major %d and minor %d\n", my_null_major, my_null_minor);

out:
    return result;
}

static void __exit my_null_exit(void)
{
    dev_t devno = MKDEV(my_null_major, my_null_minor);
    cdev_del(&cdev);
    unregister_chrdev_region(devno, 1);
}

module_init(my_null_init);
module_exit(my_null_exit);
```

## Compiling

To compile it, we need the sources of the Linux kernel that will be used to test the driver. I chose kernel 5.10.54. First, put the following Makefile in the same folder as the `my_null` code:

```makefile
obj-m += my_null.o

clean:
    rm -rf *.o *~ core .depend .*.cmd *.ko *.mod.c .tmp_versions Module.symvers modules.order
```

Now, to compile:

```terminal
make -C /path/to/linux-5.10.54/ M=/path/to/my_drivers/my_null_folder/
```

This produces a `my_null.ko` file, which is our device driver compiled.

## Installing the driver

To install the driver, first, run `insmod my_null.ko`:

```terminal
# insmod my_null.ko 
[   45.184357] my_null: loading out-of-tree module taints kernel.
[   45.185256] my_null: module license 'MIT' taints kernel.
[   45.186051] Disabling lock debugging due to kernel taint
[   45.188019] my_null: setup with major 248 and minor 0
```

However, if you take a look at `/dev/`, you see there is nothing new there. That's because it is necessary to create a device node:

```terminal
# mknod /dev/my_null c 248 0
```

where the first argument is the path of the 'file' to be created, 'c' stands for a character device, 248 is the major and 0 is the minor numbers, as was printed when running `insmod` before.

## Profit

Using our new device driver:

```terminal
# wget -O /dev/my_null https://releases.ubuntu.com/21.04/ubuntu-21.04-desktop-amd64.iso
Resolving releases.ubuntu.com (releases.ubuntu.com)... 91.189.91.123, 91.189.91.124, 2001:67c:1562::28, ...
Connecting to releases.ubuntu.com (releases.ubuntu.com)|91.189.91.123|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 2818738176 (2.6G) [application/x-iso9660-image]
Saving to: '/dev/my_null'

/dev/my_null          2%[                    ]  76.13M  6.59MB/s    eta 6m 33s ^C
# cat /dev/my_null 
# 
```

This is another trick that can be done with `/dev/null`, and now with `/dev/my_null` too! It is used to check the internet speed. If we then verify /dev/my_null by `cat`ing it, we see nothing is returned, as expected.