### COM 341, Operating Systems
# Project #2

![Compiling a Linux Kernel](http://i.imgur.com/t0c7Wav.gif)

## Required Tools

Ensure that you have the Bash shell with common Unix utilities (GNU coreutils or
any alternatives), and the Git version control system installed on your system
together with an SSH client.

* On Windows, you can get the official Git [distribution](https://git-scm.com/downloads)
  with everything you need in one package.

* On macOS, you can get Git by installing Xcode or the Command Line Tools
  [package](https://developer.apple.com/opensource) from Apple.

* On Ubuntu Linux, you can install Git through the system's package manager with
  the command `sudo apt install git`.

macOS and Ubuntu Linux have Bash and basic Unix programs installed by default.

## Part #1

The goal of the first part is to modify an old version of the Linux kernel
bundled with the Ubuntu 16.10 GNU/Linux distribution by adding two custom system
calls to it. This will allow a small task information utility to query
information about running processes directly without consulting the proc
filesystem `procfs`. Follow all the steps in the Readme file under the
`ubuntu-yakkety` directory to finish the task.

### What to Submit

1. Ensure that during the compilation process you have specified your last name
   and the first letter of the first name in the `...debian.master/changelog`
   file in the correct form (all letters are in lowercase, no `_` or `-`
   characters between your last name and the first letter of the first name).

2. In your private course repository that was given to you by the instructor
   during the lecture, create the path `project-2/part-1/`.

3. In your private course repository, create a directory `project-2/part-1`. Put
   the following files into it from the Linux source tree (NOT from the
   `kernel-project` directory)

    * `task_info/`
    * `init/Kconfig`
    * `include/linux/syscalls.h`
    * `include/uapi/asm-generic/unistd.h`
    * `arch/x86/entry/syscalls/syscall_32.tbl`
    * `arch/x86/entry/syscalls/syscall_64.tbl`
    * `kernel/sys_ni.c`
    * `Makefile`
    * `debian.master/changelog`

Create a `results.txt` file from the home directory (`~/`) of your virtual
machine.

```bash
sha512sum *.deb > results.txt
```

Put the `results.txt` file into the `project-2/part-1` directory of your private
repository with all the other files.

4. Commit and push your repository through Git. Submit the last commit ID to
   Canvas before the deadline.

## Part #2

The goal of the second task is to port the `task_info` system calls' code from
the previous part to the new version of the Linux kernel bundled together with
the Ubuntu 20.04 LTS distribution. Follow all the steps in the Readme file under
the `ubuntu-focal` directory to finish the task.

### What to Submit

1. Ensure that during the compilation process you have specified your last name
   and the first letter of the first name in the `...debian.master/changelog`
   file in the correct form (all letters are in lowercase, no `_` or `-`
   characters between your last name and the first letter of the first name).

2. In your private course repository that was given to you by the instructor
   during the lecture, create the path `project-2/part-2/`.

3. In your private course repository, create a directory `project-2/part-2`. Put
   the following files into it from the Linux source tree (NOT from the
   `kernel-project` directory)

    * `task_info/`
    * `init/Kconfig`
    * `include/linux/syscalls.h`
    * `include/uapi/asm-generic/unistd.h`
    * `arch/x86/entry/syscalls/syscall_32.tbl`
    * `arch/x86/entry/syscalls/syscall_64.tbl`
    * `kernel/sys_ni.c`
    * `Makefile`
    * `debian.master/changelog`

Create a `results.txt` file from the home directory (`~/`) of your virtual
machine.

```bash
sha512sum *.deb > results.txt
```

Put the `results.txt` file into the `project-2/part-2` directory of your private
repository with all the other files.

4. Commit and push your repository through Git. Submit the last commit ID to
   Canvas before the deadline.

### Deadline

Check Canvas for information about the deadlines.
