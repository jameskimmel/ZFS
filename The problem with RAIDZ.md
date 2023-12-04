# The problem with RAIDZ or why you probably won't get the storage efficiency you think you will get.
As a ZFS rookie, I struggled a fair bit to find out what settings I should use for my Proxmox hypervisor. Hopefully this could help other ZFS rookies. 
This text focuses on Proxmox, but it generally applies to all ZFS systems. 

## TLDR
RAIDZ is only great for sequential reads and writes of big files. An example of that would be a fileserver that mostly hosts big files. 
For VMs or iSCSI, RAIDZ will not get you the storage efficiency you think you will get and also perform bad. 

## Introduction and glossary
Before we start, we have to learn about some ZFS glossary. These are important to understand the examples later on.

### sector size:
older HDDs used to have a sector size of 512b, while newer HDDs have 4k sectors. SSDs can have even bigger sectors, but their firmware controllers are mostly tuned for 4k sectors. There are still enterprise HDDs that come with 512e, where the "e" stands for emulation. These are not 512b but 4k drives, they only emulate to be 512. For this whole text, I assume that we have drives with 4k sectors.

### ashift:
ashift sets the sector size, ZFS should use. ashift is a power of 2, so setting ashift=12 will result in 4k. Ashift must match your drive's sector size. Extremely likely this will be 12 and also automatically detected.

### dataset:
A dataset is inside a pool and is like a file system. There can be multiple datasets in the same pool, and each dataset has its own settings like compression, dedup, quota, and many more. They also can have child datasets that by default inherit the parent's settings. Datasets are useful to create a network share or create a mount point for local files. In Proxmox, datasets are mostly used locally for ISO images, container templates, and VZdump backup files.

### zvol:
zvols or ZFS volumes are also inside a pool. Rather than mounting as a file system, it exposes a block device under /dev/zvol/poolname/dataset. This allows to back disks of virtual machines or to make it available to other network hosts using iSCSI. In Proxmox, zvols are mostly used for disk images and containers.

### recordsize:
Recordsize applies to datasets. ZFS datasets use by default a recordsize of 128KB. It can be set between 512b to 16MB (1MB before openZFS v2.2).

### volblocksize:
Zvols have a volblocksize property that is analogous to recordsize.
Since openZFS v2.2 the default value is 16k, while Proxmox 8.1 still uses 8k as default.

Now with that technical stuff out of the way, let's look at real-life examples.

## Dataset example
Let's look at an example of a dataset with the default recordsize of 128k and how that would work. We assume that we want to store a file 128k in size (after compression).

For a 3-disk wide RAIDZ1, the total stripe width is 3.

One stripe has 2 data blocks and 1 parity block. Each is 4k in size.  
So one stripe has 8k data blocks and a 4k parity block.  
To store a 128k file, we need 128k / 4k = 32 data blocks.  
To store 32 data blocks, we need 32 data blocks / 2 data blocks per stripe = 16 stripes.   
Or you could also say, to store a 128k file, we need 128k / 8k data blocks = 16 stripes.  
Each of these stripes consists of two 4k data blocks and a 4k parity block.  
In total, we store 128k data blocks (16 stripes * 8k data blocks) and 64k parity blocks (16 stripes * 4k parity).  
Data blocks + parity blocks = total blocks  
128k + 64k = 192k.  
That means we write 192k in blocks to store a 128k file.  
128k / 192k = 66.66% storage efficiency.  

This is a best-case scenario. Just like one would expect from a 3-wide RAID5 or RAIDZ1, you "lose" a third of storage.

Now, what happens if the file is smaller than the recordsize of 128? A 20k file?

We calculate the same thing for our 20k file.  
20k divided by 8k (2 data parts, each 4k) = 2.5 stripes. Half-data stripes are impossible. So we need 3 stripes to store our data.

The first stripe has 8k data blocks and a 4k parity block.  
The second stripe has 8k data blocks and a 4k parity block.  
The third stripe is special.  
We already saved 16k of data in the first two blocks, so we only have 4k data left to save.  
That is why the third stripe has a 4k data block and a 4k parity block.  
Now the efficiency has changed. If we calculate all together, we wrote 20k data blocks, 12k parity blocks, and one 4k padding block.  
We wrote 32k to store a 20k file.  
20k / 32k = 62.5% storage efficiency.  

This is not what you intuitively would expect. What happens if the situation gets even worse and we wanna save a 4k file?

We calculate the same thing for a 4k file.  
We simply store a 4k data block on one disk and one parity block on another disk. In total, we wrote a 4k data block and a 4k parity block.  
We wrote 8k in blocks to store a 4k file.  
4k / 8k = 50% storage efficiency.  

This is the same storage efficiency we would expect from a mirror.
It does't apply to you, if you have a 3-wide RAIDZ1 and only write files where the size is a multiple of 8k. For huge files like pictures, movies, and songs, the efficiency loss for not being exactly a multiple of 8k, the loss gets negligible.




