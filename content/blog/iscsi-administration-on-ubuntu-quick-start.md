+++
title = "iSCSI administration on Ubuntu - Quick Start"
date = "2014-09-05T00:00:00-00:00"
tags = ["cloud", "openstack", "tutorial", "quickstart", "cheatsheet", "lvm", "iscsi", "ubuntu"]
type = "post"
+++

This post get's you started with iSCSI administration on an Ubuntu machine.
Although I have used Ubuntu Trusty (14.04) version, it should work with Precise
(12.04) too, with the latest packages.

#### Prerequisites 
Make sure you have atleast a little idea of what these terms
mean: iSCSI, LUN, IQN, initiator, target and portal. Google and wikipedia are
your friends.

#### A quick summary:
There are two parts of iSCSI communication - initiator and target. So let's take an example. There is a storage server in your
company, where you have a 'drive' for your team. The storage server is the
'target', and your laptop, where you'll mount the drive to access it's contents
is the 'initiator'. In other words, target is like a 'server' which stores
data, and allows initiators (think 'clients') to connect to it.

In this short hands-on introduction, we'll use the same Ubuntu machine as
target as well as initiator. We can use a file as the storage behind the
target, but this post also shows how to use LVM logical volume as the backing
store for the iSCSI target. 

Actually, we'll back the logical volume (LV) with a file, as shown in
[this](http://www.rushiagr.com/blog/2014/01/14/quick-start-linux-logical-volume-manager/),
so essentially we're just using 'file as a backing store for targets' but in a
roundabout way :)

OK, let's get started. Make sure you execute all the following commands as root
user.

First install the required dependencies

    apt-get install lvm2 tgt open-iscsi


#### Initialize logical volume
Create a file of 1GB, create a volume group over it, and then over it, create a
400MB logical volume, and see if it got created or not

    root@ra:~# truncate --size 1G backingfile
    root@ra:~# sudo losetup --find --show backingfile 
    /dev/loop0
    root@ra:~# sudo vgcreate myvg /dev/loop0
      No physical volume label read from /dev/loop0
      Physical volume "/dev/loop0" successfully created
      Volume group "myvg" successfully created
    root@ra:~# sudo lvcreate --size 400M --name mylv myvg
      Logical volume "mylv" created
    root@ra:~# lvs
      LV   VG   Attr      LSize   Pool Origin Data%  Move Log Copy% Convert
      mylv myvg -wi-a---- 400.00m                                           

#### Target administration
Now let's create a target, with target ID 1, and give it an IQN (iSCSI
Qualified Name) `iqn.2001-04.example.com:your.first.iscsi.target`:

    tgtadm --lld iscsi --op new --mode target --tid 1 -T iqn.2001-04.example.com:your.first.iscsi.target

List the target, see it's properties:

    root@ra:~# tgtadm --lld iscsi --op show --mode target
    Target 1: iqn.2001-04.example.com:your.first.iscsi.target
        System information:
            Driver: iscsi
            State: ready
        I_T nexus information:
        LUN information:
            LUN: 0
                Type: controller
                SCSI ID: IET     00010000
                SCSI SN: beaf10
                Size: 0 MB, Block size: 1
                Online: Yes
                Removable media: No
                Prevent removal: No
                Readonly: No
                SWP: No
                Thin-provisioning: No
                Backing store type: null
                Backing store path: None
                Backing store flags: 
        Account information:
        ACL information:

You can see there is a LUN, LUN 0 attached to the target. Let's attach our
logical volume `mylv` as LUN 1 to the target.

    tgtadm --lld iscsi --op new --mode logicalunit --tid 1 --lun 1 -b /dev/myvg/mylv

Here, actually you could've attached a flat file as a LUN to the target. So you
could've skipped all the intermediate steps and attached the `backingfile`
directly to the target like this:

    tgtadm --lld iscsi --op new --mode logicalunit --tid 1 --lun 1 -b backingfile

A backing file would've been good enough for this demo, but you know the benefits of logical volume isn't it? :)

Okay, let's see if the LUN got created:

    root@ra:~# tgtadm --lld iscsi --op show --mode target
    Target 1: iqn.2001-04.example.com:your.first.iscsi.target
        System information:
            Driver: iscsi
            State: ready
        I_T nexus information:
        LUN information:
            LUN: 0
                Type: controller
                SCSI ID: IET     00010000
                SCSI SN: beaf10
                Size: 0 MB, Block size: 1
                Online: Yes
                Removable media: No
                Prevent removal: No
                Readonly: No
                SWP: No
                Thin-provisioning: No
                Backing store type: null
                Backing store path: None
                Backing store flags: 
            LUN: 1
                Type: disk
                SCSI ID: IET     00010001
                SCSI SN: beaf11
                Size: 419 MB, Block size: 512
                Online: Yes
                Removable media: No
                Prevent removal: No
                Readonly: No
                SWP: No
                Thin-provisioning: No
                Backing store type: rdwr
                Backing store path: /dev/myvg/mylv
                Backing store flags: 
        Account information:
        ACL information:

Now let's allow all initiators to bind to this target:

    tgtadm --lld iscsi --op bind --mode target --tid 1 -I ALL

We're done with the 'target' side now. You can check, using `netstat` that port
3260, the default port, is now open. Note that all our commands so far started with
`tgtadm`, i.e., the target administration utility.

#### Initiator administration

Now let's start from the 'initiator' end. We'll behave as if we're a client
trying to connect to the server -- the target.

Discover all the targets on our local machine (`127.0.0.1`).

    root@ra:~# sudo iscsiadm --mode discovery --type sendtargets --portal 127.0.0.1
    127.0.0.1:3260,1 iqn.2001-04.example.com:your.first.iscsi.target

From the client perspective, we're now able to see a target. Let's login into
that target

    root@ra:~# sudo iscsiadm --mode node --targetname iqn.2001-04.example.com:your.first.iscsi.target --portal 127.0.0.1:3260 --login
    Logging in to [iface: default, target: iqn.2001-04.example.com:your.first.iscsi.target, portal: 127.0.0.1,3260] (multiple)
    Login to [iface: default, target: iqn.2001-04.example.com:your.first.iscsi.target, portal: 127.0.0.1,3260] successful.

After logging in, the target will be visible in the client's system as a new
device. Running a `fdisk -l` shows that there is a new device `/dev/sda` is now
present.

    root@ra:~# fdisk -l
    
    Disk /dev/vda: 57.1 GB, 57076908032 bytes
    255 heads, 63 sectors/track, 6939 cylinders, total 111478336 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x0001cd46
    
       Device Boot      Start         End      Blocks   Id  System
    /dev/vda1   *        2048   106520575    53259264   83  Linux
    /dev/vda2       106522622   111476735     2477057    5  Extended
    /dev/vda5       106522624   111476735     2477056   82  Linux swap / Solaris
    
    Disk /dev/mapper/myvg-mylv: 419 MB, 419430400 bytes
    255 heads, 63 sectors/track, 50 cylinders, total 819200 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x00000000
    
    Disk /dev/mapper/myvg-mylv doesn't contain a valid partition table
    
    Disk /dev/sda: 419 MB, 419430400 bytes
    13 heads, 62 sectors/track, 1016 cylinders, total 819200 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x00000000
    
    Disk /dev/sda doesn't contain a valid partition table

Now we just need to format this device with a filesystem, say EXT4, and then
mount it at some location to start using it!

    root@ra:~# sudo mkfs.ext4 /dev/sda
    mke2fs 1.42.9 (4-Feb-2014)
    /dev/sda is entire device, not just one partition!
    Proceed anyway? (y,n) y
    Filesystem label=
    OS type: Linux
    Block size=1024 (log=0)
    Fragment size=1024 (log=0)
    Stride=0 blocks, Stripe width=0 blocks
    102400 inodes, 409600 blocks
    20480 blocks (5.00%) reserved for the super user
    First data block=1
    Maximum filesystem blocks=67633152
    50 block groups
    8192 blocks per group, 8192 fragments per group
    2048 inodes per group
    Superblock backups stored on blocks: 
        8193, 24577, 40961, 57345, 73729, 204801, 221185, 401409
    
    Allocating group tables: done                            
    Writing inode tables: done                            
    Creating journal (8192 blocks): done
    Writing superblocks and filesystem accounting information: done 
    
    root@ra:~# mkdir tempmount
    root@ra:~# mount /dev/sda tempmount/
    root@ra:~# cd tempmount/
    root@ra:~/tempmount# ls
    lost+found
    root@ra:~/tempmount# 

#### Destruction
The simplest way to get rid of all the things you've created is to unmount the
device, and restart the system.

Aaand done! 

Cheers!
