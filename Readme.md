COM 341, Operating Systems
==========================
# Project #2

![Compiling a Linux Kernel](http://i.imgur.com/t0c7Wav.gif)

After getting the shell from the previous project, the infamous instructor [Sergei Rachmiroff](https://i.imgur.com/hLHngQQ.jpg) has decided to create a few more essential system utilities for his operating system. The first program he wrote was a task information utility called `tasks`. The program works in a very similar manner to `top` and `ps`. It prints a table with information about processes currently running on the system. Unfortunately, the OS Sergei is building has yet to get a debugger. That is why Sergei wants to test and debug the `tasks` program on GNU/Linux first. Unfortunately, programs like `top` and `ps` query information about the processes on Linux through a special pseudo-filesystem called `/proc` (`man proc`). Sergei's OS does not have such a fancy subsystem in it. Instead, it has two system calls called `get_pids()` and `get_task_info(...)` to provide the data about the running programs. To recreate the environment of his OS on GNU/Linux, Sergei has ported the code from his OS with detailed instructions on how to integrate it into the old version of the Ubuntu he is using. Sergei asks you to add the code into Linux sources and compile the kernel for him as it looks like his toaster of a computer is not too powerful to do that in a reasonable amount of time for the new Ubuntu 24.04 distribution.

In this work, we hope you will

* Learn about alternative interfaces to talk to an OS (such as pseudo-filesystems).
* Learn what the code in the kernel looks like.
* Learn what a real PCB/TCB (Process/Thread Control Block) looks like.
* Learn how to add new system calls into the Linux kernel.
* Learn how to configure a Linux kernel before compilation.
* Learn how to compile a Linux kernel.
* Learn about the #1 programmer's excuse for legitimately [slacking off](https://xkcd.com/303).
* Learn how to work with complex code bases not written by you
* Learn how to use cross-referencing tools and version control to search for information to modify existing code
* Learn why your Android phone (unless it is from Google) does not get OS updates the same day Google releases them
* Learn to work with other sources of information such as man pages, books, and specialized social networks to get the necessary information.

## Required Tools

On your host machine, you will need a Unix shell with essential Unix command line utilities and an SSH client to connect to the course Vagrant envronment. You may also find the `scp` program useful to transfer files between host and guest operating systems.

## What to Do

The goal of the project is to add and port several new system calls from the old Ubuntu 16.04 Linux kernel to the new one bundled with the Ubuntu 24.04 LTS distribution. Follow all the steps in the Readme file under the `ubuntu-xenial` directory to finish the task.

### What to Submit

1. Ensure that during the compilation process, you have specified your last name and the first letter of the first name in the `...debian.master/changelog` file in the correct form (all letters are in lowercase, no `_` or `-` characters between your last name and the first letter of the first name).

2. Put the updated or compiled files outlined below into the `ubuntu-noble` directory of your private course repository.

    * `ubuntu-noble/task_info/get_pids.c`
    * `ubuntu-noble/task_info/get_task_info.c`
    * `ubuntu-noble/vmlinuz-6.8.0-*-generic`

3. Commit and push your repository through Git. Submit a URL to the last commit on GitHub to Moodle before the deadline.

## Deadline

Check Moodle for information regarding the deadlines.
