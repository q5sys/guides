### TrueOS CG Turn a 1-Disk ZFS Root-Pool Into a 2-Disk Mirrored Root-Pool

#### Preface:

This community guide article describes how to create a **root mirror disk** of a **TrueOS installation** with on-board utilities. The guide is useful when only one disk drive was available during TrueOS installation, but a *bootable* mirrored ZFS installation of TrueOS on two disks (RAID 1) is desired. On hardware destruction **TrueOS can be booted from either remaining good disk** drive then, with minimal downtime incured.


#### Requirements:

+ TrueOS correctly installed and booted with ZFS
+ A second hard disk (either HDD, SSD or SSHD) installed
+ A backup of all your data in case something goes wrong


#### Steps:

+ First list all physical devices, the gpart partition data and then show the ZFS pool status and associated mount points for a good system overview.
> geom DISK status -ga
> gpart show -r 
> zpool status
> zpool status _root-pool-name_
> mount | sort

+ In this example TrueOS was installed with default parameters (no partition changes), enabled GELI encryption and the standard FreeBSD bootloader (no multiboot with GNU/Linux or Windows), with GPT as the partitioning table. As a second step, the disk layout must be written out with _gpart_. **Note:** adding a smaller disk than the original one is not covered by this how-to. The _gpart_ utility on FreeBSD is also quite different from what people are used to from other operating systems such as GNU/Linux. It is also assumed ada0 is our first disk and ada1 our second disk.
> sudo gpart backup _ada0_ | sudo gpart restore _ada1_
> gpart status

+ Next the bootcode (MBR and GPT, plus GELI information) must be written out to the new mirror disk drive. **Note**: the bootloader is not installed with _installboot_, however. To be 100% sure the first partition (BOOT), from the disk TrueOS is installed on, can also be copied with diskdump (dd). It is recommended to flush all OS buffers/caches afterwards.
> sudo gpart bootcode -b /boot/pmbr -p /boot/gptzfsboot -i 1 _ada1_
> sudo dd if=_/dev/ada0p1_ of=_/dev/ada1p1_
> sudo sync

+ In TrueOS the installer uses a _/dev/label/swap0.eli_ device to reference the SWAP partition. In this example SWAP will be turned of and then replaced. For the crypto partitions two (ZFS) and three (SWAP) we use AES in XTS mode. **Note**: TrueOS does not use a password file by default, since the GELI boot implementation with them is yet not advised in the FreeBSD handbook.
> sudo swapoff _/dev/label/swap0.eli_
> sudo geli init -b -s 512 _ada1p2_
> sudo geli attach _ada1p2_
> sudo geli init -b -s 512 _ada0p3_
> sudo geli attach _ada0p3_
> sudo geli init -b -s 512 _ada1p3_
> sudo geli attach _ada1p3_
> geom DISK status -ga
> geli list
> // The corresponding change in _/etc/fstab_ has to be made!
> sudo swapon _/dev/ada0p3.eli_
> swapinfo

+ Next, partition two (ZFS) on the second new disk can be added to the ZFS root pool. **Note**: The first partition (BOOT), containing all the boot information and partitions from the disk drive, and the third partition (SWAP) will not be mirrored.
> sudo zpool attach _root-pool-name_ mirror _ada0p2.eli_ _ada1p2.eli_
> zpool status -v

+ In this step, the correct boot flag _GELIBOOT_ must explicitly be set, to mark the second partition (ZFS) on the new mirror disk as bootable with GELI encryption. In addition to that, we recommend to add the GEOM mirror module and set canmount for the second partition's (ZFS) file system. **Note**: The name of the datasets (e.g. _12.0-CURRENT-up-20180213_152723_) may vary of course. The consecutive scrub operation is strongly recommended can take a few minutes or longer depending on your amount of data. Also the verbose boot option must be enabled in the bootloader configuration loader.conf(5) to unlock SWAP during boot.
> sudo geli configure -g _/dev/ada1p2.eli_
> geli list
> sudo echo 'geom_mirror_load="YES"' > /boot/loader.conf
> sudo echo 'verbose_loading="YES"' > /boot/loader.conf
> zfs list -t all
> sudo zfs set canmount=on _root-pool-name/ROOT/initial_
> sudo zfs set canmount=on _root-pool-name/ROOT/12.0-CURRENT-up-20180213_152723_
> sudo zpool scrub _zfspool-name_
> zpool status

+ Finally the kernel disks parameters and _beadm_ data should be listed and checked for correctness. Especially the boot dataset parameter must be set correctly to access ZFS data in _/boot_ during the boot process.
> kenv | egrep "kernel|dev=...|mount|active|root=" | sort
> kenv | egrep "boot" | grep fs
> beadm list

**That's it.** You now have a 1:1 copy and ZFS mirror (RAID1) of your hard disk with TrueOS installed.
