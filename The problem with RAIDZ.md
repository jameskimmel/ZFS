# The problem with RAIDZ or why you probably won't get the storage efficiency you think you will get.
As a ZFS rookie, I struggled a fair bit to find out what settings I should use for my Proxmox hypervisor. Hopefully, this could help other ZFS rookies. 
This text focuses on Proxmox, but it generally applies to all ZFS systems. 

## TLDR
RAIDZ is only great for sequential reads and writes of big files. An example of that would be a fileserver that mostly hosts big files. 
For VMs or iSCSI, RAIDZ will not get you the storage efficiency you think you will get and also will perform badly. 

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
Or you could also say, that to store a 128k file, we need 128k / 8k data blocks = 16 stripes.  
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
It doesn't apply to you, if you have a 3-wide RAIDZ1 and only write files where the size is a multiple of 8k. For huge files like pictures, movies, and songs, the efficiency loss for not being exactly a multiple of 8k, the loss gets negligible.

## ZVOL and volblocksize
For Proxmox we mostly don't use datasets though. We use VMs with RAW disks that are stored on a Zvol.  
For Zvols and their fixed volblocksize, it gets more complicated.  

In the early days, the default volblocksize was 8k and it was recommended to turn off compression.
Nowadays, it is recommended to enable compression and the current default is 16k since v2.2. Still not the default in Proxmox though. Some people in the forum recommend going as high as 64k on SSDs.

In theory, you wanna have writes that exactly match your volblocksize.  
For MySQL or MariaDB, this would be 16k. But because you can't predict compression, and compression works very well for stuff like MySQL?, you don't actually get a 16k write (or is it stored on a 16k volblocksize, but that blocksize itself can be compressed?)?.  
A larger volblocksize is good mostly sequential workloads and can gain compression efficiency.  
Smaller volblocksize is good for random workloads, has less IO amplification, less fragmentation, but will use more metadata and have worse space efficiency.  

For the following examples, we assume ashift = 12 or 4k, because that is the default for modern drives. 
We look at the different volblocksizes and how they behave on different pools. 

### volblocksize 16k
This is the default size for openZFS since 2.2.  

#### RAIDZ1 with 3 drives
With 3 drives, we get a stripe 3 drives wide.  
Each stripe has four two 4k data blocks and one 4k parity block.  
For a volblock of 16k, we need two stripes (16k/8k).  
Each stripe has two 4k data blocks, in total 16k. 
Each stripe has one 4k parity block, in total 8k. 
That gets us to 24k in total to store 16k. 
Storage efficiency is 66%, as expected.

#### RAIDZ1 with 4 drives
With 4 drives, we get a stripe 4 drives wide.  
Each stripe has four three 4k data blocks and one 4k parity block.  
For a volblock of 16k, we need 1.33 stripes (16k/12k).  
The first stripe has three 4k data blocks, in total 12k.  
The first stripe also has one 4k block for parity. 
The second stripe has one 4k data block.  
The second stripe also has one 4k block for parity.  
In total we have four 4k data blocks and two 4k parity blocks.  
That gets us to 24k in total to store 16k.  
Storage efficiency is 66%. 
We would have naturally expected a storage efficiency of 75%!  

#### RAIDZ2 with 6 drives

