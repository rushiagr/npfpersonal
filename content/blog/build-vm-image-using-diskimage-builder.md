+++
title = "Build VM Images using Diskimage-builder"
date = "2016-01-02T00:00:00-00:00"
tags = ["cheatsheet", "quickstart", "notes", "cloud", "openstack", "vm"]
type = "post"
+++

OpenStack has this nice tool [diskimage-builder](https://github.com/openstack/diskimage-builder)to create virtual machine images without the need
of a cloud. These vm images can then be uploaded to cloud (e.g. in Glance for
OpenStack cloud), and they become immediately usable. You can also create VMs in
virtualbox from these images (though I don't remember exact steps to make the
image work with VirtualBox. Do let me know if you get the VM working with
VirtualBox/Vagrant)

Here I'll describe ways to create a bare cloud-uploadable Ubuntu image. I will
also provide information as to how to build an image which will have some
packages pre-installed in them. Note that the commands here will create only
one image file as opposed to three -- one each for ramdisk, kernel and machine image.

Prerequisites

    sudo apt-get install qemu-utils
    git clone http://github.com/openstack/diskimage-builder
    cd diskimage-builder
    sudo pip install -r requirements.txt

All the binaries are in bin filder. You can go in the `bin\` directory to
execute diskimage-builder commands, or add that directory to your `$PATH`

Create bare Ubuntu image, which you can directly upload to a cloud e.g.
OpenStack. 

    disk-image-create -a amd64 -o ubuntu-amd64 vm ubuntu

Image generated will be of name `ubuntu-amd64.qcow2`. Such an image will be for
same OS version as your host Ubuntu version. If you want
to build an image against a different OS version, specify
`DIB_RELEASE=<releasename>` as a prefix to the command.

    DIB_RELEASE=trusty disk-image-create -a amd64 -o ubuntu-amd64 vm ubuntu

Create an Ubuntu VM image which you can boot via KVM or VirtualBox/Vagrant.
You will need to manually
add public keys to authorized_keys for a user inside that VM.

    disk-image-create -o base -a amd64 vm base ubuntu cloud-init-nocloud

Create an image with `mysql-server` and `tmux` package (and their dependencies) installed inside the image:

    disk-image-create -a amd64 -o ubuntu-amd64 -p mysql-server,tmux vm ubuntu

How to upload image to glance:

    glance image-create --name dib-ubuntu --disk-format=qcow2 --container-format=bare < img/ubuntu-amd64.qcow2

Where `ubuntu-amd64.qcow2` is the image to upload.

Thanks!
