# What's going on here?

This is my Vagrantfile to help setup the infrastructure for ceph. I am using 2 different setups, a custom CentOS box with a built in /dev/sdb for the osd machines (You can build this from my packer repo CentOS-7.3-puppet-virtualbox.json), or RAW host disk access attached to /dev/sdc. The built in CentOS box with sdb usage is for smaller tests, and the physical disk attachment is for larger tests.

# Why bridged networking?
I am using bridged networking with my home network for a few reasons. A single host with 7-8 vm's running a clustered file system might be a bit heavy for 1 physical host, it depends on the host and disks used. Also, I am killing vms and reshuffling data, and streaming from this cephfs at the same time, if my client reading from cephfs is on the same host as the 3-4 osds then the I/O is even higher than shuffling. With bridged networking I can vagrant up mon1 osd1 osd2 on 1 physical host, and vagrant up admin mon2 osd3 osd4 on another hostr, and even then mon3 osd5 osd6 on another, I can split as much as I want. I could also start off with a test setup all virtual, and migrate virtual to physical hosts on the same subnets. Adding and removing hosts and shuffling data around like one might in a data center might be a good way to learn the software.

# What's your test setup?

I have 2 HP proliant microservers, 12gb ram, The original crappy cpu G1610T, and 5 hard disks. OS disk is sde, and my ceph data drives are sda to sdd. The data drives range from 1TB - 1.5TB.

# Ceph-dash

Ceph dash is installed on the mons and can be seen on http://mon1:80 or http://192.168.2.102:80, apache needs permission to read the keyring file:

    chmod 0660 /etc/ceph/ceph.client.admin.keyring
    chown root:apache  /etc/ceph/ceph.client.admin.keyring


# How to attach physical disks to vagrant?
If you've run out space for your tests by using the built in sdb in my CentOS box and you want to play with bigger disks, you can attach physical disks using VboxManage and servers.yaml

Be very forking careful here.
Use at your own risk.
You have been warned!!! 

Now that you have been warned, we create vmdk files to point to disk devices.

```bash
Physical disks
[root@nas1 puppet]# VBoxManage internalcommands createrawvmdk -filename "osd1.vmdk" -rawdisk /dev/sba
RAW host disk access VMDK file osd1.vmdk created successfully.
[root@nas1 puppet]# VBoxManage internalcommands createrawvmdk -filename "osd2.vmdk" -rawdisk /dev/sbb
RAW host disk access VMDK file osd2.vmdk created successfully.
[root@nas1 puppet]# VBoxManage internalcommands createrawvmdk -filename "osd3.vmdk" -rawdisk /dev/sbc
RAW host disk access VMDK file osd3.vmdk created successfully.
[root@nas1 puppet]# VBoxManage internalcommands createrawvmdk -filename "osd4.vmdk" -rawdisk /dev/sbd
RAW host disk access VMDK file osd4.vmdk created successfully.
```

In the Vagrantfile there is an if statement checking if the server entry in servers.yaml contains a vmdk text string, and if it does, it attempts to attach that string filename as a vmdk file to the vm.


# Playing

To be formatted, just notes for now

```bash
[ceph@mon1 ~]$ sudo ceph osd lspools
0 rbd,
[ceph@mon1 ~]$ sudo ceph osd pool get rbd size
size: 3
[ceph@mon1 ~]$ sudo ceph osd pool get rbd min_size
min_size: 2
[ceph@mon1 ~]$ sudo ceph osd pool set rbd size 2
set pool 0 size to 2
[ceph@mon1 ~]$ sudo ceph osd pool set rbd min_size 1
set pool 0 min_size to 1
[ceph@mon1 ~]$ sudo ceph osd pool get rbd size
size: 2
[ceph@mon1 ~]$ sudo ceph osd pool get rbd min_size
min_size: 1
```

# Sources

There's 2 websites I've been referencing when playing with ceph:
https://www.virtualtothecore.com/en/adventures-ceph-storage-part-1-introduction/
https://www.virtualtothecore.com/en/quickly-build-a-new-ceph-cluster-with-ceph-deploy-on-centos-7/
https://www.howtoforge.com/tutorial/how-to-build-a-ceph-cluster-on-centos-7/
https://www.howtoforge.com/tutorial/using-ceph-as-block-device-on-centos-7/
