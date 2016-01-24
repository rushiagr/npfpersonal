+++
title = "DevStack behind proxy"
date = "2014-08-05T00:00:00-00:00"
tags = ["tech", "cloud", "openstack", "devstack"]
type = "post"
+++

I have now seen some people struggling to get DevStack working behind proxy. Some, thinking it is a bug in DevStack, have actually posted patches for it too! Here, I'll tell you the simple way to get `stack.sh` complete succesfully from behind a proxy.

By default, `devstack` will clone from the 'actual' OpenStack git repositories, residing at `git://git.openstack.org`. Some people might face a problem with it, as DevStack uses `git` protocol to clone the repo. We'll instead use HTTP which is provided by GitHub mirror  (yes, you heard it right. GitHub is just a 'mirror' for OpenStack code, not the primary repository). For this we'll need to set `GIT_BASE` in `localrc` as:

    GIT_BASE=http://github.com

Export http and https proxy variables

    export http_proxy=<your-http-proxy>
    export https_proxy=<your-https-proxy>

Now, you will need to export `no_proxy` environment variable. This environment variable should contain localhost, as well as the IP your current machine has got. Say your current machine has IP `12.34.56.78`:

    export no_proxy=127.0.0.1,12.34.56.78

After you have exported these three variables, you're free to run `./stack.sh`, and it should finish successfully.

If you are doing a single-node devstack setup, you don't need to do anything
else and can stop here. If you are doing a multi-node setup, the services
running on one node might not communicate properly with services on a different node. In order to
fix this, do this: go to individual services running inside screens, stop the
service (by pressing `CTRL`+`C`), unset the proxy environment variables (`unset
http_proxy https_proxy no_proxy`), and restart the service again (by pressing
up arrow and then pressing `Enter`).

Cheers!
