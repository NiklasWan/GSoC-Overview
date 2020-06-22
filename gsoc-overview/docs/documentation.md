# Coding Conventions

This document basically is just a summary of the linux kernel coding standards, which can be found [here](https://www.kernel.org/doc/html/v4.10/process/coding-style.html#).

- Indentation width: 8 chars
- More than 3 levels of indentation are a sign of bad code design
- switch statemenst: case labels not indented!
- no whitespaces at the end of files
- no multiple assignments on single line
- Max. Line Length is 80 Columns unless readability is significantly improved when exceeding the limit
  - However user-visible strings such as printk msgs should not be split

## Placing braces

- non-function statements:

```c
if (x is true) {
        we do y;
}
```

- function statements:

```c
int function(int x)
{
        body of function;
}
```

- closing brace is on a line by its own, except it is followed by continuation of the same statement e.g. do-while loop if- else if - else structures
- When single statement will do dont use braces
- However when one branch in a conditional statement has more than one statement use braces in all conditional branches i.e:

```c
if (condition) {
        do_this();
        do_that();
} else {
        otherwise();
}
```

## Spaces

- use spaces after following keywords:
  `if, switch, case, for, do, while``
- but not for:
  `sizeof, typeof, alignof, or __attribute__`
- no spaces inside paranthesized expressions
- define a pointer like:

```c
int *my_pointer;
```

- use one space around those binary and ternary operators:
  `= + - < > * / % | & ^ <= >= == != ? :`
- no spaces for:
  `& * + - ~ ! sizeof typeof alignof __attribute__ defined -- ++ . ->`
- No trailing whitespace at the end of lines

## Naming

- generally variable names should be short
- global variables and function should be descriptive
- use global variables only when you really need them

## Typedefs

- no typedefs for structures and pointers
- Just use a type def when one of the rules listed [here](https://www.kernel.org/doc/html/v4.10/process/coding-style.html#typedefs) applies

## Functions

- should do one thing and do it well
- should fit on two screenfuls of text (the ISO/ANSI screen size is 80x24)
- use helper functions with descriptive names, when splitting up functions
- number of local variables should not exceed 5-10
- seperate functions by one blank line in source files
- when function is **exported** macro should follow immediately after function, i.e.:

```c
int system_is_up(void)
{
        return system_state == SYSTEM_RUNNING;
}
EXPORT_SYMBOL(system_is_up);
```

- for prototypes include both type and name

## Centralized exiting of functions

- goto statements can be used to exit from multiple locations in a function
- goto allows for reduced nesting
- GW-BASICS like `err1:` or `err2:` should be avoided

## Commenting

- never try to expain how code works
- tell what code does
- try avoid comments in function bodys
- when commenting kernel API functions use kernel-doc format
- comment data
  Preffered Style for multiline comments:

```C
/*
 * This is the preferred style for multi-line
 * comments in the Linux kernel source code.
 * Please use it consistently.
 *
 * Description:  A column of asterisks on the left side,
 * with beginning and ending almost-blank lines.
 */
```

Preferred style for files in /inet and drivers/net:

```C
/* The preferred comment style for files in net/ and drivers/net
 * looks like this.
 *
 * It is nearly the same as the generally preferred comment style,
 * but there is no initial almost-blank line.
 */
```

## Kconfig configuration files

- lines under a config definition are indented with one tab, help text is indented with two additional spaces
- dangerous features should advertise this in their prompt string

## Data structures

- data structures, which are used outside the single-threaded environment they are created and destroyed in should always have reference counts

## Macros, Enums and RTL

- define constants and labels like this:

```c
#define CONSTANT 0x12345
```

- generally inline functions are preferred over macros
- Macros with multiple statements should be enclosed in a do - while block, i.e.:

```c
#define macrofun(a, b, c)                       \
        do {                                    \
                if (a == 5)                     \
                        do_this(b, c);          \
        } while (0)
```

## Printing kernel messages

- kernel mnessages should be concise, clear and unambiguous
- Printing numbers should be avoided
- use drive model diagnostic macros in <linux/device.h>, they make sure messages are matched to the right device and driver
- messages not associated with a particular driver should use macros defined in <linux/printk.h>

## Allocating memory

- use kmalloc(), kzalloc(), kmalloc_array(), kcalloc(), vmalloc(), and vzalloc() to allocate memory in kernel space
- preferred way of passing size of a struct:

```c
p = kmalloc(sizeof(*p), ...);
```

- preferred for allocating an array:

```c
p = kmalloc_array(n, sizeof(...), ...);
```

- preferred way of allocating zeroed array:

```c
p = kcalloc(n, sizeof(...), ...);
```

## The inline disease

- abundant use of inline function leads to a bigger kernel, which slows down the system
- rule of thumb: inline functions not longer than 3 lines of code
- exception: when a parameter is known to be a compiletime constant and the compiler can optimize most of function away at compile time

## Function return values and names

- Convention for return types of functions, which dont return the value of a computation:

```
""If the name of a function is an action or an imperative command,
the function should return an error-code integer.  If the name
is a predicate, the function should return a "succeeded" boolean.""
```

## Don’t re-invent the kernel macros

- look in <include/linux/kernel.h> for helpfull macros, to prevent writing your own versions

## Editor modelines and other cruft

- Don't include editor configuration options in source files

## Inline assembly

- use assembly only when neccessary
- write helper functions to wrap common parts of inline assembly
- large non-trivial assembly functions go in .S files, with corresponding C prototypes in C headerfiles
  - corresponding C prototypes for assembly functions should use asmlinkage
- For single line assembly with multiple instructions: put every instruction on a single line in a separated quoted string

## Coditional Compilation

- always try to avoid preprocessor conditionals in .c files, try to use them in .h files, defining functions for use in .c files. For #else case provide no-op stub version of function
- use IS_ENABLED macro to convert Kconfig symbol into a C boolean expression where possible and use in a normal C conditional

## References

[1] https://www.kernel.org/doc/html/v4.10/process/coding-style.html




# Embedded Linux on BBB

## Before getting started, the required toolchain has to be setup:

Development machine runs on Ubuntu Linux 18.04 LTS

### Steps to compile Linux Kernel:

1. Cross Compiler, Git and lzop:

   To allow the development machine to compile against a different architecture (in this case ARM), a cross compilation toolchain is needed.

   Installation is achieved via: `sudo apt-get install gcc-arm-linux-gnueabi`

   To clone required linux repos, install Git: `sudo apt-get install git`

   To be able to compress the kernel, install lzop: `sudo apt-get install lzop`

   To be able to install Uboot, libssl-dev needs to be installed: `sudo apt-get install libssl-dev`

2) U-boot:

   The Beagleboard Black uses the uboot boot loader to boot the system.

   To install U-Boot first download the latest version using: `wget ftp://ftp.denx.de/pub/u-boot/u-boot-<year>.<month>.tar.bz2`

   Untar with: `tar -xjf u-boot-<year>.<month>.tar.bz2`

   Change into directory: `cd u-boot-<year>.<month>`

   Compile Uboot: `make sandbox_defconfig tools-only`

   Install mkimage for Uboot image creation: `sudo install tools/mkimage /usr/local/bin`

3) Get and compile the right kernel:

   To download the Kernel got to Beagleboard.org GitHub and clone the Linux Kernel: `git clone git://github.com/beagleboard/linux.git`

   Change into directory and check out right Branch (we use 4.19-rt): `cd linux && git checkout 4.19-rt`

   Configure the Linux Buildsystem with default BBB configuration file: `make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- bb.org_defconfig`

   Build Linux zImage: `make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- -j4`

   Build Device Tree Binaries and uImage with: `make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- uImage dtbs LOADADDR=0x80008000 -j4`

   Build kernel modules with: `make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- modules -j4`

4) Install to BBB via TFTP:
   To configure BBB for TFTP setup, a USB-to-Serial cable is higly recommended.

   First insert micro SD Card to Development Machine and create 2 Partitions using i.e. gparted. The first will be the boot partition, so make sure to use fat and enable boot flag. The second will be ext3 and will host the root file system. Copy your root file system to the ext3 partition and copy MLO bootloader and uEnv.txt to the fat boot partition.

   Configure your uEnv.txt the folowing way:

   ```
   console=ttyO0,115200n8
   ipaddr=192.178.27.2
   serverip=192.178.27.1
   loadaddr=0x82000000
   fdtaddr=0x88000000
   loadfromsd=tftp ${loadaddr} uImage;tftp ${fdtaddr} am335x-boneblack.dtb
   linuxbootargs=setenv bootargs console=${console} root=/dev/mmcblk0p2 rw
   uenvcmd=setenv autoload no; run loadfromsd; run linuxbootargs; bootm ${loadaddr} - ${fdtaddr}
   ```

   To configure Linux Development machine as tftp host set ip to serverip: `sudo ifconfig <your network interface> 192.178.27.1`

   Now install tftp and configure accordingly: `sudo apt-get install tftpd`

   Go to `/etc/tftpd.d` and create a file called `tftp`. Copy the following content into the file for configuration:

   ```
   # description: The tftp server serves files using the trivial file transfer
   #       protocol.  The tftp protocol is often used to boot diskless
   #       workstations, download configuration files to network-aware printers,
   #       and to start the installation process for some operating systems.
   service tftp
   {
         socket_type             = dgram
         protocol                = udp
         wait                    = yes
         user                    = root
         server                  = /usr/sbin/in.tftpd
         server_args             = -s /home/cpeacock/export
   #       disable                 = yes
         per_source              = 11
         cps                     = 100 2
   }
   ```

   Make directory according to server line in config: `sudo mkdir usr/sbin/in.tftpd`

   Copy the `uImage` and the `am335x-boneblack.dtb` to the tftp directory and boot your BBB with ethernet connection enabled to your development machine.

## References:

[1] https://wiki.beyondlogic.org/index.php?title=Setup_TFTP_Server

[2] https://wiki.beyondlogic.org/index.php/BeagleBoneBlack_Building_Kernel

# CTAG Face Alsa Driver

The CTAG Face Alsa Driver runs on 4.19-rt, for both BBB and BBAI. To get the Face running on your device, please follow all steps.

1.) Clone kernel from [here](https://github.com/NiklasWan/linux/tree/dev_gsoc_face_4.19-rt)
    `git clone https://github.com/NiklasWan/linux/tree/dev_gsoc_face_4.19-rt`
2.) Load BB device configuration:
    `make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- bb.org_defconfig`
3.) Open menuconfig and Go to Device Drivers - Sound Card Support - Advanced Linux Sound Architecture - ALSA for SoC audio support:
    Enable module build for SoC Audio Support for CTAG face-2-4 Audio Card (AD1938)
4.) Build kernel:
    `make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- bindeb-pkg -j8`
5.) Download current Image for your device at: https://beagleboard.org/latest-images
6.) Install image using Balena Etcher
7.) Copy built linux image, linux header and linux libc to sd card:
    `sudo cp linux-* /media/dev/rootfs/home/debian`
8.) Start BB from Sd Card and Install kernel:
    `sudo dpkg -i linux-headers*`
    `sudo dpkg -i linux-libc*`
    `sudo dpkg -i linux-image*`
9.) Restart:
    `sudo restart`
10.) Clone overlay repo:
    `git clone https://github.com/NiklasWan/bb.org-overlays.git && cd bb.org-overlays`
11.) Install dtc + dependencies:
    `sudo ./dtc-overlays.sh`
12.) Install overlays:
    `git checkout dev_gsoc_face && sudo ./install.sh`
13.) Edit Uboot uEnv.txt:
    For BBAI:
      add the following lines to /boot/uEnv.txt:
        `enable-uboot-overlays=1`
        `uboot_overlay_addr0=/lib/firmware/BBAI_BB-BONE-FACE-8CH-00A0.dtbo`
    For BBB:
      comment out the following line:
        `#enable_uboot_cape_universal=1`
      uncomment:
        `disable_uboot_overlay_video=1`
        `disable_uboot_overlay_audio=1`
      add: 
        `uboot_overlay_addr0=/lib/firmware/BB-CTAG-SW-8CH-00A0.dtbo`

14.) Clone https://github.com/ctag-fh-kiel/ctag-face-2-4.git:
  `git clone https://github.com/ctag-fh-kiel/ctag-face-2-4.git & cd ctag-face-2-4`
15.) Install alsa config:
  `cd alsa-configs & sudo cp asound.conf.8ch ~/.asoundrc`
16.) Reboot your board and use the audio card :-)