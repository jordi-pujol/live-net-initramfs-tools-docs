# live-net-initramfs-tools

#  More initramfs-tools features.

initramfs-tools is a Debian utility to boot a Debian Linux OS.

This is a fork of the Debian project.

I am developing new features and enhancing the standard utility.

New features:
  - 1- rootdir. Main filesystem resides in a subdirectory of the root device.
  - 2- liveupp, livelow. Boot a Live OS, with or without persistence.
  - 3- mountrd. Mount over rootmnt the entries listed in an fstab formatted 
file.
  - 4- swap. Option to early mount swap partitions or swap files.
  - 5- conf=c1,c2,...,cN Option to specify a bunch of configuration options.
Configured in a script stored in /etc/initramfs-tools/conf.
  - 6- modules. This option specifies a list of modules to be loaded 
early in the initramfs-tools scripts.

New feature:

1- Rootdir. Main filesystem resides in a subdirectory of the root device.

Option:

    - rootdir=/dirname # name of the directory where the filesystem resides

Option rootdir may not be specified at the same time that the live* 
options.

New feature:

2- Live. Boot a Live OS, with or without persistence.

The upper directory may reside on tmpfs or on a disk directory.

Options:

    - liveupp=/dir[@devname][@cleanup]
   ```
      Values:
      - /dir@devname@cleanup - upper directory on a device,
        will use the root device when devname is not specified.
                device parameter is positional.
        cleanup - clear out persistent directory before booting the filesystem.
                cleanup parameter is positional.
      - @tmpfs - temporary filesystem, no persistence
   ```
    - livelow=/dir1/low1[@devname1][@toram|tozram|tocache,mount][;...]
   ```
      Value is a semicolon (;) separated list of lower filesystems.
      The option livelow=  may be specified multiple times to add more
      lower filesystems to the overlay.
      Every lower filesystem is contained in a file or a directory:
      - an squashfs file or a file that contains any other filesystem type.
        livelow=/path/to/filesystem.squashfs@devname
      - a directory.
        livelow=/path/to/dir.d@devname
      - options toram or tozram, and their companion option mount:
        will create a RAM device and copy the file or dir into.
        Option toram uses a tmpfs device and option tozram uses a 
        compressed zram device. The device that originally contains the 
        file will be unmounted when is not needed if the mount option 
        is not specified.
      - option tocache:
        Use vmtouch to load files or dirs into filesystem's disk page
        cache and lock the corresponding memory pages.
   ```

New feature:

3- Mountrd.

Mount over rootmnt the entries listed in an fstab formatted 
file. This is some kind of special format where the first two fields of 
an entry may contain escape characters and shell functions, also must 
be sequentially ordered.

Will check the filesystem for device entries where MNT_PASS is not 
equal to 0. See the examples below.

Options:

- mountrd
   ```
      the default file name is /etc/initrd.fstab
    - mountrd=[/path/to/some-file.fstab][@devname[@mount]]
   ```

Extension for the device name:
A single slash "/" means that the path of the fstab file must be from 
the rootmnt directory.

Default fstab file is /etc/initrd.fstab in the ROOT device.

* Format of an initrd.fstab file:
   ```
  # <file system>	<mount point>	<type>	<options>	<dump>	<pass>
  Specification of the source file system.
    - Default source is a directory in the root filesystem.
    - This form is also allowed:
      directory@device[@toram|tozram|tocache,mount]
   ```

New feature:

4- swap.

Option to early mount swap partitions or swap files.
Options:
    - swap
   ```
      auto detect existing swap partitions
      swap=[/path/to/swfile.sw][@devname][@options];...;[/path/to/swfile.sw][@devname][@options]
   ```
    - noswap
   ```
      clear the swap variable that contains the value of previously
      specified swap.
      Value is a semicolon (;) separated list of swapping file systems.
   ```
Examples:
   ```
    - swap=@UUID=ABCDEFG[@options]
      to mount a swap partition
    - swap=/path/to/file.swp[@device][@options]
      to mount a swap file that resides in the root partition
    - swap=/path/to/file.swp@UUID=ABCDEFG@sw,prio=10
      to mount a swap file that resides in the partition
   ```
Also option swap may be specified multiple times in the kernel cmdline.

New feature:

5- conf=c1,c2,...,cN

The superuser must copy the file
/usr/docs/initramfs-tools/cmdline_conf

to

/etc/initramfs-tools/conf.d 

and configure it according to the actual install.

Internal initramfs variable is also named conf.

Examples:

- conf=c1,c2,...,cN

Option conf may be specified multiple times in kernel cmdline.

New feature:

6- modules

This kernel cmdline option specifies a list of modules to be loaded 
early in the initramfs-tools scripts.
   ```
     - Value is a semicolon (;) separated list of module names followed
      with their options.
    - modules=m1;m2\040opt=v;m3
    - nomodules
      clear the modules variable that contains the value of previously
      specified modules.
   ```
      
Examples:

    - modules=overlay;zram\040num_devices=0;loop;ext2

************************************************************************

Live has been checked with local disks, more testing is needed to use 
NFS.

************************************************************************

It's advised to specify paths and their corresponding device using the form:
path@device
therefore initramfs will mount automatically the device in a subdirectory of
/run/initramfs/mnt

The physical root device is exposed in directory
/run/initramfs/mnt/rootfs
so it will be available for other purposes.

************************************************************************
Following the initramfs-tools rules, the device names may be:
   ```
- /path/name
  of the real block device name.
- /path/name
  of a symbolic link to the real block device name.
- LABEL=* | UUID=* | PARTLABEL=* | PARTUUID=*
- tmpfs
  to use when liveupp resides in tmpfs.
- /
  when mountrd file path is relative to the rootmnt directory.
   ```
************** Example kernel cmdline for rootdir  *********************
   ```
root=UUID=cb54628e-5d1a-424b-997a-01f92e7b191f rw \
rootdir=/LneTMINI64 \
mountrd=@/
   ```
************************************************************************

************** Example kernel cmdline for mountrd  *********************
   ```
root=UUID=cb54628e-5d1a-424b-997a-01f92e7b191f rw \
mountrd=/etc/m.fstab
   ```
************************************************************************

*** Example /etc/initrd.fstab file *************************************
   ```
# /etc/initrd.fstab: static file system information.
# A custom initrd will mount these entries
#   when the option "mountrd" is specified in kernel cmdline.
# <file system>	<mount point>	<type>	<options>	<dump>	<pass>

/${DFLAVOUR}@${HDISK}1	/boot	auto	bind,ro	0	1
/LneTKernel/firmware@${HDISK}1	/lib/firmware	auto	bind,ro	0	1
/LneTKernel/$(uname\040-r)@${HDISK}1	/lib/modules/$(uname\040-r)	auto	bind,ro	0	1
tmpfs	/var/log	tmpfs	mode=755,nosuid,relatime,rw,size=100%	0	1
tmpfs	/var/tmp	tmpfs	nodev,nosuid,relatime,rw,size=100%,mode=1777	0	1
tmpfs	/tmp	tmpfs	nodev,nosuid,relatime,rw,size=100%,mode=1777	0	0
${HDISK}3	swap	swap	sw	0	0
/swapfile.swap@${HDISK}2	none	auto	sw	0	0
   ```
************************************************************************

*** Example kernel cmdline for a non persistent Live on tmpfs **********
   ```
root=UUID=3e8630d5-6b47-4191-8d22-d5e91ace1220 \
liveupp=@tmpfs \
livelow=/LneTMINI64/00filesystem.squashfs,/LneTMINI64/01live.d
   ```
************************************************************************

*** Example kernel cmdline for a non-persistent Live OS ****************
*** over a disk directory    *****************************************
   ```
root=UUID=cb54628e-5d1a-424b-997a-01f92e7b191f rw mountrd=@/ \
liveupp=/LiveCOW@@cleanup \
livelow=/livefsdir/filesystem.squashfs,/livefsdir/customizations.d
   ```
************************************************************************

*** Example kernel cmdline for a persistent Live OS ********************
*** on several disk partitions *****************************************
   ```
root=PARTUUID=3e8630d5-02 rw mountrd=@/ \
liveupp=/LiveCOW \
livelow=/livefsdir/filesystem.squashfs@PARTUUID=3e8630d5-01 \
livelow=/livefsdir/customizations.d@PARTUUID=3e8630d5-01
   ```
************************************************************************
