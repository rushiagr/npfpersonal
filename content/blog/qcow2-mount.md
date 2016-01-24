+++
title = "Mounting QCOW2 images"
date = "2014-08-02T00:00:00-00:00"
tags = ["virtualization", "tutorial", "cloud", "tech"]
type = "post"
+++

Isn't it fun that even before you start a VM out of an image, you can add files to that image, see and edit the directory and file structure of that VM? 

I wanted to boot a VM out of a disk-image, but how will I know out of the 256 available IPs for that VM, which one actually got assigned? I tried vnc console, but the connection was terribly flaky. Even so, it was felt quite ugly to use an interface when I was trying to move to a keyboard-only (command line) world.  So I just inserted a static IP into the `/etc/network/interfaces` file of that image! (I wasn't aware of `arp-scan` before I discovered the trick described in this post) 

We'll mount the image, tweak the filesystem of that image, and then boot the image.

Install `qemu-utils` and enable `ndb` module

    sudo apt-get install qemu-utils
	sudo modprobe nbd

Use any qcow2 image, and if you don't have any, download a small CirrOS cloud image (around 13MB).

    wget http://download.cirros-cloud.net/0.3.2/cirros-0.3.2-x86_64-disk.img
       
Connect the image to the first nbd device

	sudo qemu-nbd -c /dev/nbd0 cirros-0.3.2-x86_64-disk.img

Mount the image. For nbd0, see all the devices available (`/dev/nbd0<some-number-or-string>`) and try attaching to starting from the first one

	sudo mount /dev/nbd0p1 /mnt

Now at /mnt, you can see the complete filesystem of that image, and make necessary changes. You can do all sorts of things -- change `sources.list`, `/etc/network/interfaces`, put additional files inside the VM for particular users, etc.

After you're done, unmount it.

	sudo umount /mnt

And disconnect the loopback device too

	sudo qemu-nbd -d /dev/nbd0

Done!

PS: I actually created two functions for mounting and unmounting, so that I don't remember all these commands. Find them [here](https://github.com/rushiagr/myutils/blob/master/aliases/qcow2-mount.sh).

Credits: Vigneshvar introduced me to `qemu-nbd` tool.
