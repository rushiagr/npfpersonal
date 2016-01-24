+++
title = "Docker quick start notes"
date = "2015-06-07T00:00:00-00:00"
tags = ["containers", "docker", "technology", "lxc", "virtualization", "cloud", "cheatsheet", "quickstart"]
type = "post"
+++

After reading about docker and containers, I thought let's try them out.
Here are my notes. Obviously all of them are taken from Internet. Maybe this
collection here will help someone start with docker faster than spending time
searching all over the internet.

It assumes your base OS is Ubuntu 14.04 Trusty Tahr (when was the last time
you saw the codename spelled 'Trusty Tahr' and not 'Trusty'? :) ).


Install docker

    sudo apt-get install docker.io

See docker version

    sudo docker version

Pull an Ubuntu Trusty docker image

    sudo docker pull ubuntu:14.04

Alternatively, you can search for a docker image 'tutorial' in docker's repository

    sudo docker search tutorial

And them pull a docker image 'tutorial' by user 'learn'

    sudo docker pull learn/tutorial

List all docker images present in the system

    sudo docker images

Run a docker image, and execute command 'echo "hello world"' in the docker
container created out of that image

    sudo docker run ubuntu:14.04 echo "hello world"

Container information is stored in /var/lib/docker

If you run the above command multiple times, it will create a new container
each time.

To know the ID of the last container, run

    sudo docker ps -l

To list all the running containers

    sudo docker ps

Note that the above command will not show the container we last run, because
the container which we ran last time terminated just after it finished
executing echo command.

Create a new docker image by name `<yourname>/echo` by 'committing' the last
container which you ran

    sudo docker commit <container ID> <yourname>/echo

Now running `sudo docker images` will list you two containers instead of one

Now you can run this new docker container like this:

    sudo docker run <yourname>/echo ls -alrth

If we installed something, or created a file in the old container, it will
be visible now in this container too.

Get more information about a docker image or a running container:

    sudo docker inspect <yourname>/echo


To push docker image to docker repository

    sudo docker push <yourname>/echo

To download ubuntu Trusty base image if not present locally, and open a shell session into it

    sudo docker run -t -i ubuntu:14.04 /bin/bash

-i i.e. --interactive=false, keeps STDIN open even if not attached

-t i.e. --tty=false allocates a pseudo tty

Don't worry what these mean. If you add these options, you'll see that
you already get logged in into the container shell, and the container
only dies off once you exit from that session (usually by writing `exit`
or pressing CTRL + D.

To remove an image:

    sudo docker rmi learn/tutorial

#### Things not covered in this tutorial:
1. Create your own custom docker images and share with other people:
    [https://docs.docker.com/userguide/dockerimages/](https://docs.docker.com/userguide/dockerimages/)

Cheers!
