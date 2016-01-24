+++
title = "OpenStack Keystone with Cassandra Database in DevStack"
date = "2015-09-10T00:00:00-00:00"
tags = ["cloud", "cheatsheet", "notes", "openstack", "keystone", "cassandra", "nosql", "databases", "distributed", "scalability", "devstack"]
type = "post"
+++

Using Cassandra as backing database for OpenStack Keystone was our solution
to multi-region deployment problem of OpenStack cloud. This blog post is not
to discuss the problem. We are talking about how to have a development
environment to play around with Keystone working with a dev Cassandra deployment.

#### "Just run this script and you're all set!"
I've put together all commands into a script which can be found here:

    https://raw.githubusercontent.com/rushiagr/keystone-cassandra/master/keystone-cassandra.sh

If you have a fresh Ubuntu VM, just copy this script into that machine and
execute it. Give it 15-20 mins at least (depending upon your internet connection), and it will set up:

1. DevStack with Keystone installed and running with all the data stored in/fetched from local Cassandra installation
2. A Cassandra development cluster (CCM) with 5 nodes and replication factor of 3

Of course, you will need Internet access inside the VM. Also, give the VM around
3GB of RAM, else things might not work properly. Actually, if you change the
Cassandra configuration to one node instead of 5, probably 2 GB will suffice. But I've
not tried it. (Let me know if you tried it and it works!)

Notes:

1. Remember, this is a dev cluster. It's not supposed to be used in production. Hell, it's not even ready for it.
2. Keystone is running on 127.0.0.1. I've done this so that it will work on any person's VM
3. I've tested only on a Ubuntu Trusty VM, on Amazon EC2 m4.large instance which has 8 GB RAM. OpenStack on AWS -- ironic, isn't it? :)
4. I'm using Java which comes with Ubuntu's APT packages. In production one is supposed to use Oracle Java as per Cassandra folks.
5. The code for this script is located at `https://github.com/rushiagr/keystone/tree/liberty-cassandra`, i.e. on `liberty-cassandra` branch. Note that this work is currently based upon Keystone's Liberty code as on first week of June. It might not work directly with latest code as it might require fixing imports which might have become outdated. However, I don't think it's going to take more than an hour to make it work with latest code; just that I don't have enough motivation right now to keep the code updated with 'latest' all the time.

#### Credits
This work was done by the 'distributed database' team of 4 people: Ajaya Agrawal, Rushi Agrawal (me), Vivek Dhayaal and Yogeshwar Shenoy, listed in alphabetical order. And obviously Reliance, for providing us an opportunity to work on this thing.
