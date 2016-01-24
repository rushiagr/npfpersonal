+++
title = "Apache mod_wsgi parallelization notes"
date = "2015-07-24T00:00:00-00:00"
tags = ["cloud", "cheatsheet", "notes", "apache", "server", "devops", "operations", "systems"]
type = "post"
+++


This is my notes on
['Processes and Threading'](https://code.google.com/p/modwsgi/wiki/ProcessesAndThreading)
doc by the author of mod_wsgi module of Apache. This blog post just serves as a 'quick refresher', and
is only useful if you have read the original document but it's been too long since you
read it :)

## Apache with mod_wsgi

A Python application can run with multiple processes as well as multiple threads
with mod_wsgi.

### Prefork multiprocessing module

Apache creates multiple processes, and each request is handled by one process.
A process only handles one request at a time.
This means, if you have set number of processes to 1, there will be only one
request handeled at a time overall.

### Worker multiprocessing module

Multiple processes, and multiple threads in each processa.
Even if a process is handling a request, another thread in the same process
can handle one more request.
You might need some synchronization primitive to make sure multiple threads
of same process don't corrupt shared memory (only occurs when shared memory
is mutated)

### But GIL?

Python GIL problem is largely alleviated with mod_wsgi since multiple processes
can handle requests, and GIL has impact ranging to only one process. One more
point to note is that the apache code which maps a URL/request to a wsgi application,
and the code which maps static file URLs to actual static files to serve is
written in C, and is free from GIL.

In the wsgi python code, two environment variables: 'wsgi.multithread' and
'wsgi.multiprocess' will define which of the above two modules are going to be
used.
