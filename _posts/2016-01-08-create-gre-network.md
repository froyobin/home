---
layout: post
title: Creating Gre Network
description: ""
category: study
tags: [study,openstack,gre]
imagefeature:
comments: true
share: true
---


# **1. Introduction**
The gre network can help to build a point-point network that for the public gateway, it hides the detail of the network. In the following sections, I will show you how to configure the gre 
network through a practical case.

The network information are described below:

    node 1(Centos): 9.119.43.71 no Internet Access mask 255.255.255.0 gateway 9.119.43.1

    node 2(Ubuntu): 9.110.190.35 has Internet Access mask 255.255.255.0 gateway 9.110.190.1

I would like node 2 can have Internet access through node 1. As node 2 and node 1 are in different subnet, Snat cannot work directly. So one solution is to use gre + snat.



# **2. Close the firewall and insert the gre module**
On node 1, close the firewall with:

    systemctl stop firewalld
    systemctl disable firewalld

On node 2, close the firewall with:
     ufw disable

Run command on both nodes to make sure gre module is inserted.
    modprobe gre
    lsmod|grep gre


#  **3. Create the gre tunnels**
Create gre on node 2:

     sudo ip tunnel add gre0 mode gre remote 9.119.43.71 local 9.110.190.35 ttl 255
     sudo ip link set gre0 up
     sudo ip addr add 10.10.10.2 peer 10.10.10.1 dev gre1

Create gre on node 1:
     
     sudo ip tunnel add gre0 mode gre remote 9.110.190.35 local 9.119.43.71 ttl 255
     sudo ip link set gre0 up
     sudo ip addr add 10.10.10.1 peer 10.10.10.2 dev gre1

# **4. Enalbe ipv4 forward**
On node 2 run

Edit /etc/sys/net.ipv4.ip_forward = 1 and set:

    net.ipv4.ip_forward = 1
    sysctl -p


#**5.Router table IPtables Configuration**
As the default gateway of node 1 is 9.119.43.1, so we should let all packages go through gre0 except the packages send to the 9.110.190.35. So set the rules below:
 
    ip route add 9.110.190.35/32 via 9.119.43.1
    route del default   (You shoud create the outging route firstly!!! or your machine cannot be reached anymore !!!!!)

At this moment your machine can only be accessed through node2!

    route add default gw 10.10.10.2

Now node 1 has been set up to pass all the packages through gre0.

On the node2, add rules to forward packages from gre to eth2
    
      ip route add 10.10.10.0/24 dev gre0
      iptables -t nat -A POSTROUTING -s 10.10.10.0/255.255.255.0 -o eth2 -j MASQUERADE


Now nod 1 has the Internet access!




