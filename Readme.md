COM 341, Operating Systems
==========================
# Project #2

![Compiling a Linux Kernel](http://i.imgur.com/t0c7Wav.gif)

After getting the shell from the previous project, the infamous instructor [Sergei Rachmiroff](https://i.imgur.com/hLHngQQ.jpg) has decided to create a few more essential system utilities for his operating system. The first program he wrote was a task information utility called `tasks`. The program works in a very similar manner to `top` and `ps`. It prints a table with information about processes currently running on the system. Unfortunately, the OS Sergei is building has yet to get a debugger. That is why Sergei wants to test and debug the `tasks` program on GNU/Linux first. Unfortunately, programs like `top` and `ps` query information about the processes on Linux through a special pseudo-filesystem called `/proc` (`man proc`). Sergei's OS does not have such a fancy subsystem in it. Instead, it has two system calls called `get_pids()` and `get_task_info(...)` to provide the data about the running programs. To recreate the environment of his OS on GNU/Linux, Sergei has ported the code from his OS with detailed instructions on how to integrate it into the old version of the Ubuntu he is using. Sergei asks you to add the code into Linux sources and compile the kernel for him as it looks like his toaster of a computer is not too powerful to do that in a reasonable amount of time.

In this work, we hope you will

* Work with a different popular virtualization system.
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

On your host machine, you will need a Unix shell with essential Unix command line utilities and an SSH client to connect to virtualized, emulated, and containerized systems. You may also find the `scp` program useful to transfer files between host and guest operating systems. On Windows, you can get all the necessary things by installing Git from its official [Website](https://git-scm.com). On macOS, you can get everything by installing Command Line Tools for Xcode with the `xcode-select --install` command. Finally, you may also use our lab computers. Lab machines have all the necessary software for this task.

## Part #1

The goal of the first part is to modify an old version of the Linux kernel bundled with the Ubuntu 16.10 GNU/Linux distribution by adding two custom system calls to it. This will allow a small task information utility to query information about running processes directly without consulting the proc filesystem `procfs`. Follow all the steps in the Readme file under the `ubuntu-yakkety` directory to finish the task.

### What to Submit

1. Ensure that during the compilation process, you have specified your last name and the first letter of the first name in the `...debian.master/changelog` file in the correct form (all letters are in lowercase, no `_` or `-` characters between your last name and the first letter of the first name).

2. In your private course repository given to you by the instructor during the lecture, create the path `project-02/part-01/`.

3. Create a directory `project-02/part-01` in your private course repository. Put the following files into it from the Linux source tree (NOT from the `kernel-project` directory)

    * `task_info/`
    * `init/Kconfig`
    * `include/linux/syscalls.h`
    * `include/uapi/asm-generic/unistd.h`
    * `arch/x86/entry/syscalls/syscall_32.tbl`
    * `arch/x86/entry/syscalls/syscall_64.tbl`
    * `kernel/sys_ni.c`
    * `Makefile`
    * `debian.master/changelog`
    * `~/linux-image-*-generic_*_amd64.deb`

Create a `results.txt` file in your virtual machine's home directory (`~/`).

```bash
sha512sum *.deb > results.txt
```

Put the `results.txt` file into the `project-02/part-01` directory of your private repository with all the other files.

4. Commit and push your repository through Git. Submit a URL to the last commit on GitHub to Canvas before the deadline.

## Part #2

The goal of the second task is to port the `task_info` system calls' code from the previous part to the new version of the Linux kernel bundled with the Ubuntu 22.04 LTS distribution. Follow all the steps in the Readme file under the `ubuntu-jammy` directory to finish the task.

### What to Submit

1. Ensure that during the compilation process, you have specified your last name and the first letter of the first name in the `...debian.master/changelog` file in the correct form (all letters are in lowercase, no `_` or `-` characters between your last name and the first letter of the first name).

2. In your private course repository given to you by the instructor during the lecture, create the path `project-02/part-02/`.

3. Create a directory `project-02/part-02` in your private course repository. Put the following files into it from the Linux source tree (NOT from the `kernel-project` directory)

    * `task_info/`
    * `init/Kconfig`
    * `include/linux/syscalls.h`
    * `include/uapi/asm-generic/unistd.h`
    * `arch/x86/entry/syscalls/syscall_32.tbl`
    * `arch/x86/entry/syscalls/syscall_64.tbl`
    * `kernel/sys_ni.c`
    * `Makefile`
    * `debian.master/changelog`
    * `~/linux-image-*-generic_*_amd64.deb`

Create a `results.txt` file in your virtual machine's home directory (`~/`).

```bash
sha512sum *.deb > results.txt
```

Put the `results.txt` file into the `project-02/part-02` directory of your private repository with all the other files.

4. Commit and push your repository through Git. Submit a URL to the last commit on GitHub to Canvas before the deadline.

## Deadline

Check Canvas for information about the deadlines.
