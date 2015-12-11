---
layout: post
title: Creating Openstack Glance Image
description: ""
category: study
tags: [study,openstack,neutron]
imagefeature:
comments: true
share: true
---


# **1. Use Public Cloud Image**
The simplest way to prepare a glance image is to find one on the Internet.

here are some useful link to obtain a glance image:

**Centos7** [http://cloud.centos.org/centos/7/images/](http://cloud.centos.org/centos/7/images/)

**Centos6** [http://cloud.centos.org/centos/6/images/](http://cloud.centos.org/centos/6/images/)

**Ubuntu** [http://cloud-images.ubuntu.com/](http://cloud-images.ubuntu.com/)

**Fedora**[https://getfedora.org/en/cloud/download/](https://getfedora.org/en/cloud/download/)

As these imges use cloud-init to do the initialization, usually, tenants should use the the key pairs to login the system.

Sometimes, we hope to use password to login the system, the scripts below can help you achieve this goal.

{% highlight python %}
	#cloud-config
            password: mysecret
            chpasswd: { expire: False }
            ssh_pwauth: True


	#cloud-config
		chpasswd:
 		list: |
   			root:passw0rd
   			centos:stackops
 		expire: False
		ssh_pwauth: True
{% endhighlight %}

# **2. Customize the Public Cloud Image**
Sometimes, we may need to customize the public cloud image. I will show you how to modify the cloud image to enable password login method.

  
  
  
  
  
  
  
##    **1.1. Download the Cloud Image**
	download the cloud image from public website, in my case, I take centos7 for instance.
    
##   **1.2. Install Guestfs Tools**
	 sudo apt-get install guestfs-tools
     
##    **1.3. Mount  cloud image**
	sudo guestmount -a CentOS-7-x86_64-GenericCloud-1503.qcow2 -i --rw /mnt

##    **1.4. Modify the files**
	sudo su
    cd /mnt
    chroot /mnt
    vim /etc/cloud/cloud.conf
    
To enable password login, just modify these lines:

	disable_root: 1
	ssh_pwauth:   0
   
To:

	disable_root: 0
	ssh_pwauth:   1
   
set root password:

	password root
##    **1.5. Finish and Umount**
	Exit the chroot environment and umount /mnt
    
After these steps, upload the image to the cloud and enjoy it:)

# **2. Export from Standarded Installation**


##    **2.1. Install the virt-manager**

Firstly, intall the virt-manager to create and start the virtual machines.


##    **2.1. Prepare a qcow2 file**

Create a qcow2 file for the vms with command:


	qemu-img create -f qcow2 /data/centos-6.4.qcow2 10G
    
    
##    **2.1. Install the System**

install the virtual machine with the help of virt-manager. Remember to hardisk format to **qcow2** and make sure you have enable the network interface.

In order to implemet the function like auto resize disk size, and to obtain publick keys, please remember to install **cloud-init** packages and **cloud-utils-growpart**.
On red hat system, try to run:

	yum install -y cloud-init
	yum install -y dracut-modules-growpart cloud-utils-growpart

On the RHEL sysyem, it may faild in package update, try to run command below:

	yum clean all
    yum distro-sync
  

##  **2.2  Post work**

Install the ACPI to support power management in virtual layer
	
    # yum install acpid
	# chkconfig acpid on

Disable zeroconf router

	# echo "NOZEROCONF=yes" >> /etc/sysconfig/network
    
   
Configure the console

	serial --unit=0 --speed=115200
	terminal --timeout=10 console serial
	# Edit the kernel line to add the console entries
	kernel ... console=tty0 console=ttyS0,115200n8
   
Clean Mac address information and undefine the virtual machine manager by virt-manager

	virt-sysprep -d centos7
    virsh undefine centos7
    
congratulations!! your image is ready for uploading!


