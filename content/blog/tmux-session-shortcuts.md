+++
title = "Tmux session shortcuts"
date = "2016-01-10T00:00:00-00:00"
tags = ["cheatsheet", "quickstart", "notes", "tmux", "productivity"]
type = "post"
+++

Tmux is awesome. But the session management commands are way too long for my
liking. Listing all sessions is `tmux list-sessions`, attaching to a session
`mysession` is `tmux attach -t mysession`, etc.

So I created a few functions and aliases, which can be found
[here](https://github.com/rushiagr/myutils/blob/master/aliases/tmux.sh).

The general idea is, all commands start with `mx`, which is basically a
shortcut for 'tMuX'. So `mxl` is to 'l'ist tmux sessions, `mxa` is to 'a'ttach
to a tmux session, etc.

List all running tmux sessions

    r@rushi:~$ mxl
    0: 4 windows (created Sun Jan 10 17:14:11 2016) [89x23] (attached)

You can see one tmux session. Let's create another tmux session with name
`dev`.

    mx dev

List all sessions now

    r@rushi:~$ mxl
    0: 4 windows (created Sun Jan 10 17:14:11 2016) [89x23] (attached)
    dev: 1 windows (created Sun Jan 10 17:59:30 2016) [89x23] (attached)

To attach to session with name `dev`:

    mxa dev

You can also omit session name, and it will attach to the last session you
attached to.

If there was no session with name `dev2`, and you type this:

    mxa dev2

It will automatically create a session for you and attach you to it.

To detach:

    mxd

I find this `mxd` to be easier to type than both `CTRL`+`d` and `tmux
detach`.


### Installation
You just need to copy the content in the above referenced link to `~/.bashrc`
file and from a new terminal session things will be ready for you to use :)
