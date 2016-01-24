+++
title = "Cache APT packages with Squid proxy"
date = "2015-06-05T00:00:00-00:00"
tags = ["squid", "apt", "cache", "packaging", "openstack"]
type = "post"
+++

TL;DR: Know how to install and set up Squid proxy, so that you can cache packages,
and hence save bandwidth if you want to install those packages again and again.
Also works if you are already behind a squid proxy.

## Problem: Repetitive download. Slow.


If you deal with virtual machines a lot, you might know the pain of
managing packages on each one of them. Every time I had to create a new VM,
I would run `apt-get update` (to get information about all the latest packages
available for my Ubuntu system), `apt-get dist-upgrade` (to install latest
versions of all packages already installed), and also install some packages
not present in stock Ubuntu image, like `git` (yes, it's 2015
and Ubuntu still doesn't come pre-installed with `git`), `ipython`, `bwm-ng`
and some others. This would mean I'm downloading the same file over the network
over and over again. Now there are two ways to deal with this situation

## Solution 1: Local Ubuntu mirror - Super fast but unweildy

The first solution is to download a complete Ubuntu mirror to your computer.
That is, download ALL Ubuntu packages to your system, and then it is super fast.
The first download will be close to 80GBs though. It would have been fine for
me to download 80GBs, but you'll realize the problem when you want to update
this mirror. If you are trying to update the local mirror every week or so,
each time it will ask you to download around 5GB of data. And that unfortunately
is too much for me to download every few days.

## Solution 2: Cache with Squid proxy - Just about perfect

The other alternative is use a local cache, using Squid proxy. It works like
just another cache: if you want a package of a specific version, Squid will connect
over the internet to find more details about that file. Once it gets these details,
it checks if a file (package) matching those details is already present in the local
cache. If it is locally present, it just sends this local copy to the requester.
So the total Internet bandwidth utilised is only to get the file details, which
is miniscule (Bytes) compared to downloading the whole package (MBs)j. If the
details doesn't match any locally cached packages, the proxy fetches that package
from internet and responds to the requester.

## Practical!

Enough of theory, let's put theory to some practice :)

All of the commands below are run on Ubuntu 14.04 (Trusty).


Install Squid proxy package.

    sudo apt-get install squid

Configure: replace `/etc/squid3/squid.conf` and make it contain these lines.
You will need root permissions to edit this file

    acl localhost src 127.0.0.1/32 ::1
    acl to_localhost dst 127.0.0.0/8 0.0.0.0/32 ::1
    acl localnet src 10.0.0.0/8 # RFC1918 possible internal network
    acl localnet src 172.16.0.0/12  # RFC1918 possible internal network
    acl localnet src 192.168.0.0/16 # RFC1918 possible internal network
    acl SSL_ports port 443
    acl Safe_ports port 80      # http
    acl Safe_ports port 21      # ftp
    acl Safe_ports port 443     # https
    acl Safe_ports port 70      # gopher
    acl Safe_ports port 210     # wais
    acl Safe_ports port 1025-65535  # unregistered ports
    acl Safe_ports port 280     # http-mgmt
    acl Safe_ports port 488     # gss-http
    acl Safe_ports port 591     # filemaker
    acl Safe_ports port 777     # multiling http
    acl CONNECT method CONNECT
    http_access allow manager localhost
    http_access deny manager
    http_access deny !Safe_ports
    http_access deny CONNECT !SSL_ports
    http_access allow localnet
    http_access allow localhost
    http_access deny all
    http_port 3128
    maximum_object_size 1024 MB
    cache_dir aufs /var/spool/squid3 5000 24 256
    coredump_dir /var/spool/squid3
    refresh_pattern ^ftp:       1440    20% 10080
    refresh_pattern ^gopher:    1440    0%  1440
    refresh_pattern -i (/cgi-bin/|\?) 0 0%  0
    refresh_pattern (Release|Packages(.gz)*)$      0       20%     2880
    refresh_pattern .       0   20% 4320
    refresh_all_ims on

You don't need to know or remember what is happening here right now. Just copy
and paste :)

Restart the service:

    sudo service squid3 restart

Now squid service is running, and listening on port 3128. You can use any IP
of your base system which is accessible from your VMs to get packages
via this cache. I give my base system an IP of `192.168.100.1`, so I just
need to do:

    export http_proxy=http://192.168.100.1:3128/

to source the proxy environment variable, which we'll use to point the APT system
to, to fetch packages from. To test if you proxy is working fine locally,
you can provide `127.0.0.1`, your localhost IP instead.

And after that can start using the cache to download packages by just passing `-E`
option to the `sudo` command

    sudo -E apt-get install <your package>

Sure there are alternative ways of using the proxy, but this is my favourite!

## I'm already behind a proxy!

Worry not, add these lines to `squid.conf`, restart squid and you're all set for using the
brand new proxy instead of the old one :)

    cache_peer 10.135.121.138 parent 3128 0 no-query no-digest
    never_direct allow all

## Ending thoughts

You can go to `/var/spool/squid3` and run a `du -sch` to see the total size
of cached files. I find it easy sometimes to calculate the total size of
files this directory holds, to make sure the proxy is working correctly --
if you can 'new' packages being downloaded, but the size of this directory
is not increasing, they're not coming via this proxy, and you need to figure
out why :)

One more important thing I should tell is that the configuration file
we've used not only caches APT packages, but also any static files
hosted anywhere on the internet. So if let's say you want to download an
Ubuntu ISO or some other ISO multiple times in your setup (say, inside VMs),
you can cache the ISO file as well with our current setup.

Tell me what is the size your `/var/spool/squid3/` directory has
reached. Mine is at 1GB right now after a year of it's usage.

Cheers!

-Rushi
