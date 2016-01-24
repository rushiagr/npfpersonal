+++
title = "Amazon EC2 API with OpenStack - Developer Quick Start"
date = "2014-08-09T00:00:00-00:00"
tags = ["cloud", "openstack", "tutorial", "quickstart", "cheatsheet", "boto", "amazon", "aws", "ec2"]
type = "post"
+++

OpenStack has support for EC2 API, that is, you can fire Amazon's API against an OpenStack cloud and it will still work. This article gets you started with using them locally against DevStack. It is more of a starter guide to a novice.

Fire a DevStack with it's default settings. See [this post](http://www.rushiagr.com/blog/2014/04/03/openstack-in-an-hour-with-devstack) for more information on it.

	git clone http://github.com/openstack-dev/devstack
	cd devstack/
	./stack.sh

Source openrc

	source openrc

View all EC2 credentials available for the current user (here, `demo` user in `demo` tenant)

    $ keystone ec2-credentials-list
    +----------------------------------+----------------------------------+----------------------------------+
    |              tenant              |              access              |              secret              |
    +----------------------------------+----------------------------------+----------------------------------+
    | 0e9f99a6f2064464aa054d305ba08052 | ef61007dae74468eb9593ffbbd22d9f1 | 28c7ad6248de4e6a8649b3e2d122ac5d |
    | 9b93a67201264492be3d0998b87d821b | 1b0a617dbef347cb968c8eed160de0b3 | b6525738ad6044ea9c49abeefabf86de |
    +----------------------------------+----------------------------------+----------------------------------+

But which one is my current tenant? Let's get that from parsing the output of `token-get` command

    $ keystone token-get | grep tenant | awk '{print $4}'
    0e9f99a6f2064464aa054d305ba08052

Note the access and secret keys.

Let's get started with the `boto` client for consuming AWS APIs. I prefer `ipython` shell, for its interactive features, but normal Python shell is just fine too. (Install ipython as `sudo apt-get install ipython`).

Import necessary module

    >> import boto

Create a `conn` connection object, which we'll use for querying our cloud

    >> conn = boto.connect_ec2_endpoint('http://10.0.1.126:8773/services/Cloud',
                aws_access_key_id='ef61007dae74468eb9593ffbbd22d9f1',
                aws_secret_access_key='28c7ad6248de4e6a8649b3e2d122ac5d')

Here `10.0.1.126` is the IP of my machine. Don't forget to change it to yours.

If everything is successful, call to `get_all_instances()` will return an empty list

    >> conn.get_all_instances()
    []

Okay, now let's create an instance. List all the images first

    In [20]: conn.get_all_images()
    Out[20]:
    [Image:aki-00000001,
     Image:ari-00000002,
     Image:ami-00000003,
     Image:ami-00000004]

Image `ami-00000003` should be the cirros image from which we'll create an instance. But still, let's confirm that
    
    In [26]: img = conn.get_image('ami-00000003')

    In [27]: img.name
    Out[27]: u'cirros-0.3.2-x86_64-uec'

Now let's use this image to create an instance. Boto's `get_all_instances` returns a list of reservations, which makes getting the instance object slightly roundabout.

    In [35]: conn.run_instances(image_id='ami-00000003', instance_type='m1.tiny')
    Out[35]: Reservation:r-08b8idoz

    In [40]: reservations = conn.get_all_instances()

    In [42]: resvn = reservations[0]

    In [44]: instance = resvn.instances[0]

    In [45]: instance.state
    Out[45]: u'running'

And then delete it
    
    In [47]: conn.terminate_instances('i-00000002')
    Out[47]: [Instance:i-00000002]

    In [50]: conn.get_all_reservations()
    Out[50]: []

That's it for now :)

Use `ipython` or `bpython` for exploring boto library more and discover more APIs.

If you want to see what EC2 API was actually called behind the scenes, create a file `/etc/boto.cfg` and add these two lines. Now whenever you will use an interactive Python terminal, you'll see on your screen the EC2 API being called.

    [Boto]
    debug=2

Cheers!
