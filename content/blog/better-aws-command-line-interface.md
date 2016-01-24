+++
title = "Better AWS Command Line Interface"
date = "2016-01-02T00:00:00-00:00"
tags = ["commandline", "cli", "terminal", "ux", "aws", "python", "boto"]
type = "post"
+++

When I first tried to use the AWS CLIs provided by Amazon, I found them not so
human-friendly. Sure, maybe they're good for automating stuff, but if you just
want to view the public IP of a virtual machine, and the CLI throws this on
your screen for just one VM instance, you're likely not going to be impressed:

    r@rushi:~$ aws ec2 describe-instances
    {
        "Reservations": [
            {
                "OwnerId": "123456789012",
                "ReservationId": "r-abcd1234",
                "Groups": [],
                "Instances": [
                    {
                        "Monitoring": {
                            "State": "disabled"
                        },
                        "PublicDnsName": "ec2-52-52-52-52.ap-southeast-1.compute.amazonaws.com",
                        "State": {
                            "Code": 16,
                            "Name": "running"
                        },
                        "EbsOptimized": false,
                        "LaunchTime": "2015-12-31T12:59:59.000Z",
                        "PublicIpAddress": "52.52.52.52",
                        "PrivateIpAddress": "172.31.31.31",
                        "ProductCodes": [],
                        "VpcId": "vpc-1234abcd",
                        "StateTransitionReason": "",
                        "InstanceId": "i-1234abcd",
                        "ImageId": "ami-1234abcd",
                        "PrivateDnsName": "ip-172-31-31-31.ap-southeast-1.compute.internal",
                        "KeyName": "mykey",
                        "SecurityGroups": [
                            {
                                "GroupName": "rushi-sg",
                                "GroupId": "sg-abcd1234"
                            }
                        ],
                        "ClientToken": "",
                        "SubnetId": "subnet-abcd1234",
                        "InstanceType": "t2.micro",
                        "NetworkInterfaces": [
                            {
                                "Status": "in-use",
                                "MacAddress": "06:f3:82:a1:fb:c5",
                                "SourceDestCheck": true,
                                "VpcId": "vpc-abcd1234",
                                "Description": "",
                                "Association": {
                                    "PublicIp": "52.52.52.52",
                                    "PublicDnsName": "ec2-52-52-52-52.ap-southeast-1.compute.amazonaws.com",
                                    "IpOwnerId": "amazon"
                                },
                                "NetworkInterfaceId": "eni-abcd1234",
                                "PrivateIpAddresses": [
                                    {
                                        "PrivateDnsName": "ip-172-31-31-32.ap-southeast-1.compute.internal",
                                        "Association": {
                                            "PublicIp": "52.52.52.52",
                                            "PublicDnsName": "ec2-52-52-52-52.ap-southeast-1.compute.amazonaws.com",
                                            "IpOwnerId": "amazon"
                                        },
                                        "Primary": true,
                                        "PrivateIpAddress": "172.31.31.31"
                                    }
                                ],
                                "PrivateDnsName": "ip-172-31-31-31.ap-southeast-1.compute.internal",
                                "Attachment": {
                                    "Status": "attached",
                                    "DeviceIndex": 0,
                                    "DeleteOnTermination": true,
                                    "AttachmentId": "eni-attach-abcd1234",
                                    "AttachTime": "2015-12-31T12:59:59.000Z"
                                },
                                "Groups": [
                                    {
                                        "GroupName": "mygroup",
                                        "GroupId": "sg-abcd1234"
                                    }
                                ],
                                "SubnetId": "subnet-abcd1234",
                                "OwnerId": "123456789012",
                                "PrivateIpAddress": "172.31.31.31"
                            }
                        ],
                        "SourceDestCheck": true,
                        "Placement": {
                            "Tenancy": "default",
                            "GroupName": "",
                            "AvailabilityZone": "ap-southeast-1a"
                        },
                        "Hypervisor": "xen",
                        "BlockDeviceMappings": [
                            {
                                "DeviceName": "/dev/sda1",
                                "Ebs": {
                                    "Status": "attached",
                                    "DeleteOnTermination": true,
                                    "VolumeId": "vol-abcd1234",
                                    "AttachTime": "2015-12-31T12:59:59.000Z"
                                }
                            }
                        ],
                        "Architecture": "x86_64",
                        "RootDeviceType": "ebs",
                        "RootDeviceName": "/dev/sda1",
                        "VirtualizationType": "hvm",
                        "Tags": [
                            {
                                "Value": "rushi some VM",
                                "Key": "Name"
                            }
                        ],
                        "AmiLaunchIndex": 0
                    }
                ]
            },
        ]
    }

The only things I'm interested in, after creating a VM is to see it's public
IP, it's flavor, volume size, and instance name. The JSON output which it
throws on my face makes viewing that basic information much harder.

Fortunately, I am a programmer. So I wrote a simple CLI: `lsvm`:

    r@rushi:~$ lsvm -h
    lsvm [-h] [-s] [<name>]
        -h      Prints helptext and exits
        -s      Prints sizes of VM disks in GB, starting with root disk
        <name>  Only prints VM whose name contains '<name>'

Installing the CLI is simple. Just run this command:

    sudo sh -c "$(wget -q https://raw.githubusercontent.com/rushiagr/public/master/scripts/simplest-aws-cli.sh -O -)"

Ensure that you have internet connection before running this command, and also
make sure that your computer can run `pip` commands. Keep your access and
secret key handy.

### List all the VM instances

    r@rushi:~$ lsvm
        ID              Name           Status   Flavor        IP      Vols
    i-abcd1234     rushi dev m/c      running  t2.micro 52.12.123.123  1
    i-abcd1233   rushi pkg builder    running  t2.micro 52.12.123.122  1
    i-abcd1232 rushi vanilla devstack running  t2.large 54.12.123.121  1
    i-abcd1231  rushi dbaas devstack  running m4.xlarge 52.12.123.120  1

List all VMs whose name contains word 'devstack'

    r@rushi:~$ lsvm
        ID              Name           Status   Flavor        IP      Vols
    i-abcd1232 rushi vanilla devstack running  t2.large 54.12.123.121  1
    i-abcd1231  rushi dbaas devstack  running m4.xlarge 52.12.123.120  1

Show sizes of volumes of instances:

    r@rushi:~$ lsvm  -s
        ID                   Name               Status   Flavor        IP       Vols(GB)
    i-abcd1234          rushi dev m/c          running  t2.micro 52.12.123.123    [8]
    i-abcd1233        rushi pkg builder        running  t2.micro 52.12.123.122    [8]
    i-abcd1232      rushi vanilla devstack     running  t2.large 54.12.123.121    [50]
    i-abcd1231       rushi dbaas devstack      running m4.xlarge 52.12.123.120    [50]


### Creating VM instance
Creating a VM instance is tough too, with AWS CLI, so I made another command
`mkvm` (installed already with the previous script) which is very intuitive for human use. It asks for information in a
step-by-step manner - exactly the way a human sitting in front of a computer should be
treated with:

    r@rushi:~$ mkvm
    Only Ubuntu image supported as of now
    Available flavors: ['t1.micro', 'm1.small', 'm1.medium', 'm1.large', 'm1.xlarge', 'm3.medium', 'm3.large', 'm3.xlarge', 'm3.2xlarge', 'm4.large', 'm4.xlarge', 'm4.2xlarge', 'm4.4xlarge', 'm4.10xlarge', 't2.micro', 't2.small', 't2.medium', 't2.large', 'm2.xlarge', 'm2.2xlarge', 'm2.4xlarge', 'cr1.8xlarge', 'i2.xlarge', 'i2.2xlarge', 'i2.4xlarge', 'i2.8xlarge', 'hi1.4xlarge', 'hs1.8xlarge', 'c1.medium', 'c1.xlarge', 'c3.large', 'c3.xlarge', 'c3.2xlarge', 'c3.4xlarge', 'c3.8xlarge', 'c4.large', 'c4.xlarge', 'c4.2xlarge', 'c4.4xlarge', 'c4.8xlarge', 'cc1.4xlarge', 'cc2.8xlarge', 'g2.2xlarge', 'cg1.4xlarge', 'r3.large', 'r3.xlarge', 'r3.2xlarge', 'r3.4xlarge', 'r3.8xlarge', 'd2.xlarge', 'd2.2xlarge', 'd2.4xlarge', 'd2.8xlarge']
    Select flavor ['l' to list]: t2.micro
    Available key pairs: ['rushi-kp-1', 'prod-keypair', 'test-keypair']
    Select keypair: rushi-kp-1
    Available security groups: ['Rushi SecGroup', 'openToAll', 'Rushi SG', 'Rushi Devstack SG']
    Select security group. None to create new one: Rushi SecGroup
    Enter root volume size in GBs: 8
    r@rushi:~$

### Ending notes
The AWS CLI was created maybe to help automation, but they're not quite
suitable for human use. Here is my attempt to show to the world how the things
could be made easier for human consumption. I use these commands quite a lot
when I want to quickly create a VM, or want to get information about already
created VMs. Logging in into the console is way too slow for my liking.


Just to recap:

1. The commands are shorter, so you have to type less
2. The output is concise - only important information is included
3. `mkvm` provides you with information which you'll require while creating the
   instances, e.g. security group names, etc.
4. Nowhere do you need to specify (hexadecimal) IDs while creating VMs

I just spent a few hours over the weekend to get this working. So obviously
the code is going to have rough edges, frowned-upon software practices, and a
lot of unhandled edge cases. I'll be pretty excited if you want to help wit
the idea! I've created a repository at
[http://github.com/rushiagr/aclih](http://github.com/rushiagr/aclih) where
please feel free to submit pull requests or issues. Interested in writing
'rmvm' to delete VM anyone? :)

Thank you :)
