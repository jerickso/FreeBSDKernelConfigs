# FreeBSD Kernel Configs (For use on virtual machines)
This repository contains preconfigured amd64 FreeBSD Kernel Configs, mainly for various virtual servers or cloud based systems.  These configs can be used as a base for your own custom kernels on actual hardware or in the cloud.  Currently, the configs are for FreeBSD version 13.0.

## Why build your own kernel?
Even though some of the functionality in the kernel of FreeBSD is contained in modules, building a custom kernel provides:
* Faster boot times
* Lower memory usage
* Lower storage requirements
* Possibility of including drivers/functionality not included by the base kernel
* Faster kernel build times if rebuilding from source
* Slightly more secure as the attack surface is lowered because unused code is removed

## Why would I NOT want to build my own kernel?
* If you use `freebsd-update` to keep your system up to date, that utility will stop working if you have a custom kernel.
* If you do not know what hardware you have, installing a custom kernel can prevent your system from booting, requiring you to go through a recovery procedure (which may consist of booting off recovery media, mount the root partition, move /boot/kernel to /boot/kernel.bad, move /boot/kernel.old to /boot/kernel, and rebooting, tho your mileage may vary)
* You don't have the time, desire, or need to build your own kernel

## OK, what are the files in this Repository?
The files are:
* BASE - ALL other configs import this file.  This file is the bare minimum configuration with most bells and whistles removed.  This config will probably not work on its own as it removes all of the NIC drivers and most storage drivers.  This file imports the GENERIC config file and disables most of the items that I think are unnecessary for most of my configurations.  The advantage of importing GENERIC (Instead of copying what is necessary) is when GENERIC is updated, a rebuild will include any new features or functionality added to GENERIC which may be necessary to boot the system.
* VMWARE - Build for a VMware system, ie ESXi, Fusion, or VMware Workstation
* AZURE - Build for Microsoft's Azure, Hyper-V, or Virtual PC

Future/Untested config files:
* GOOGLE - Build for the Google Compute Engine
* AWS - Build for Amazon Web Services EC2


## To use
Make sure to have the latest stable sources at /usr/src.

Copy all these files into the following directory on a FreeBSD system:
`/usr/src/sys/amd64/conf`


Note, there is a make build variable 'KERNCONFDIR' that can be set to override the directory where the KERNCONF file is at.  But since the BASE config file references the default GENERIC kernel config located in the default directory, setting this variable will not work as it will not find the GENERIC kernel.  Until setting this variable allows multiple directories or checks the base directory in its search, this option is not viable.


### Building (Way 1)
```shell
cd /usr/src
make buildkernel KERNCONF=VMWARE
make installkernel KERNCONF=VMWARE
```

# Building (Way 2)
If you rebuild your kernel often, you can also place the KERNCONF assignment in `/etc/make.conf`:
```make
KERNCONF=VMWARE
```

And build without the assignment on the command line:
```shell
cd /usr/src
make buildkernel
make installkernel
```

## How can I tell what options were used to build a kernel?
If the `options INCLUDE_CONFIG_FILE` was set on the kernel when it was built (which is set on the default GENERIC kernel), you can use the `config` command to extract the kernel build options.  To see what options are in your existing kernel, type the following command:
```shell
config -x /boot/kernel/kernel | more
```

## What other options are possible for a kernel config file?
```shell
cd /usr/src/sys/amd64/conf
make LINT
more LINT
```
While this will list the possible options, it will not tell you what those options do.  The `/usr/src/sys/amd64/conf/NOTE` file may provide some insight into some of the options, but be aware that there is a lot of limited documented options.

## Hmmmmm, I may have broken something with my new kernel
If you build your kernel using the 'buildkernel', 'installkernel' method, the following procedure 'may' work:
Upon booting, you should encounter the FreeBSD boot loader screen that says "Welcome to FreeBSD", hit '5' to change the kernel to use `kernel.old`.  Then hit `Enter` to boot the old kernel.  Once you have logged in, type the following to restore your old kernel
```
mv /boot/kernel /boot/kernel.bad
mv /boot/kernel.old /boot/kernel
reboot
```

## Is there a way for me to know what drivers the kernel has loaded?
This one liner will list the drivers the kernel has detected.  After booting up, log in and run the following:
```shell
dmesg | grep '^[[:lower:]_]*[0-9]:' | cut -d ':' -f 1 | sed -r 's/[0-9]*//g' | sort -u
```

## References
[FreeBSD Handbook on If Something Goes Wrong](https://www.freebsd.org/doc/handbook/kernelconfig-trouble.html)
[FreeBSD Handbook on Building a Custom Kernel](https://www.freebsd.org/doc/handbook/kernelconfig-building.html)
