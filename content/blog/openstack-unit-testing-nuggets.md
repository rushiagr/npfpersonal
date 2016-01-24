+++
title = "OpenStack Unit Testing Nuggets"
date = "2014-09-05T00:00:00-00:00"
tags = ["cloud", "openstack", "tutorial", "quickstart", "cheatsheet", "testing"]
type = "post"
+++

A small post about little things I found out while running unit tests in
OpenStack.


## Unit-testing setup

Everybody knows `./run_tests.sh` is used to run the unit tests of an OpenStack
project. But, you require to install dependencies before doing it. And
installing dependencies might not always succeed. So make sure you install
these packages before running `pip install -r requirements.txt`:

    sudo apt-get install build-essential libssl-dev libffi-dev \
        python-dev libxslt1-dev libpq-dev python-mysqldb \
        libmysqlclient-dev libvirt-dev

Atleast `cinder` and `nova` dependencies will get installed properly after
this.

## run_tests frequently used commands

To force the tests to NOT run in a virtual environment, even if it is present:

    ./run_tests.sh -N

Force a clean rebuild of virtual environment

    ./run_tests.sh -f

Run only PEP8 checks

    ./run_tests.sh -p

Run PEP8 checks only on the files which have been changed since last commit

    ./run_tests.sh -8

Run all tests from a specific file only, e.g. nova/tests/test_utils.py

    ./run_tests.sh nova.tests.test_utils

Run all tests of only a specific class inside a test file

    ./run_tests.sh nova.tests.test_utils.ResourceFilterTestCase

Run only a specific test

    ./run_tests.sh nova.tests.test_utils.ResourceFilterTestCase.test_resource_filtering


## Wildcards while running the tests

Frequently you'll find yourself testing only a couple of tests. In such cases,
a wildcard will save you from typing the whole path of the test. The below
command will also run `test_resource_filtering` test:

    ./run_tests.sh nova.tests.*resource_filt*

I currently don't know how to make a test work without adding `nova.tests`
before it

## run_tests is not happy
Sometimes you'll see running `./run_tests.sh` can throw a lot of lines of
ununderstandable gibberish on your screen. In the end it will say `testr
failed`, but it won't give an indication of where it failed and why. I have
seen that this happens due to only one of the following two reasons:

1. *Syntax error*: There is a syntax error in your code.

2. *Dependencies outdated*: Dependencies in your virtual environment is
outdated. In such cases, you will need to recreate a virtual environment with
latest packages. Or better: just update the virtual environment with the latest
packages using this command:

    ./run_tests.sh -u

UPDATE: I've seen that nowadays it doesn't throw a lot of gibberish, but just
says 'testr failed', without any error log or stacktrace. This is the same
situation -- can only happen when there is a syntax error, or if the
dependencies are outdated.

That's it for now.
