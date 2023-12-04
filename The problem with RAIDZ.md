# The problem with RAIDZ or why you probably won't get the storage efficiency you think you will get.
As a ZFS rookie, I struggled a fair bit to find out what settings I should use for my Proxmox hypervisor. Hopefully this could help other ZFS rookies. 

## TLDR
RAIDZ is only great for sequential reads and writes of big files. An example of that would be a fileserver that mostly hosts big files. 

## Introduction and glossary

Before we start, we have to learn about some ZFS glossary. These are important to understand the examples later on.

sector size:
older HDDs used to have a sector size of 512b, while newer HDDs have 4k sectors. SSDs can have even bigger sectors, but their firmware controllers are mostly tuned for 4k sectors. There are still enterprise HDDs that come with 512e, where the "e" stands for emulation. These are not 512b but 4k drives, they only emulate to be 512. For this whole text, I assume that we have drives with 4k sectors.

ashift:
ashift sets the sector size, ZFS should use. ashift is a power of 2, so setting ashift=12 will result in 4k. Ashift must match your drive's sector size. Extremely likely this will be 12 and also automatically detected.

dataset:
A dataset is inside a pool and is like a file system. There can be multiple datasets in the same pool, and each dataset has its own settings like compression, dedup, quota, and many more. They also can have child datasets that by default inherit the parent's settings. Datasets are useful to create a network share or create a mount point for local files. In Proxmox, datasets are mostly used locally for ISO images, container templates, and VZdump backup files.

zvol:
zvols or ZFS volumes are also inside a pool. Rather than mounting as a file system, it exposes a block device under /dev/zvol/poolname/dataset. This allows to back disks of virtual machines or to make it available to other network hosts using iSCSI. In Proxmox, zvols are mostly used for disk images and containers.

recordsize:
Recordsize applies to datasets. ZFS datasets use by default a recordsize of 128KB. It can be set between 512b to 16MB (1MB before openZFS v2.2).

volblocksize:
Zvols have a volblocksize property that is analogous to recordsize.
Since openZFS v2.2 the default value is 16k, while Proxmox 8.1 still uses 8k as default.

Now with that technical stuff out of the way, let's look at real-life examples.
