---
layout: post
title: First study on OVERLAYFS
description: ""
category: study
tags: [study,openstack,docker,OVERLAY]
imagefeature:
comments: true
share: true
---


# **1. OVERLAYFS**

OVERLAY file system is really a awesome technnology to log to organise different files based on different layers. With the idea of OVERLAY, users can create/update the files on the uppest layer while the lower layers can only be read! Live disk and Openwrt system usually build their filesystem on OVERLAYFS, thus store the uses' modification in upper layout and the READ-ONLY disk data is mounted as the lower layer, in this way, system can easily revert users' modification.

Docker also employs OVERLAYFS to implement its' filesystem in docker container. [Docker LAYOUTFS](https://docs.docker.com/engine/userguide/storagedriver/overlayfs-driver/)

![Docker Image]({{ site.url }}/images/overlay_constructs.jpg)



# **1. LAYOUTFS Test**



## 1.2. TEST with the system folders

In tmp, we create lower upper workdir and overlay. we create a file called 123.txt in folder lower.
	
	cd /tmp
	mkdir lower upper workdir overlay
	sudo mount -t overlay -o \
	lowerdir=/tmp/lower,\
	upperdir=/tmp/upper,\
	workdir=/tmp/workdir \
	none /tmp/overlay

with ls /tmp/overlay, we can see that 123.txt is in overlay folder.

1. touch 234.txt in /tmp/overlay, this file is created in folder upper.
2. rm /tmp/overlay/123.txt, 123.txt still in lower, but it create a file like:

	c--------- 1 ubuntu ubuntu 0, 0 Jun  6 21:45 123.txt
	


## 1.2. TEST with the filesystem

	mkdir lower upper overlay
	# Lets create a fake block device to hold our "lower" filesystem
	dd if=/dev/zero of=lower-fs.img bs=4096 count=10240
	dd if=/dev/zero of=upper-fs.img bs=4096 count=10240
	
	# Give this block device an ext4 filesystem.
	mkfs -t ext4 lower-fs.img
	mkfs -t ext4 upper-fs.img
	
	# Mount the filesystem we just created and give it a file
	sudo mount lower-fs.img /tmp/lower
	sudo chown $USER:$USER /tmp/lower
	echo "hello world" >> /tmp/lower/lower-file.txt
	
	# Remount the lower filesystem as read only just for giggles
	sudo mount -o remount,ro lower-fs.img /tmp/lower
	
	# Mount the upper filesystem
	sudo mount upper-fs.img /tmp/upper
	sudo chown $USER:$USER /tmp/upper
	
	# Create the workdir in the upper filesystem and the 
	# directory in the upper filesystem that will act as the upper
	# directory (they both have to be in the same filesystem)
	mkdir /tmp/upper/upper
	mkdir /tmp/upper/workdir
	
	# Create our overlayfs mount
	sudo mount -t overlay -o \
	lowerdir=/tmp/lower,\
	upperdir=/tmp/upper/upper,\
	workdir=/tmp/upper/workdir \
	none /tmp/overlay
	

Quite like the first test, when I create a new file called 234, it has the following file structure.


	.
	|-- lost+found [error opening dir]
	|-- upper
	|   `-- 324
	`-- workdir
    	`-- work [error opening dir]


## 1.3. NESTING support in OVERLAYFS
The nesting overlayFS can be mounted as following:

 
	sudo mount -t overlay -o \
	lowerdir=/tmp/lower:/tmp/lowest,\
	upperdir=/tmp/upper,\
	workdir=/tmp/workdir \
	none /tmp/overlay
