---
permalink: /documents/linux/device-driver/
title: "Device driver"
excerpt: "how to flow device-dirver. device, drivers, device driver, device node ..."
toc: true
---

## info

A Linux device driver is software that is responsible for communication between the Linux kernel and hardware.<br>
It provides an interface between <span style="{{ site.code }}">kernel space</span> and <span style="{{ site.code }}">user space</span> ,<br>
and it interacts with specific hardware devices to make them available to the operating system.<br>

How device drivers work and how they are created/added.<br>
Device drivers are software for using hardware in a kernel or user space.<br>

This is a graph that is how a device is being used in linux system.
```
|  User Space  |                Kernel Space                 |              Hardware Space               |
.              .                                             .                                           .
.              .                                             .                                           .
.               \                                            .                                +----------+
.              +-+                              +------------+      +------------+            |          |
+------------+ | |                              |            |      |            | <--------> | Device 1 |
| user app 1 | | |-> System call/Device call -> |            |      |            |            |          |
+------------+ | |                              |            |      |            |            +----------+
               | |                              |            |      |            |            +----------+
+------------+ | |                              |            | <--> |   Device   |  hardware  |          |
| user app 2 | | |-> System call/Device call -> | Bus Driver | <--> |            |  protocol  | Device 1 |
+------------+ | |                              |            | <--> | Controller |            |          |
               | |                              |            |      |            |            +----------+
+------------+ | |                              |            |      |            |            +----------+
| user app 3 | | |-> System call/Device call -> |            |      |            |            |          |
+------------+ | |                              |            |      |            | <--------> | Device 1 |
.              +-+                              +------------+      +------------+            |          |
.     ...       / ` System call                              .                                +----------+
.              .     interface                               .                                           .
.              .                                             .                                           .
|  User Space  |                Kernel Space                 |              Hardware Space               |

```
Device drivers exist in kernel space.<br>
It takes system calls from the user space and connects the bus driver & the device controller.<br>
Or, it means that the bus driver is included.<br>
<br><br>

There are three main types of device drivers.

### char device driver

Deals with sequentiality of data.<br>

### block device driver

Managed by the file system inside the kernel and with internal buffers.<br>

### network device driver

Provides the ability to connect with the network layer to send and receive network packets over network communication.<br>

### node

UNIX-based device drivers are in file format.<br>
Typically present in the <span style="{{ site.code }}">/dev</span> path.<br>
Device files can be created with the <span style="{{ site.code }}">mknod</span> command.
```
$ mknod /dev/{name} {type} {major} {minor}
```
<br>

type has block, character and FIFO device, see <span style="{{ site.code }}">mknod --help</span> for more information.<br>
As you can see from the command format, the device drivers have major, minor numbers,<br>
which are mentioned again in the driver code making section.<br>

### udev

It is Daemon that created for the purpose of automatically generating device files.<br>
Dynamic Device File Allocation.<br>
Provide API to gather information from system devices (sysfs)<br>

**udev-rules**<br>
<span style="{{ site.code }}">uevent</span> occurs when the device is connected,<br>
and the udev daemon automatically creates the device driver.<br>

You can add drivers to dynamically assign by creating a rules file.<br>
The path is <span style="{{ site.code }}">/etc/udev/rules.d</span> . and rules name is <span style="{{ site.code }}">##-name.rules</span> , <br>
The preceding number refers to the order in which the rules file is called.<br>

for example,<br>
50-uart.rules 70-canfd.rules When there are two files, uart rules apply first.<br>


## common

This is the default example driver.<br>
<U>It is not a code that interacts with the actual hardware.</U><br>
The main focus is on describing the basic format.<br>

refers to: [one](https://github.com/how2flow/mydriver)<br>
It is described with the Git project.
```
$ git clone https://github.com/how2flow/mydriver
```
```
├── Kconfig
├── LICENSE
├── Makefile
└── README.md
```

The license for the kernel is a <span style="{{ site.code }}">GPL</span> <span style="{{ site.code }}">GPL-2</span> license.<br>
The same goes for device drivers.<br>
Any code that operates within the kernel should be published as an open source; this is not limited to device drivers.<br>

The code first writes the license and copyright syntax.<br>
<span style="{{ site.code }}">LICENSE</span>
```
// SPDX-License-Identifier: GPL-2.0
/*
 * Copyright (c) year name <e-mail>
 */
```
<br>

Add driver-related content from <span style="{{ site.code }}">Makefile</span> that exists in the same path of the driver you added.<br>
In some cases, you may need to add or modify the <span style="{{ site.code }}">Kconfig</span> .<br>

<span style="{{ site.code }}">Kconfig</span>
```
+#
+# my custom drivers configuration
+#
+
+menu "My devices"
+
+config MYDEV
+       tristate "My drivers support"
+       help
+         It's for practice.
+          Usually, you set it up as a module, build it, and install it.
+
+endmenu
```
<br>

<span style="{{ site.code }}">Makefile</span>
```
+# SPDX-License-Identifier: GPL-2.0
+#
+# Makefile for the my drivers
+# Copyright (C) Steve Jeong <steve@how2flow.net>
+
+obj-$(CONFIG_MYDEV) +=
```
<br>

In kernel source,<br>
<br>
<span style="{{ site.code }}">drivers/Kconfig</span>
```
diff --git a/drivers/Kconfig b/drivers/Kconfig
index ...
--- a/drivers/Kconfig
+++ b/drivers/Kconfig
@@ -244,4 +244,6 @@ ..

+source "drivers/mydriver/Kconfig"
+
 endmenu
```
<br>

<span style="{{ site.code }}">drivers/Makefile</span>
```
diff --git a/drivers/Makefile b/drivers/Makefile
index ...
--- a/drivers/Makefile
+++ b/drivers/Makefile
@@ -193,3 +193,4 @@ ..

+obj-$(CONFIG_MYDEV) += mydriver/
```

\+ add <span style="{{ site.code }}">CONFIG_MYDEV=m</span> in your defconfig.<br>

## char device driver

### prepare

Create a virtual character device driver.<br>
file name is <span style="{{ site.code }}">my_char.c</span>
```
--- /dev/null
+++ b/my_char.c
@@ -0,0 +1,75 @@
+// SPDX-License-Identifier: GPL-2.0
+/*
+ * Copyright (C) Steve Jeong <steve@how2flow.net>
+ */re--
+
+#include <linux/module.h>
+#include <linux/fs.h>
+#include <linux/init.h>
+#include <linux/cdev.h>
+
+#define DRIVER_NAME "my_char"
+#define NUM_DEVICES 1
+
+static dev_t device_num;
+static struct cdev cdev;
+
+static int my_char_open(struct inode *inode, struct file *file)
+{
+       printk(KERN_INFO "my_char_open\n");
+       return 0;
+}
+
+static int my_char_release(struct inode *inode, struct file *file)
+{
+       printk(KERN_INFO "my_char release\n");
+       return 0;
+}
+
+static struct file_operations my_char_fops = {
+       .owner = THIS_MODULE,
+       .open = my_char_open,
+       .release = my_char_release,
+};
+
+static int __init my_char_init(void)
+{
+       int ret;
+
+       // register character device
+       ret = alloc_chrdev_region(&device_num, 0, NUM_DEVICES, DRIVER_NAME);
+       if (ret < 0) {
+               printk(KERN_ERR "Failed to allocate device number\n");
+               return ret;
+       }
+
+       // character device init
+       cdev_init(&cdev, &my_char_fops);
+       cdev.owner = THIS_MODULE;
+
+       // add character device (cdev_init() -> cdev_add())
+       ret = cdev_add(&cdev, device_num, NUM_DEVICES);
+       if (ret < 0) {
+               unregister_chrdev_region(device_num, NUM_DEVICES);
+               printk(KERN_ERR "Failed to add device to the system\n");
+               return ret;
+       }
+
+       printk(KERN_INFO "my_char driver initialized\n");
+       return 0;
+}
+
+static void __exit my_char_exit(void)
+{
+       // unregister character device (cdev_del() -> unregister_chrdev_region())
+       cdev_del(&cdev);
+       unregister_chrdev_region(device_num, NUM_DEVICES);
+       printk(KERN_INFO "my_char driver removed\n");
+}
+
+module_init(my_char_init);
+module_exit(my_char_exit);
+
+MODULE_LICENSE("GPL v2");
+MODULE_AUTHOR("Steve Jeong");
+MODULE_DESCRIPTION("A simple character device driver example");
```
<br>

Minimal headers to define character devices
```
#include <linux/module.h>
#include <linux/fs.h>
#include <linux/init.h>
#include <linux/cdev.h>
```
<br>

Elements for Character Device Initialization
You will receive or assign a hardware number through <B>device_num</B>.
```
#define DRIVER_NAME "my_char"
#define NUM_DEVICES 1

static dev_t device_num;
static struct cdev cdev;
```
<br>

A device driver is what makes a device interactable on a user interface.<br>
Therefore, the device must exist as a file in the user interface.<br>

Define <span style="{{ site.code }}">file_operations</span> to exist as a file.<br>
Most basic implementation of open/release.
```
static int my_char_open(struct inode *inode, struct file *file)
{
       printk(KERN_INFO "my_char_open\n");
       return 0;
}

static int my_char_release(struct inode *inode, struct file *file)
{
       printk(KERN_INFO "my_char release\n");
       return 0;
}

static struct file_operations my_char_fops = {
       .owner = THIS_MODULE,
       .open = my_char_open,
       .release = my_char_release,
};
```
<br>

Implementing module init/exit<br>
with macro <span style="{{ site.code }}">module_init</span> and <span style="{{ site.code }}">module_exit</span> .
```
static int __init my_char_init(void)
{
       int ret;

       // register character device
       ret = alloc_chrdev_region(&device_num, 0, NUM_DEVICES, DRIVER_NAME);
       if (ret < 0) {
               printk(KERN_ERR "Failed to allocate device number\n");
               return ret;
       }

       // character device init
       cdev_init(&cdev, &my_char_fops);
       cdev.owner = THIS_MODULE;

       // add character device (cdev_init() -> cdev_add())
       ret = cdev_add(&cdev, device_num, NUM_DEVICES);
       if (ret < 0) {
               unregister_chrdev_region(device_num, NUM_DEVICES);
               printk(KERN_ERR "Failed to add device to the system\n");
               return ret;
       }

       printk(KERN_INFO "my_char driver initialized\n");
       return 0;
}

static void __exit my_char_exit(void)
{
       // unregister character device (cdev_del() -> unregister_chrdev_region())
       cdev_del(&cdev);
       unregister_chrdev_region(device_num, NUM_DEVICES);
       printk(KERN_INFO "my_char driver removed\n");
}

module_init(my_char_init);
module_exit(my_char_exit);
```
<br>

Add information about the module.
```
MODULE_LICENSE("GPL v2");
MODULE_AUTHOR("Steve Jeong");
MODULE_DESCRIPTION("A simple character device driver example");
```

After kernel build & install,<br>
check kernel message with <span style="{{ site.code }}">dmesg</span> .
```
$ reboot
$ modprobe my_char
$ lsmod | grep my_char
my_char                16384  0
```
```
$ dmesg
[   ...] my_char driver initialized
```
<br>

You can't find a device file for interaction.<br>
First, let's make it manually
```
$ cat /proc/devices | grep my_char
236 my_char
```
```
$ sudo mknod /dev/my_char c 236 0
$ ls -l /dev/my_char
crw-rw---- 1 root root 236, 0 Feb 14 08:07 /dev/my_char
```
<br>

However, you cannot create nodes manually every time..<br>
Let's make a character device node.
```
diff --git a/my_char.c b/my_char.c
index ...
--- a/my_char.c
+++ b/my_char.c
@@ -7,12 +7,14 @@
 #include <linux/fs.h>
 #include <linux/init.h>
 #include <linux/cdev.h>
+#include <linux/device.h>

 #define DRIVER_NAME "my_char"
 #define NUM_DEVICES 1

 static dev_t device_num;
 static struct cdev cdev;
+static struct class *my_char_class;

 static int my_char_open(struct inode *inode, struct file *file)
 {
@@ -55,6 +57,18 @@ static int __init my_char_init(void)
                return ret;
        }

+       // create class
+       my_char_class = class_create(THIS_MODULE, DRIVER_NAME);
+       if (IS_ERR(my_char_class)) {
+               cdev_del(&cdev); // because it's after cdev_add
+               unregister_chrdev_region(device_num, NUM_DEVICES);
+               printk(KERN_ERR "Failed to create class\n");
+               return PTR_ERR(my_char_class);
+       }
+
+       // create device (create_class() -> device_create())
+       device_create(my_char_class, NULL, device_num, NULL, DRIVER_NAME);
+
        printk(KERN_INFO "my_char driver initialized\n");
        return 0;
 }

 static void __exit my_char_exit(void)
 {
+       // destroy device and class
+       device_destroy(my_char_class, device_num);
+       class_destroy(my_char_class);
+
        // unregister character device (cdev_del() -> unregister_chrdev_region())
        cdev_del(&cdev);
        unregister_chrdev_region(device_num, NUM_DEVICES);
```
First, add a class structure.<br>
And create a node under <span style="{{ site.code }}">/sys/class</span> with the <span style="{{ site.code }}">class_create</span> function.<br>
Then, call the <span style="{{ site.code }}">device_create</span> function to create a character device under <span style="{{ site.code }}">/dev</span> .<br>
<U>If you've created it, you have to destroy it</U>.<br>
The <span style="{{ site.code }}">linux/device.h</span> header is added for this operation.


After kernel build & install,<br>
Check the changes.
```
$ reboot
$ modprobe my_char
$ ls -l /dev/my_char
crw-rw---- 1 root root 236, 0 Feb 14 08:10 /dev/my_char
```
If you want to create a node automatically, write <span style="{{ site.code }}">udev rules</span> .<br>
<br>

I will add the read/write action to try the interaction.

### ops: read/write

You must add the function pointers <span style="{{ site.code }}">.read</span> and <span style="{{ site.code }}">.write</span> in file_operations<br>
to add read/write operations.
```
diff --git a/my_char.c b/my_char.c
index ...
--- a/my_char.c
+++ b/my_char.c
@@ -8,13 +8,16 @@
 #include <linux/init.h>
 #include <linux/cdev.h>
 #include <linux/device.h>
+#include <linux/uaccess.h>

 #define DRIVER_NAME "my_char"
 #define NUM_DEVICES 1
+#define BUF_SIZE 1024

 static dev_t device_num;
 static struct cdev cdev;
 static struct class *my_char_class;
+static char device_buffer[BUF_SIZE];

 static int my_char_open(struct inode *inode, struct file *file)
 {
@@ -28,10 +31,32 @@ static int my_char_release(struct inode *inode, struct file *file)
        return 0;
 }

+static ssize_t my_char_read(struct file *file, char __user *user_buffer, size_t count, loff_t *offset)
+{
+       // device -> user
+       if (copy_to_user(user_buffer, device_buffer, count) != 0) {
+               return -EFAULT;
+       }
+
+       return count;
+}
+
+static ssize_t my_char_write(struct file *file, const char __user *user_buffer, size_t count, loff_t *offset)
+{
+       // user -> device
+       if (copy_from_user(device_buffer, user_buffer, count) != 0) {
+               return -EFAULT;
+       }
+
+       return count;
+}
+
 static struct file_operations my_char_fops = {
        .owner = THIS_MODULE,
        .open = my_char_open,
        .release = my_char_release,
+       .read = my_char_read,
+       .write = my_char_write,
 };
```
Add buffers for data exchange.<br>
The buffer allows you to read or write data from the userspace.<br>

With <span style="{{ site.code }}">copy_to_user(void __user \* to, const void \* from, unsigned long n)</span> and <span style="{{ site.code }}">copy_from_user(void __user \* to, const void \* from, unsigned long n)</span>. <br>
<br>
<B>Why does it use these two APIs instead of copying directly?</B><br>
<U>This is because,<br>
The memory address of the user area and the memory address of the kernel area<br>
are different from each other.</U><br>
<br>
The <span style="{{ site.code }}">linux/uaccess.h</span> header is added for this operation.
<details>
  <summary> (plus... )</summary>
  <p>
  It is parts of <span style="{{ site.code }}">copy_to_user</span> & <span style="{{ site.code }}">copy_from_user</span>
  <pre><code>
  static __always_inline __must_check unsigned long
  __copy_to_user(void __user *to, const void *from, unsigned long n)
  {
      might_fault();
      if (should_fail_usercopy())
          return n;
      instrument_copy_to_user(to, from, n);
      check_object_size(from, n, true);
      return raw_copy_to_user(to, from, n);
  }
  ...
  static __always_inline __must_check unsigned long
  __copy_from_user(void *to, const void __user *from, unsigned long n)
  {
      might_fault();
      if (should_fail_usercopy())
          return n;
      instrument_copy_from_user(to, from, n);
      check_object_size(to, n, false);
      return raw_copy_from_user(to, from, n);
  }
  </code></pre>
  <span style="{{ site.code }}">copy_to_user</span> 's <span style="{{ site.code }}">from</span> must not be less than its <span style="{{ site.code }}">n</span> ,<br>
  <span style="{{ site.code }}">copy_form_user</span> 's <span style="{{ site.code }}">to</span> must not be less than its <span style="{{ site.code }}">n</span> .<br>
  --- x
  </p>
</details>
<br>

After kernel build & install,<br>
Check the changes.
```
$ reboot
$ modprobe my_char
$ echo "1234" > /dev/char
$ cat /dev/char
```
<br>

However, Occured kernel panic!<br>
You can check kernel panic with <span style="{{ site.code }}">dmesg</span> .
```
[   ...] Buffer overflow detected (1024 < 131072)!
```

Because read side,<br>
the <span style="{{ site.code }}">count</span> is the kernel's buffer.<br>
It is what need to copy the kernel's buffer size from the device buffer,<br>
The device buffer is only 1024 bytes.<br>

Fix it,
```
diff --git a/my_char.c b/my_char.c
index ...
--- a/my_char.c
+++ b/my_char.c
@@ -33,12 +33,16 @@ static int my_char_release(struct inode *inode, struct file *file)

 static ssize_t my_char_read(struct file *file, char __user *user_buffer, size_t count, loff_t *offset)
 {
+       ssize_t bytes_to_copy = min(count, (size_t)(sizeof(device_buffer) - *offset));
+
        // device -> user
-       if (copy_to_user(user_buffer, device_buffer, count) != 0) {
+       if (copy_to_user(user_buffer, device_buffer + *offset, bytes_to_copy) != 0) {
                return -EFAULT;
        }

-       return count;
+       *offset += bytes_to_copy;
+
+       return bytes_to_copy;
 }
```
The kernel determines the size of the data to be read,<br>
which is the minimum between the requested number of bytes and the current file offset ( <span style="{{ site.code }}">\*offset</span> ).<br>
<br>
The kernel copies data to the user buffer.<br>
It copies data from the kernel buffer to the user buffer using the <span style="{{ site.code }}">copy_to_user(to, from, size)</span> function.<br>
<br>
Update the file offset ( <span style="{{ site.code }}">\*offset</span> ),<br>
<B>increase the current file offset by the size of the read data to prepare for the next read action</B>.<br>
If you don't update the offset, you will continue to return the buffer value from the first location of the buffer.<br>
<br>
Returns the number of bytes read and the size of the data read and passes it to the user.<br>

After kernel build & install,<br>
Retry,
```
$ reboot
$ modprobe my_char
$ echo "1234" > /dev/char
$ cat /dev/char
1234
```
<br>

### ops: ioctl

You must add the function pointers <span style="{{ site.code }}">.unlocked unlocked_ioctl</span> in file_operations.<br>
to add ioctl operations.
```
diff --git a/my_char.c b/my_char.c
index ...
--- a/my_char.c
+++ b/my_char.c
@@ -9,10 +9,15 @@
 #include <linux/cdev.h>
 #include <linux/device.h>
 #include <linux/uaccess.h>
+#include <linux/ioctl.h>
+#include <linux/string.h>
+#include <linux/slab.h>

 #define DRIVER_NAME "my_char"
 #define NUM_DEVICES 1
 #define BUF_SIZE 1024
+#define IOCTL_WRITE _IOW('k', 1, char *)
+#define IOCTL_READ _IOR('k', 2, char *)

 static dev_t device_num;
 static struct cdev cdev;
@@ -55,12 +60,57 @@ static ssize_t my_char_write(struct file *file, const char __user *user_buffer,
        return count;
 }

+static long my_char_ioctl(struct file *file, unsigned int cmd, unsigned long arg)
+{
+    char *temp_buffer = NULL;
+    size_t len;
+
+    switch (cmd) {
+        case IOCTL_WRITE:
+            temp_buffer = kzalloc(BUF_SIZE, GFP_KERNEL);
+            if (!temp_buffer) {
+                       return -ENOMEM;
+            }
+
+            if (copy_from_user(temp_buffer, (char *)arg, BUF_SIZE) != 0) {
+                       return -EFAULT;
+            }
+
+            temp_buffer[BUF_SIZE - 1] = '\0';
+            strncpy(device_buffer, temp_buffer, BUF_SIZE);
+            printk(KERN_INFO "Received message from user space: %s\n", device_buffer);
+            break;
+        case IOCTL_READ:
+            temp_buffer = kzalloc(BUF_SIZE, GFP_KERNEL);
+            if (!temp_buffer) {
+                       return -ENOMEM;
+            }
+
+            len = strlen(device_buffer);
+            if (copy_to_user((char *)arg, device_buffer, len + 1) != 0) {
+                       return -EFAULT;
+            }
+
+            printk(KERN_INFO "Sent message to user space: %s\n", device_buffer);
+            break;
+        default:
+            return -ENOTTY; // command not recognized
+    }
+
+    // free dynamically allocated memory
+    if (temp_buffer)
+        kfree(temp_buffer);
+
+    return 0;
+}
+
 static struct file_operations my_char_fops = {
        .owner = THIS_MODULE,
        .open = my_char_open,
        .release = my_char_release,
        .read = my_char_read,
        .write = my_char_write,
+        .unlocked_ioctl = my_char_ioctl,
 };
```

<span style="{{ site.code }}">ioctl</span> is related to I/O control.<br>
Because it's an example, we decided to simply pass the string.<br>

If you create a file under <span style="{{ site.code }}">/dev</span> with the driver,<br>
You can control I/O by open, read and write files in the user space.<br>
When creating the actual driver, you can add control of I/O.<br>

Added <span style="{{ site.code }}">linux/string.h</span> and <span style="{{ site.code }}">linux/slab.h</span> for string processing.<br>
The <span style="{{ site.code }}">linux/slab.h</span> header is related to kernel hash mem.<br>
set up a virtual buffer by <span style="{{ site.code }}">kzalloc</span>, destroy by <span style="{{ site.code }}">kfree</span> .<br>
<br>

After kernel build & install,<br>
Try to communicate user space with kernel space.
```
/* test.c */
#include <errno.h>
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/ioctl.h>

#define IOCTL_WRITE _IOW('k', 1, char *)
#define IOCTL_READ _IOR('k', 2, char *)

int main()
{
    int fd;
    char send[] = "Hello world!";
    char recv[100];

    fd = open("/dev/my_char", O_RDWR);
    if (fd < 0) {
        perror("Failed to open the device...");
        return errno;
    }  

    if (ioctl(fd, IOCTL_WRITE, send) == -1) {
        perror("Failed to get data through ioctl...");
        return errno;
    }
    printf("Sent data to device driver: %s\n", send);

    if (ioctl(fd, IOCTL_READ, recv) == -1) {
        perror("Failed to set data through ioctl...");
        return errno;
    }  
    printf("Received data from device driver: %s\n", recv);

    close(fd);

    return 0;
}
```
```
$ gcc -o test test.c
$ ./main
$ dmesg
```
```
[   74.415852] my_char driver initialized
...
[   84.657010] my_char_open
[   84.657133] my_char release
[  192.975441] my_char_open
[  192.975466] Received message from user space: Hello world!
[  192.975662] Sent message to user space: Hello world!
[  192.975679] my_char release
[  203.249811] my_char_open
[  203.249922] my_char release
```
```
$ cat /dev/my_char
Hello world!
```
<br>

When writing the code, you should consider the buffer's memory size carefully.<br>
Be careful when using <span style="{{ site.code }}">copy_to_user</span> or <span style="{{ site.code }}">copy_from_user</span> .<br>
<br>

\+ fops also has <span style="{{ site.code }}">.comat_ioctl</span> that supports 32-bit operations.<br>

### ops: others

File operation has many other functions.<br>
<span style="{{ site.code }}">.mmap</span> or <span style="{{ site.code }}">.llseek</span> is also used a lot.<br>

## network device driver

Linux network device drivers are used to control and manage network interface cards (NICs).<br>
Below is an example code for network device drivers that control simple Ethernet NICs.<br>

### prepare

Create a virtual network device driver.<br>
file name is <span style="{{ site.code }}">my_net.c</span>
```
diff --git a/my_net.c b/my_net.c
new file mode 100644
index ...
--- /dev/null
+++ b/my_net.c
@@ -0,0 +1,66 @@
+// SPDX-License-Identifier: GPL-2.0
+/*
+ * Copyright (C) Steve Jeong <steve@how2flow.net>
+ */
+
+#include <linux/module.h>
+#include <linux/netdevice.h>
+#include <linux/etherdevice.h>
+
+#define DRIVER_NAME "my_net"
+#define MAC_ADDR_LEN ETH_ALEN
+
+static struct net_device *my_net_device;
+
+static int my_net_open(struct net_device *dev)
+{
+       printk(KERN_INFO "Network device opened\n");
+       netif_start_queue(dev);
+       return 0;
+}
+
+static int my_net_stop(struct net_device *dev)
+{
+       printk(KERN_INFO "Network device closed\n");
+       netif_stop_queue(dev);
+       return 0;
+}
+
+static struct net_device_ops my_net_device_ops = {
+       .ndo_open = my_net_open,
+       .ndo_stop = my_net_stop,
+};
+
+static int __init my_net_init(void)
+{
+       my_net_device = alloc_netdev(0, DRIVER_NAME, NET_NAME_UNKNOWN, ether_setup);
+       if (!my_net_device) {
+               printk(KERN_ERR "Failed to allocate net device\n");
+               return -ENOMEM;
+       }
+
+       my_net_device->netdev_ops = &my_net_device_ops;
+
+       if (register_netdev(my_net_device)) {
+               printk(KERN_ERR "Failed to register net device\n");
+               free_netdev(my_net_device);
+               return -ENODEV;
+       }
+
+       printk(KERN_INFO "Network device registered\n");
+       return 0;
+}
+
+static void __exit my_net_exit(void)
+{
+       unregister_netdev(my_net_device);
+       free_netdev(my_net_device);
+       printk(KERN_INFO "Network device unregistered\n");
+}
+
+module_init(my_net_init);
+module_exit(my_net_exit);
+
+MODULE_LICENSE("GPL v2");
+MODULE_AUTHOR("Steve Jeong");
+MODULE_DESCRIPTION("A simple network device driver example");
```
<br>

Minimal headers to define network(ethernet) devices
```
#include <linux/module.h>
#include <linux/netdevice.h>
#include <linux/etherdevice.h>
```
<br>

Elements for Network Device Initialization
```
#define DRIVER_NAME "my_net"
#define MAC_ADDR_LEN ETH_ALEN

static struct net_device *my_net_device;
```
<br>

A device driver is what makes a device interactable on a user interface.<br>
Therefore, the device must exist as a file in the user interface.<br>

Define <span style="{{ site.code }}">net_device_ops</span> to exist as a net.<br>
Most basic implementation of open/stop.
```
static int my_net_open(struct net_device *dev)
{
       printk(KERN_INFO "Network device opened\n");
       netif_start_queue(dev);

       return 0;
}

static int my_net_stop(struct net_device *dev)
{
       printk(KERN_INFO "Network device closed\n");
       netif_stop_queue(dev);

       return 0;
}

static struct net_device_ops my_net_device_ops = {
       .ndo_open = my_net_open,
       .ndo_stop = my_net_stop,
};
```
<br>

Implementing module init/exit<br>
with macro <span style="{{ site.code }}">module_init</span> and <span style="{{ site.code }}">module_exit</span> .
```
static int __init my_net_init(void)
{
       my_net_device = alloc_netdev(0, DRIVER_NAME, NET_NAME_UNKNOWN, ether_setup);
       if (!my_net_device) {
               printk(KERN_ERR "Failed to allocate net device\n");
               return -ENOMEM;
       }

       my_net_device->netdev_ops = &my_net_device_ops;

       if (register_netdev(my_net_device)) {
               printk(KERN_ERR "Failed to register net device\n");
               free_netdev(my_net_device);
               return -ENODEV;
       }

       printk(KERN_INFO "Network device registered\n");

       return 0;
}

static void __exit my_net_exit(void)
{
       unregister_netdev(my_net_device);
       free_netdev(my_net_device);
       printk(KERN_INFO "Network device unregistered\n");
}

module_init(my_net_init);
module_exit(my_net_exit);
```
<br>

Add information about the module.
```
MODULE_LICENSE("GPL v2");
MODULE_AUTHOR("Steve Jeong");
MODULE_DESCRIPTION("A simple network device driver example");
```
<br>

After kernel build & install,
check kernel message with <span style="{{ site.code }}">dmesg</span> .
```
$ reboot
$ modprobe my_net
$ lsmod | grep my_net
my_net                 16384  0
```
```
$ dmesg
[   ...] Network device registered
```
<br>

You can find a network device
```
$ ls /sys/class/net
eth0 lo my_net
```
<br>
Also, you can find with <span style="{{ site.code }}">ifconfig</span>
```
$ ifconfig my_net
my_net: flags=4098<BROADCAST,MULTICAST>  mtu 1500
        ether 00:00:00:00:00:00  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
<br>

You need to implement logic to send and receive network packets correctly.<br>

### ops: tx/rx

You must add the function pointers <span style="{{ site.code }}">.ndo_start_xmit</span> in net_device_ops and add the handler <span style="{{ site.code }}">rx_handler_result_t</span><br>
to add tx/rx operations
```
diff --git a/my_net.c b/my_net.c
index ...
--- a/my_net.c
+++ b/my_net.c
@@ -4,8 +4,11 @@
  */

 #include <linux/module.h>
+#include <linux/kernel.h>
+#include <linux/init.h>
 #include <linux/netdevice.h>
 #include <linux/etherdevice.h>
+#include <linux/skbuff.h>
 
 #define DRIVER_NAME "my_net"
 #define MAC_ADDR_LEN ETH_ALEN
@@ -28,9 +31,29 @@ static int my_net_stop(struct net_device *dev)
        return 0;
 }
 
+static netdev_tx_t my_net_xmit(struct sk_buff *skb, struct net_device *dev)
+{
+       struct sk_buff *new_skb;
+
+       printk(KERN_INFO "transmit\n", THIS_MODULE);
+
+       new_skb = skb_copy(skb, GFP_ATOMIC);
+       if (!new_skb) {
+               printk(KERN_ERR "skb_copy failed\n");
+               return NETDEV_TX_OK;
+       }
+
+       netif_rx(new_skb);
+
+       dev_kfree_skb_any(skb);
+
+       return NETDEV_TX_OK;
+}
+
 static struct net_device_ops my_net_device_ops = {
        .ndo_open = my_net_open,
        .ndo_stop = my_net_stop,
+       .ndo_start_xmit = my_net_xmit,
 };
 
 static int __init my_net_init(void)
@@ -42,6 +65,9 @@ static int __init my_net_init(void)
        }
 
        my_net_device->netdev_ops = &my_net_device_ops;
+       my_net_device->type = ARPHRD_LOOPBACK;
+       my_net_device->mtu = 1500;
+       my_net_device->flags |= IFF_LOOPBACK;
 
```
<br>

After kernel build & install,<br>
Check the changes with <span style="{{ site.code }}">test.sh</span>.
