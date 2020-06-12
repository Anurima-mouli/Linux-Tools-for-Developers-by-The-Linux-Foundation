Linux Tools for Developers
==========================

by The Linux Foundation

# Week 2

#### Title: Files and Filesystems

### Overview

* Linux works with many **types** of **filesystems** both native filesystems like **ext2**, **ext3** and **ext4**
* **umask** --> controls the default access rights on files
* **Journaling filesystems** which all modern file systems are, which have extended capabilities and can recover far more quickly in case of some kind of disaster

###### Types of Files

* There are a limited number of basic file types, each of which are displayed in the following directory listing	
	```bash
	$ ls -lF
	total 8
	brw-r--r-- 1 coop coop 200, 0 Mar  9 15:07 a_block_device_node
	crw-r--r-- 1 coop coop 200, 0 Mar  9 15:06 a_character_device_node
	drwxrwxr-x 2 coop coop   4096 Mar  9 15:05 a_directory/
	prw-rw-r-- 1 coop coop      0 Mar  9 15:04 a_fifo|
	-rw-rw-r-- 1 coop coop   1601 Mar  9 15:04 a_file
	lrwxrwxrwx 1 coop coop      4 Mar  9 15:06 a_symbolic_link_to_directory -> /usr/
	lrwxrwxrwx 1 coop coop      6 Mar  9 15:10 a_symbolic_link_to_a_file -> a_file
	srwxrwxr-x 1 coop coop      0 Mar  9 15:09 mysock=
	```
* The first character in the listing shows the type of file
	* **`-`** --> Normal File
	* **`d`** --> Directory
	* **`l`** --> Symbolic link
	* **`p`** --> Named pipe (FIFO) --> If it's a symbolic link, there's an "l"
	* **`s`** --> Unix domain socket --> These work similarly to pipes, but they work through a networking infrastructure
	* **`b`** --> Block device node --> such as a hard disk or a CD-ROM, etc
	* **`c`** --> Character device node --> such as an audio card, a line printer, etc., where you send streams of data byte by byte
* The file utility program can be used to ascertain file types, like
	```bash
	$ file *
	acpitool:             ELF 64-bit LSB executable, AMD x86-64, version 1 (SYSV),
	                          for GNU/Linux 2.6.9, dynamically linked
	                          (uses shared libs), for GNU/Linux 2.6.9, not stripped
	atob:                 C shell script text executable
	awkit.sh:             Bourne-Again shell script text executable
	blank:                JPEG image data, JFIF standard 1.01
	cdc.ORIG3:            Bourne-Again shell script text executable
	copy_configs.sh:      ASCII text
	gtk-recordMyDesktop:  python script text executable
	gztobz2:              Bourne-Again shell script text executable
	html2ps:              shell archive or script for antique kernel text
	```
* There's big difference between block devices and character devices, is that
	* **Block Devices** always work (not surprisingly) in blocks of data
	* **Character Devices** work in one byte at a time
* In any case, you usually only find these **device nodes** in the `/dev` directory, they don't belong anywhere else by convention


##### Permissions and Access Rights

###### Changing Permissions and Ownership

* Changing file permissions is done with **chmod**
* Changing file ownership is done with **chown**
* Changing the group is done with **chgrp**
* There are a number of different ways to use **chmod**
	* For instance, to give the **owner** and **world** execute permission, and remove the **group** write permission
		```bash
		$ ls -l a_file
		-rw-rw-r-- 1 coop coop 1601 Mar  9 15:04 a_file
		$ chmod uo+x,g-w a_file
		$ ls -l a_file
		-rwxr--r-x 1 coop coop 1601 Mar  9 15:04 a_file
		```
		* where 
			* **u** stands for user (owner)
			* **o** stands for other (world)
			* **g** stands for group
	```ruby
	* This kind of syntax can be difficult to type and remember, so one often uses a shorthand which lets you set all the permissions in one step
	* This is done with a simple algorithm, and a single digit suffices to specify all three permission bits for each entity. 
	* This digit is the sum of
		* 4 if read permission is desired - **r**
		* 2 if write permission is desired - **w**
		* 1 if execute permission is desired - **x**
	* Thus, 7 means read/write/execute, 6 means read/write, and 5 means read/execute
	```
* When you apply this when using **chmod**, you have to give three digits for each degree of freedom, like
	```bash
	$ chmod 755 a_file
	$ ls -l a_file
	-rwxr-xr-x 1 coop coop 1601 Mar  9 15:04 a_file
	```
* Sample Commands
	1. Changing the group ownership of the file is as simple as
		```bash
		$ chgrp aproject a_file
		```
	1. Changing the ownership is as simple as
		```bash
		$ chown coop a_file
		```
	1. You can change both at the same time with
		```bash
		$ chown coop.aproject a_file
		```
		* where you separate the owner and the group with a period
	1. All three commands mentioned above can take an -R option, which stands for recursive
		* Example
			```bash
			$ chown -R coop.aproject .
			$ chown -R coop.aproject subdir
			```
		* will **change** the **owner** and **group** of all files in the **current directory** and **all its subdirectories** in the first command, and in **subdir** and all its subdirectories in the second command.


#### Title: Linux Filesystems

### Filesystems and VFS

###### VFS ( Virtual File System )

> **Reiser** filesystem was the first journaling implementation in the Linux kernel

* Stands between the actual filesystem and the applications that want to use files
* Applications do not need to know about what kind of filesystem the data's on or what the physical media is or the hardware, and you can also use **N**etwork **F**ile **S**ystems, such as **NFS**
* Whenever an application needs to access a file it makes a call to the **Virtual Filesystem**, and then that is translated into methods that the actual **filesystem** and **hardware** can use
* Some filesystem types are sufficiently different from Linux native ones or Unix ones, and require more manipulation in order to be worked within Linux
	* For instance
		* **VFAT** which comes from **Microsoft**, doesn't have read/write execute permissions for owner, group and world, the three types of entities that standard Unix filesystems have to deal with
	* **VFAT** has an **FAT**, a **F**ile **A**llocation **T**able at the beginning of the disk, rather than in the actual directories
	* It's less secure and more prone to failure
* The native filesystems that arose in Linux are the **ext2**, **ext3** and **ext4** variants
	* They all have utilities for formatting the filesystem, for checking the filesystem
	* You can reset a lot of parameters and tune them after creation or tune them when you first format a partition for the filesystem
		* These days people generally use **ext4**
* **Journaling filesystems** recover quickly from system crashes or if you have a bad shut down such as a power cord goes out or something, and you can do that with little or no corruption
	* So that when the filesystem reboots, you only have to check the most recent transaction that was going on when things went bad rather than check the entire filesystem
* **Journaling filesystems** easily available on the Linux 
	* **ext3**
	* **ext4** pretty dominant
	* **Reiser** was actually the first journaling implementation in the Linux kernel, but it no longer is being maintained then there's no development on it for complicated reasons
	* **JFS** came over from IBM
	* **XFS** came over from SGI
		* **XFS** is actually the **default** filesystem now on **Red Hat Enterprise Linux** systems.
	* **btrfs** or **Butter FS** or **B TRee** filesystem is the most recent that is native to Linux and has a lot of advanced features
		* **Btrfs** is actually the **default** filesystem on **SUSE** systems
* The main development in **Butter FS** was **Shepherded by Chris Mason**
	* It has some very advanced capabilities
		* It can take frequent snapshots of the entire filesystem and then you can rewind back to it or you can do that even for sub-volumes or sections of the entire filesystem and do that almost instantly
			* It does that without wasting much space, because it uses **COW** or **c**opy-**o**n-**w**rite techniques
				* So, you only have to keep extra copies of things which change not the entire filesystem.
		* It has its own internal framework for adding and removing new partitions and physical media, so you can shrink or expand filesystems
			* much as **LVM** (**L**ogical **V**olume **M**anagement)

###### Mounting Filesystems

* In UNIX-like operating systems, all files are arranged in one big filesystem tree rooted at `/`
* Many different partitions on many different devices may be coalesced together by mounting partitions on various mount points, or directories in the tree
* The full form/Syntax of the mount command is
	```bash
	$ sudo mount [-t type] [-o options] device dir
	```
* In most cases, the filesystem type can be deduced automatically from the first few bytes of the partition, and default options can be used, so it can be as simple as
	```bash
	$ sudo mount /dev/sda8 /usr/local
	```
* Most filesystems need to be loaded at boot and the information required to specify mount points, options, devices, etc., is specified in `/etc/fstab`
* The list of currently mounted filesystems can be seen with
	```bash
	$ sudo mount
	/dev/sda5 on / type ext3 (rw)
	proc on /proc type proc (rw)
	sysfs on /sys type sysfs (rw)
	devpts on /dev/pts type devpts (rw,gid=5,mode=620)
	/dev/sda6 on /RHEL6-32 type ext3 (rw)
	/dev/mapper/VGN-local on /usr/local type ext4 (rw)
	/dev/mapper/VGN-tmp on /tmp type ext4 (rw)
	/dev/mapper/VGN-src on /usr/src type ext4 (rw)
	/dev/mapper/VGN-virtual on /VIRTUAL type ext4 (rw)
	/dev/mapper/VGN-beagle on /BEAGLE type ext4 (rw)
	tmpfs on /dev/shm type tmpfs (rw)
	debugfs on /sys/kernel/debug type debugfs (rw)
	/dev/sda1 on /c type fuseblk (rw,allow_other,default_permissions,blksize=4096)
	/usr/local/teaching/FTP/LFT on /var/ftp/pub2 type none (rw,bind)
	/ISO_IMAGES/CENTOS/CentOS-5.5-x86_64-bin-DVD-1of2.iso on /var/ftp/pub
	                            type iso9660 (rw,loop=/dev/loop0)
	sunrpc on /var/lib/nfs/rpc_pipefs type rpc_pipefs (rw)
	/dev/sda2 on /boot type ext3 (rw)
	```
* If a directory is used as a mount point, its previous contents are hidden under the newly mounted filesystem
	* A given partition can be mounted in more than one place and changes are effective in all locations
* You can also mount NFS (Network File Systems) as in
	```bash
	sudo mount 192.168.1.100:/var/ftp/pub /mnt
	```

#
### RAID and LVM

> **RAID**, **R**edundant **A**rray of **I**ndependent **D**isks
**LVM**, **L**ogical **V**olume **M**anagement

##### RAID

* **RAID** is a **R**edundant **A**rray of **I**ndependent **D**isks
* The idea is, instead of just having one disk, you spread your input/output over multiple disks or spindles
* A lot of modern disk controller interfaces allow parallel transport to each of the devices, so that you can actually get more throughput, because you can work on more than one disk at once
	* Now, this can be implemented in software and this is a very mature part of the Linux kernel, the software implementation, or generally better in hardware that the RAID controls built right into the motherboard
* Three essential features of RAID are: 
	1. **Mirroring** --> you write the same data to more than one disk
	1. **Striping** --> that's writing data to more than one disk
	1. **Parity** is storing some extra data that lets you figure out if something has been corrupted or is a little bit wrong, so you can repair it
* There are various levels of RAID, from zero to five
	* As you go up in level, it gets more complex
		* Generally, you need to use more storage, more disk
	* But you also get much better protection and double coverage and redundancy as you go up

##### LVM

* **L**ogical **V**olume **M**anagement is a different beast
* Here, instead of just having one physical partition that you would store a filesystem in, you can have the actual physical partitions broken up into a series of small what are called **extends**
	* Which can be grouped together to form what are called **logical volumes**
		* which look like a partition to the application or many of the utilities on the system, but it really can be split up in a lot of different places
* One of the advantages to using LVM is 
	* It becomes really easy to change the size of these partitions and filesystems
		* With regular physical partitions, it can be rather tedious and difficult to expand or shrink the size of a partition
	* You can easily add new physical volumes to an LVM, as it gets required, or you can remove them
