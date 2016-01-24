+++
title = "Puppet installation from modules"
date = "2014-09-14T00:00:00-00:00"
tags = ["tutorial", "quickstart", "cheatsheet", "ubuntu", "puppet", "automation", "mysql"]
type = "post"
+++

A quick example of how to use Puppet to install and manage MySQL. We'll
download required Puppet modules from their git repositories.

Again, everything is tried on Ubuntu (14.04).

Make sure `hostname -f` shows your FQDN. Then install puppet

    sudo apt-get install puppet

We'll use `git submodules` to manage different git repositories. But first,
create our own repository

    mkdir puppet-mysql
    cd puppet-mysql
    git init

Install Puppet modules `stdlib` and `mysql` into directory `modules` as git
submodules.

    git submodule add https://github.com/puppetlabs/puppetlabs-stdlib.git modules/stdlib
    git submodule add https://github.com/puppetlabs/puppetlabs-mysql.git modules/mysql

Now create a site.pp file in the root directory of this repository, with the following contents

    node default {
        class { 'mysql::server':
            root_password => 'nova'
        }
    }

Now we'll apply this `site.pp` file to the system. As our modules directory is
different from Puppet's default, we'll need to specify that while running
Puppet.

    sudo puppet apply site.pp --modulepath modules/

To see the action in more detail, also pass the `--debug` option to the above
execution

And you're all set.

Now from your commandline, you can try to access mysql and it will work!

    mysql -uroot -pnova

Done! Cheers!
