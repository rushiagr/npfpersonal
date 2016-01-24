+++
title = "Better way to view MySQL tables"
date = "2015-12-12T00:00:00-00:00"
tags = ["cheatsheet", "mysql", "vim", "terminal", "commandline", "cli"]
type = "post"
+++

### The problem
You are trying to print a MySQL table with a large number of columns, with 
`SELECT *` command. You type `SELECT * FROM mysql.user LIMIT 1`, and your terminal
becomes [this](https://raw.githubusercontent.com/rushiagr/public/master/img/mysql-table-with-many-rows-messy.png).
If you wanted to view more than one row, you're in a trouble :)

### The solution

Run MySQL with this option:

    mysql --pager="less --chop-long-lines --quit-if-one-screen --no-init'

This will make your table display only the rows it can in the current screen, something like [this](https://raw.githubusercontent.com/rushiagr/public/master/img/mysql-with-less-pager-neat.png). You can
use the arrow keys to move sideways to view the hidden column. Pressing the 'right' arrow key will move half page towards right. Neat, isn't it?

You can create an alias for mysql:

    # Using shorter version of 'less' flags mentioned above
    alias mysql='mysql -SFX'

You can put the above line in your `~/.bashrc` file to load this alias
in every new terminal session.


### Bonus point for Vim users
`less` allows using keys `j` and `k` for scrolling down and scrolling up. Unfortunately, you cannot directly use keys `h` and `l` to move sideways using `less`. Fortunately, you can map `h` and `l` to move left or right, respectively. Here's how to do that:

Create a file `.lesskey` in your home directory, with the following contents

    l noaction 10\e)
    h noaction 10\e)

Now run this command, to generate `~/.less` file.

    lesskey

This will generate a binary file which `less` understands. If you now start a new MySQL terminal session (of course with the above said `--pages` flag), you can use Vim's `HJKL` movements. Here I have specified to move 10 characters to the right if you make one 'right' Vim movement.

### Quick setup script

Don't want to do the above stuff manually? Just run this command and your computer will be set up in a second!

    sh -c "$(wget -q https://raw.githubusercontent.com/rushiagr/public/master/scripts/mysql-pretty-table.sh -O -)"

Note that changes will take effect from a new shell session (or you can run `source ~/.bashrc` if you want things to work in the current session too.

### More information

Find more information at below links:

[About mapping 'h' and 'k' to Vim movements in 'less'](http://unix.stackexchange.com/a/169969/91602)

[About using 'less' as MySQL pager](http://stackoverflow.com/a/6422698/1143173)


Cheers!
