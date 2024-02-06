---
layout: post
title: "Custom Linux build on Raspberry Pi"
---

I recently built and installed Linux on my Raspberry Pi 4B. This post is a reference with links to tutorials I followed including corrections to what worked and what didn't, mainly intended for future reference.

## Background Story AKA Things That Went Wrong

All the things mentioned here don't work, or atleast I couldn't get them to work. Feel free to skip this section.

I started with installing Raspbian and flashing the OS to my SD card using the Raspberry Pi Imager utility. The Pi connected to my WiFi network and I logged in using SSH. Easy peasy wireless connection. I wanted to swap out the kernel with the one I built so I can mess around with the sources and try it out on real hardware. Turns out there are couple of problems with this:

1. Raspberry Pi has its own fork of Linux that it uses. [Some people](https://forums.raspberrypi.com/viewtopic.php?t=357536) have got the mainline kernel to work but it seemed like an additional hassle. This GitHub issue [tracks changes that are yet to be merged upstream](https://github.com/lategoodbye/rpi-zero/issues/43)
2. While I could theoretically put the newer kernel image through ssh, what if the Pi failed to boot afterwards? I'd have to manually insert the card to my laptop and revert it, which I wanted to avoid.

Luckily, the bootloader (which is closed source by the way) provides utilities like tryboot and autoboot for issues like this. This is mentioned in [the docs](https://www.raspberrypi.com/documentation/computers/config_txt.html#the-tryboot-filter) and [this forum post](https://forums.raspberrypi.com/viewtopic.php?t=341372) suggests it works. According to it, you could have multiple boot partitions and boot from any of them. So in my case, I could have the original kernel in one partition and my own built kernel in the other boot partition. Using tryboot, if booting the custom kernel failed, it would revert back to the original kernel and I could debug it.

But of course, I couldn't get this magical solution to work. I made 2 bootfs partitions on my sd card which would share a rootfs. But try as I may, it always rebooted from the partition with the original kernel. I tried to have the original kernel in the other partition too, only tweaking the kernel cmdline arguments but it didn't work. I have no clue why it didn't work, maybe making the second boot partition after root partition doesn't work (though forum posts suggest it does) or perhaps there was always some error in booting the other partition so it was reverting. I couldn't find any logs for it and since there is no source code to look at, I was stumped.

So after more browsing, I decided to change the bootloader to U-Boot. Maybe it wasn't really necessary but there looked to be no other way out.

## Serial Cable

To see early boot logs and get a console, a serial cable is needed to connect to the Raspberry Pi. It makes a lot of sense for logging since it is supposed to be a log of events that happened before the filesystem was even initialized. I bought [this cable off Robu](https://robu.in/product/pl2303-ta-download-cable-usb-ttl-rs232-module-usb-serial/) though any RS232 USB to TTL converter would work. Note that supplying power using this cable did turn on the Raspberry pi but I didn't get serial logs, not sure what's up with this. So, I just connect the RXD (to TXD port on board), TXD (to RXD), GND (to GND) according to the following GPIO layout.

![Raspberry Pi 4B GPIO Pinout Diagram](https://www.raspberrypi.com/documentation/computers/images/GPIO-Pinout-Diagram-2.png)

## Things That Worked

The post, [Boot a Raspberry Pi 4 using u-boot and Initramfs](https://hechao.li/2021/12/20/Boot-Raspberry-Pi-4-Using-uboot-and-Initramfs/), was a very good guide and I followed it almost exactly to make things work. I won't write things that are already written in it, I'll add extra things that I did to make my setup work:

- **Setup minicom**: I followed this [Getting Started guide](https://wiki.emacinc.com/wiki/Getting_Started_With_Minicom) setting the serial device as `/dev/ttyUSB0`. Apart from this, I needed to set `Serial Device Setup > Hardware Flow Control` to `No` to be able to use the serial console. `screen` can also be used to view the logs, but it doesn't seem to be able to give input to the console.
- **Early kernel logs**: My kernel was not booting up and I couldn't see any logs other than those U-Boot. I added the config option [CONFIG_EARLY_PRINTK](https://cateee.net/lkddb/web-lkddb/EARLY_PRINTK.html) which "is useful for kernel debugging when your machine crashes very early before the console code is initialized." To enable these logs, also add `earlycon` argument to the kernel command line. This involves changing U-Boot's `boot_cmd.txt` and comping it. Note that a lot of sources online say to use `earlyprintk` argument, but that was dropped from the kernel in favour of `earlycon`.
- **DO use initramfs**: For some reason that I'm not looking into for the moment, following the part of the tutorial that doesn't use initramfs, the control is not passed over to the BusyBox init. I tried it out by trying to `touch` files from the `init_main()` function in `init/init.c` and `touch`ing a file from the `rcS` script. Maybe changing the `shell` command line argument might help?
- **Additional steps**: Following are some additional steps I followed based on [Build a Raspberry Pi Linux System the Hard Way](https://rickcarlino.com/2021/build-a-raspbery-pi-linux-system-the-hard-way.html). Note that the tutorial is for Raspberry Pi 3B so following it primarily might result in some other issues.
  - Install Kernel Modules. This isn't mentioned in the primary tutorial I was following but seems to make sense to do
  - Root Filesystem, Part II: This talks about setting up procfs and sysfs, through BusyBox's rcS script. It also populates `/dev` folders using BusyBox's `mdev`.
- **Kernel cmdline**: This is important to get the console command line over serial, using the one specified in the article doesn't work. It should be `console=tty0 console=ttyS0,115200 earlycon 8250.nr_uarts=1 loglevel=8 root=/dev/mmcblk0p2 rw rootwait`.
  - `console` arguments should be as it is and in the specified order as well. Specifying `serial0` instead of `ttyS0` causes shell prompt to not appear, `tty0` looks to be the serial tty. Needs to be looked into more about why only this works and not other.
  - `8250.nr_uarts` is [required by the mini-uart's driver](https://forums.raspberrypi.com/viewtopic.php?t=246215#p1659905), otherwise kernel panics on boot
  - `loglevel` as 8 specifies that debug logs should also be printed to console

## Conclusion

This gets one's own built Linux running on Raspberry Pi 4B. Though the userspace is lacking since it only has a busybox shell, this should be easily upgradeable since at this point the kernel is passing control to the init system properly. Perhaps someday when this setup is stable that I'm working on it via ssh over Wi-Fi, I will try to make something like tryboot work so kernel can be upgraded over ssh. Till then, adios :)

## Appendix: Changes to environment before rebuilding

Some changes need to be made to the PATH and other variables for rebuilding after opening a new terminal. Note that these are taken from [the tutorial mentioned above](https://hechao.li/2021/12/20/Boot-Raspberry-Pi-4-Using-uboot-and-Initramfs/) and are here just for easy reference:

1. **Cross Compiler**: `export PATH=${HOME}/x-tools/aarch64-rpi4-linux-gnu/bin/:$PATH`: Setup before running any other command
2. **U-Boot**:
   1. `export CROSS_COMPILE=aarch64-rpi4-linux-gnu-`: To use the cross compiler
   2. `make`: Build U-Boot
   3. `cp u-boot.bin /media/arpit/bootfs`: Copy U-Boot binary to bootfs
   4. `./tools/mkimage -A arm64 -O linux -T script -C none -d boot_cmd.txt boot.scr; cp boot.scr /media/arpit/bootfs/`: Change boot commands
3. **Kernel**:
   1. `make -j8 ARCH=arm64 CROSS_COMPILE=aarch64-rpi4-linux-gnu-`: Build kernel
   2. `cp arch/arm64/boot/Image /media/arpit/bootfs`: Copy kernel image to bootfs
   3. `cp arch/arm64/boot/dts/broadcom/bcm2711-rpi-4-b.dtb /media/arpit/bootfs/`: Copy DTB file to bootfs (not needed if source DTS file is unchanged)
4. **BusyBox**:

   ```sh
   CROSS_COMPILE=${HOME}/x-tools/aarch64-rpi4-linux-gnu/bin/aarch64-rpi4-linux-gnu-
   make CROSS_COMPILE="$CROSS_COMPILE" # Build
   sudo make CROSS_COMPILE="$CROSS_COMPILE" install # Install in /media/arpit/rootfs, sudo since it's owned by root
   ```

