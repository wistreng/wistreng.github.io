---
layout: post
title: File Management System (Linux 2.3.1)
date: 2018-02-01 10:01:27 +1300
tags: Linux
picture_frame: shadow
---

*Linux's most traditional disk filesystem USES EXT2. So to understand the file system, you have to start by knowing EXT2! The file system is created on the hard disk, so we need to understand the physical components of the hard disk which was covered in Chapter 0, and the partitioning of the disk is covered in Chapter 3, so these two sections will be reviewed quickly below. The focus is on **inodes**, **blocks**, **superblocks**, and other basic parts of file systems.*

<!--more-->


## EXT2 - Linux second extended file system (ext2fs)


### Hard disk composition and partition


First of all, explain the physical composition of the disk:

- A circular disk (the part that primarily records data);
- The mechanical arm, with the disk readout head on the mechanical arm (erasable data on disk);
- The spindle motor rotates the disk, allowing the robotic arm's readhead to read and write data on the disk.

We know from the above that data storage and reading focus on the disk, and the physical composition of the disk is (assuming that the disk is a single disk) :

![1](http://cn.linux.vbird.org/linux_basic/0130designlinux_files/harddisk.jpg)

- Sector is the smallest physical storage unit with 512 bytes per Sector;
- Make a circle of sectors. That is, magnetic columns are the smallest unit of partition.
- The first sector is the most important, including (1) the main boot record (MBR) and partition Table (Partition table). MBR has 446 Bytes, while Partition Table has 64 bytes.


The file names of disks for various interfaces in Linux are as follows:


- /dev/sd [A-P][1-15] : file name of `SCSI`, `SATA`, `U disk`, `Flash disk` and other interfaces;
- /dev/hd[A-D][1-63] : file name of disk for `IDE` interface;


After reviewing the physical composition, let's review the disk partitioning! A partition tells the operating system that "the area that my disk can access in this partition is the area between the A and B pillars" so that the operating system knows that it can read/write/search file data in the specified block. In other words, disk partitioning means specifying the start and end magnetic pillars of the split slot.


So where is the magnetic column range specified for the split slot recorded? That's in the first sector split table! However, since the segmentation table only has 64Bytes, it can only record four pieces of segmentation slots at most, which are called primary or extended segmentation slots. The extended allocation slots can also be re-segmented into logical segments, while those that can be formatted only have primary segmentation and logical segmentation.


Finally, let's explain the definition of segmentation in Chapter 3:


- Primary split and extended allocations can have up to four strokes (hard disk - limit)
- There can be at most one extension allocation (operating system limit)
- Logical segmentation is the continuous cutting slot by extension allocation.
- After it can be formatted, the data access as a partition slot for the main segmentation and logical segmentation. Extended allocation cannot be formatted;
- The number of logical partitions varies depending on the operating system. In Linux, IDE drives have up to 59 logical partitions (5 through 63) and SATA drives 11 logical partitions (5 through 15).


### File system features


We all know that a disk partition needs to be formatted before the operating system can use the partition slot. Why do you need formatting? This is because file properties/permissions are not the same for each operating system, and in order to store the data needed for these files, the split slot needs to be formatted into a "filesystem" that the operating system can leverage.


We also know that each operating system can use different file systems. For example, Microsoft operating system before Windows 98 mainly used FAT (or FAT16) file system, Windows 2000 after the version of the so-called NTFS file system, and Linux after the orthodox file system is Ext2 (Linux Second Extended File System (ext2FS) one. Also, by default, the Windows operating system does not recognize Ext2 on Linux.


In traditional disk and filesystem applications, a partition is simply a partition that can be formatted as a filesystem. But thanks to new technologies, such as LVM and Software raid, which we often hear about, it is possible to format a split slot into multiple file systems (such as LVM) or to combine multiple split slots into a single file system (LVM, RAID)! So, while we're no longer formatting against partition, we can usually call a mounted data file system rather than a partition slot.


So how does the file system work? This has to do with operating system file data. The file data of newer operating systems typically contain a number of properties in addition to the actual contents of the file, such as the Linux operating system's file permissions (RWX) and file properties (owner, group, time parameters, and so on). File systems typically store these two pieces of data in separate blocks, with permissions and attributes placed in inodes and actual data in data block blocks. In addition, there is a superblock that records the entire file system, including the total amount of inodes and blocks, usage, residual, and so on.


Each inode and block are numbered, and the meaning of these three data can be briefly explained as follows:


- Superblock: record the overall information of the filesystem, including the - total amount of inode/block, usage, residual amount, and filesystem format and - related information;
- Inode: Records the attributes of the file. A file occupies an inode and - records the block number of the data of the file.
- Block: The actual recording of the contents of the file, if the file is too - large, will take up more than one block.


Since each inode and block are numbered, each file occupies an inode, and the inode contains the block number in which the file data is placed. Therefore, we can know that if we can find the inode of the file, we will naturally know the block number of the data placed in the file, and of course we can read the actual data of the file. This is a more efficient method, because our disk can read and fetch all the data in a short time, reading and writing performance is better.


We will inode with block, block diagram to explain, as shown in the figure below, the file system to format the inode and block, block, assume that a certain file attributes and access data is placed in the inode 4 (below small box), and the inode recorded the actual placement of the file data is 2, 7, 13, 15 of the four block number, at this time our operating system will be able to arrange the reading order of the disk, accordingly can breath will four block content read it! The data is read as indicated by the arrows in the figure below.


![inode/block](http://cn.linux.vbird.org/linux_basic/0230filesystem_files/filesystem-1.jpg)


This method of data access is called indexed allocation. Are there any other idiomatic file systems to compare? B: Yes, that's our standard flash drive (flash drive). The file system used for the flash drive is usually FAT format. The FAT file system does not have inodes, so FAT has no way to read all the blocks for the file in the first place. Each block number is recorded in the previous block, and it reads like this:


![FAT](http://cn.linux.vbird.org/linux_basic/0230filesystem_files/filesystem-2.jpg)


In the figure above, we assume that the file's data is written in sequence to the four block Numbers 1->, 7->, 4->, 15, but the file system has no way of knowing the number of the four blocks in one sitting. It has to read the blocks one by one before it knows where the next block is. If the block of writing to the same file is too dispersed, our disk read head will not be able to read all the data in one circle on the disk, so the disk will have to turn several times to read the full contents of the file!


Linux's EXT2 file system (inode) :


In Chapter 6, we introduced that Linux files contain a lot of permissions and properties in addition to the original data content, which are intended to protect the privacy of the data owned by each user. In the previous section, you can see the possible inode/block/superblock in filesystem. Why do we talk about this? The standard Linux file system, Ext2, USES inodes.


As mentioned in the previous section, the contents of the inode record the permissions and related properties of the file, while the block block records the actual contents of the file. Moreover, the inode and block are scheduled from the beginning of the file system, and unless the file system is reformatted (or the file system size is changed using commands such as resize2fs), the inode and block are fixed and do not change. But if you think about it, if my filesystem is hundreds of GIGABYtes, it would be unwise to put all the inodes and blocks together, because the number of inodes and blocks is too large to manage.


For this reason, the Ext2 file system is basically divided into block groups during formatting, each block group having a separate inode/block/superblock system. It felt like when we were soldiers, a battalion was divided into several companies, each company had its own contact system, but eventually the battalion returned the most accurate information to the general! It's easier to manage a group like this! In general, Ext2 is formatted to look something like this:

Ext2 file system schematic

FIG. 1.3.1 Schematic Diagram of ext2 file System (Note 1)


The front in the overall planning, the file system has a boot sector (boot sector), the boot sector can install to start the management procedure, this is a very important design, because in this way we can install different startup management program to an individual file system to the front, instead of covering the whole of the hard disk only MBR, so also can create multiple boot environment! The six main contents of each block group are described as follows: