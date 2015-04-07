---
layout: post
title: pstack for OS X
---

Unfortunately OS X does not have a wonderful Linux-world utility `pstack` that helps to inspect a running process with a shell one-liner:

```sh
pstack `pgrep myprogram`
```

So I wrote a replacement based on system gdb. Please use it if you find it useful.

{% gist ceacb52b1e5cf1a9594a %}

I discovered a few interesting things while writing this utility.
Homebrew `gdb` sometimes leaves a process in a sleep state after attaching and detaching. Okay, let's use system `gdb`. But it is very old (version 6), non-standard and compiled without python support so we cannot use nice python scripting (initially I found a python script to run inside gdb). Okay, there is a nice native `gdb command

```sh
thread apply all bt
```

that does everything we need. The next problem with `gdb` was a lack of `--ex` command-line argument that allows to specify a gdb script inline. So we need to create a temporary file with a gdb script. And finally OS X lacks `/proc` filesystem so to determine a process binary it's needed to consult with `ps` command.

The moral of this story. OS X and Linux are POSIX-compatible but differences environments, different flavors of developer tools and well-known bugs became nightmare to port even so simple utilities.