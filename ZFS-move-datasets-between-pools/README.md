### TrueOS CG Move ZFS Datasets Between ZFS Pools

#### Preface:

This community guide article describes how to list existing **snapshots** and **transfer** them from one **ZFS pool** to a newly created one.


#### Requirements:

+ TrueOS correctly installed and booted with ZFS
+ A backup of all your data in case something goes wrong


#### Steps:

+ First show the status, scrub the pool and list all datasets and their snapshots for good overview. The scrub can take a few minutes or longer depending on your data.
> sudo zpool scrub _old-pool-name_
> sudo zpool status
> sudo zfs list -t all
> sudo mount | sort

+ Create a new ZFS pool in case you added physical disks, or just in case you want a different RAID level, or the vdevs in your current ZFS pool are too many (security constraint). We assume a new physical disk like e.g. /dev/adaX exists. Choose a sensible name for the dataset (not "another dataset"), like e.g. _home-username_ if its a home directory you will transfer later.
> geom DISK status -ga
> zpool create _new-pool-name_ /dev/adaX
> sudo zfs create _new-pool-name/another-dataset_

+ Set the compression in advance or things will occupy much space. And set, that the pool itself cannot be mounted, adding a bit of safety.
> sudo zfs set canmount=off _new-pool-name_
> sudo zfs set compression=lz4 _new-pool-name/another-dataset_

+ Create a snapshot of the to-be-transferred dataset in the old pool. The snapshot should not occupy much space. It does not matter if the dataset is mounted somewhere. Then send the snapshot to the new pool into an existing dataset (or into the pool directly). **Hint:** stop or close all write intensive applications before transfer!
> sudo zfs snap _old-pool-name/whatever-dataset@random-temp-name_
> sudo zfs send _old-pool-name/whatever-dataset@random-temp-name_ | sudo zfs recv _new-pool-name/another-dataset-optional_

+ Now "rollback" the snapshot in the new pool to its underlying dataset named "another-dataset". That will do basically nothing in most cases, but it seems to be the most correct way to do things.
> sudo zfs rollback _new-pool-name/another-dataset@random-temp-name_

+ Afterward, a mountpoint for the new dataset in the new pool should be defined so the dataset gets mounted in the future on every boot. The zpool base gets no mountpoint though. The mount point "/whatever/you/want" could be _/usr/home/michael_, but make sure it does not actually exist yet or things will go bad!
> sudo zfs set mountpoint=none _new-pool-name_
> sudo zfs set mountpoint=_/whatever/you/want new-pool-name/another-dataset_

+ After a time (few minutes, hours or days) of **testing and verification** the data in the new pool is O.K. The old dataset in the old pool can be deleted including the snapshot. There may also be an _-r_ option which is a bit dangerous though, which is why its not listed.
> sudo zfs destroy _old-pool-name/whatever-dataset@random-temp-name_
> sudo zfs destroy _old-pool-name/whatever-dataset_

+ Finally list the compression ratio of the new poll and verify its O.K. Also the pool status and the datasets should be listed and verified. Also a scrub of the new pool can be done, although its typically not needed, since all data was just written during zfs send.
> sudo zfs get used,compressratio,compression,logicalused new-pool-name/another-dataset
> sudo zpool status
> sudo zfs list -t all
> sudo zpool scrub _new-pool-name_

**That's it.** You moved a dataset from one pool to a different one on the same machine!
