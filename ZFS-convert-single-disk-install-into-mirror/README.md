### Convert a 1-Disk ZFS Root-Pool Into a 2-Disk Mirrored Root-Pool

#### Preface:

This community guide describes how to create a **root mirror disk** of a **TrueOS installation** with on-board utilities. This guide helps configure a *bootable* mirrored ZFS installation of TrueOS on two disks (RAID 1) when only one disk drive was available during a TrueOS installation. In the event of hardware destruction **TrueOS can be booted from either remaining good disk** drive with minimal downtime.


#### Requirements:

+ TrueOS correctly installed and booted with ZFS.
+ A second hard disk (either HDD, SSD, or SSHD) installed.
+ A backup of all system data in case something goes wrong.


#### Steps:

+ First, list all physical devices, the gpart partition data, and then show the ZFS pool status and associated mount points for a good system overview:

```
geom DISK status -ga
gpart show -r 
zpool status
zpool status root-pool-name
mount | sort
```

In the above example TrueOS was installed with no partition changes, GELI encryption enabled, and the standard FreeBSD bootloader with no GNU/Linux or Windows multiboot configuration. GPT is used for the partitioning table.

+ Now the disk layout must be written out with _gpart_. **Note:** adding a smaller disk than the original one is not covered by this guide. The _gpart_ utility on FreeBSD is also quite different from other operating systems such as GNU/Linux. It is also assumed *ada0* is the first disk and *ada1* is the second disk:

```
sudo gpart backup ada0 | sudo gpart restore ada1
gpart status
```

+ Next, the MBR, GPT, and GELI bootcode information must be written out to the new mirror disk drive. **Note**: the bootloader is not installed with _installboot_. To ensure this process works, the first partition (BOOT) on the disk TrueOS is installed on can also be copied with diskdump (*dd*). It is recommended to flush all OS buffers/caches afterwards.

```
sudo gpart bootcode -b /boot/pmbr -p /boot/gptzfsboot -i 1 ada1
sudo dd if=/dev/ada0p1 of=/dev/ada1p1
sudo sync
```

+ In TrueOS, the installer uses a _/dev/label/swap0.eli_ device to reference the SWAP partition. In the next example SWAP is turned off and replaced. AES in XTS mode is used for crypto partitions two (ZFS) and three (SWAP). **Note**: TrueOS does not use a password file by default. A GELI boot implementation with these files is not yet recommended in the FreeBSD handbook.

```
sudo swapoff /dev/label/swap0.eli
sudo geli init -b -s 512 ada1p2
sudo geli attach ada1p2
sudo geli init -b -s 512 ada0p3
sudo geli attach ada0p3
sudo geli init -b -s 512 ada1p3
sudo geli attach ada1p3
geom DISK status -ga
geli list
// The corresponding change in /etc/fstab has to be made!
sudo swapon /dev/ada0p3.eli
swapinfo
```

+ Next, add partition two (ZFS) on the second disk to the ZFS root pool. **Note**: The first partition (BOOT) containing all the boot information and partitions from the disk drive and the third partition (SWAP) will not be mirrored.

```
sudo zpool attach root-pool-name mirror ada0p2.eli ada1p2.eli
zpool status -v
```

+ Set the boot flag for the second partition (ZFS) on the new mirror disk. _GELIBOOT_ must explicitly be set as bootable with GELI encryption. It is also recommended to add the GEOM mirror module and set *canmount* for the second partition's (ZFS) file system. **Note**: Dataset name (_12.0-CURRENT-up-20180213_152723_) may vary. The consecutive scrub operation is strongly recommended but can take a few minutes or longer depending on the amount of data. The verbose boot option must be enabled in the bootloader configuration **loader.conf(5)** to unlock SWAP during boot.

```
sudo geli configure -g /dev/ada1p2.eli
geli list
sudo echo 'geom_mirror_load="YES"' > /boot/loader.conf
sudo echo 'verbose_loading="YES"' > /boot/loader.conf
zfs list -t all
sudo zfs set canmount=on root-pool-name/ROOT/initial
sudo zfs set canmount=on root-pool-name/ROOT/12.0-CURRENT-up-20180213_152723
sudo zpool scrub zfspool-name
zpool status
```

+ Finally, check the kernel disks parameters and _beadm_ data. Most importantly, the *boot* dataset parameter must be set correctly to access ZFS data in _/boot_ during the boot process.

```
kenv | egrep "kernel|dev=...|mount|active|root=" | sort
kenv | egrep "boot" | grep fs
beadm list
```

**That's it.** There is now a 1:1 copy and ZFS mirror (RAID1) of the hard disk that has TrueOS installed.
