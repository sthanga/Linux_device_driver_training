# To be Studied all of the below conternt on Linud Device Driver
# 1. Char Drivers
```bash
A character device driver (char driver) allows user-space applications to read/write data one character (byte) at a time.
  /dev/ttyS0 → serial port
  /dev/i2c-* → I²C bus
  /dev/spidev* → SPI
  /dev/random
  GPIO, sensors, misc devices
  Char drivers use the kernel’s file_operations interface to handle:
  ✔ open()
  ✔ read()
  ✔ write()
  ✔ release()
  ✔ ioctl() (optional)
Allocate a device number (major/minor) :
    major number → identifies the driver
    minor number → identifies the device instance
Create and initialize a cdev structure:
  cdev_init(&my_cdev, &fops);
  cdev_add(&my_cdev, dev, 1);
 This links your driver's file_operations with the device number.
Implement file_operations:
This structure maps syscalls to driver functions:
      static struct file_operations fops = {
          .owner   = THIS_MODULE,
          .open    = my_open,
          .release = my_release,
          .read    = my_read,
          .write   = my_write,
      };

Create device node in /dev  :
    device_create(my_class, NULL, dev, NULL, "mychardev");
    User apps will then see:
    /dev/mychardev
```
```c
#include <linux/init.h>
#include <linux/module.h>
#include <linux/fs.h>
#include <linux/cdev.h>
#include <linux/uaccess.h>

#define BUFFER_SIZE 1024

static dev_t dev;
static struct cdev my_cdev;
static struct class *my_class;

static char kernel_buffer[BUFFER_SIZE];

static int my_open(struct inode *inode, struct file *file)
{
    pr_info("mychardev: device opened\n");
    return 0;
}

static int my_release(struct inode *inode, struct file *file)
{
    pr_info("mychardev: device closed\n");
    return 0;
}

static ssize_t my_read(struct file *file, char __user *buf, size_t len, loff_t *offset)
{
    pr_info("mychardev: read requested (%zu bytes)\n", len);

    if (*offset >= BUFFER_SIZE)
        return 0;

    if (len > BUFFER_SIZE - *offset)
        len = BUFFER_SIZE - *offset;

    if (copy_to_user(buf, kernel_buffer + *offset, len))
        return -EFAULT;

    *offset += len;
    return len;
}

static ssize_t my_write(struct file *file, const char __user *buf, size_t len, loff_t *offset)
{
    pr_info("mychardev: write requested (%zu bytes)\n", len);

    if (len > BUFFER_SIZE)
        len = BUFFER_SIZE;

    if (copy_from_user(kernel_buffer, buf, len))
        return -EFAULT;

    return len;
}

static struct file_operations fops = {
    .owner   = THIS_MODULE,
    .open    = my_open,
    .release = my_release,
    .read    = my_read,
    .write   = my_write,
};

static int __init my_init(void)
{
    alloc_chrdev_region(&dev, 0, 1, "mychar");
    pr_info("mychardev: registered with major=%d minor=%d\n", MAJOR(dev), MINOR(dev));

    cdev_init(&my_cdev, &fops);
    cdev_add(&my_cdev, dev, 1);

    my_class = class_create("mycharclass");
    device_create(my_class, NULL, dev, NULL, "mychardev");

    return 0;
}

static void __exit my_exit(void)
{
    device_destroy(my_class, dev);
    class_destroy(my_class);
    cdev_del(&my_cdev);
    unregister_chrdev_region(dev, 1);

    pr_info("mychardev: unloaded\n");
}

MODULE_LICENSE("GPL");
MODULE_AUTHOR("You");
MODULE_DESCRIPTION("Simple Character Device Driver Example");

module_init(my_init);
module_exit(my_exit);

```
# 2. Debugging Techniques
# 3. Concurrency and Rase Condition
# 4. Advanced Char Driver Operation
# 5. Time Delay, and Deferred work
# 6. Allocating Memory
# 7. Communication with Hardware
# 8. Interrupt Handling 
# 9. Data Types in Kernel
# 10. PCI Drivers
# 11. USB Driver
# 12. The Linux Device Driver Model
# 13. Memeory Mapping and DMA
# 14. Block Driver
# 15. Network Driver
# 16. TTY Driver

