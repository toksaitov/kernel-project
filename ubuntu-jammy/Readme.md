COM 341, Operating Systems
==========================
# Project #2 Part #2: [Stable API Nonsense](https://www.kernel.org/doc/Documentation/process/stable-api-nonsense.rst)

![Compiling a Linux Kernel](http://i.imgur.com/t0c7Wav.gif)

### General Information

In this task you need to port the code of the `task_info` directory created for
the older kernels used by the Ubuntu 16.10 to work on the kernel used by the
newer 20.04 LTS version of Ubuntu Linux.

### Steps

1. Download an installation image for [Ubuntu 22.04](https://releases.ubuntu.com/jammy)
   We recommend to select the server image under the name `ubuntu-22.04.1-live-server-amd64.iso`.

2. Open Oracle VirtualBox and open a new machine creation dialog by clicking on
   the 'New' button on the toolbar.

3. Type a descriptive name, set type to `Linux` and version to `Ubuntu
   (64-bit)`.

4. Set `memory size` to 1024 megabytes or more. Click on `Create`.

5. Set `file size` for a new virtual disk to (!) 60 gigabytes or more. Once
   again, click on `Create`.

6. Right click on your new machine and select `Settings...`.

7. Go to the `System` tab, select `processor`. Set the first slider to the
   number of cores that you have on your system.

8. Switch to the `Storage` tab, select an empty disk under the IDE controller.
   On the right side, click on the disk icon and select `Choose a disk
   file...`. Open the disk image from step #1.

9. Go to the `Network` tab, click on `Advanced`, open the `Port Forwarding`
   window. Click on the plus icon and add the following rule to forward traffic
   from your host machine on port 2222 to an SSH server on the guest machine on
   port 22.

        Name      Protocol    Host IP      Host Port    Guest IP     Guest Port
        Rule 1    TCP         127.0.0.1    2222         10.0.2.15    22

    Other options are possible but they depend on your network setup (e.g., you
    can select the `Bridged Adapter` in the `Attached to` drop-down as an
    alternative).

10. Save your settings and use the `Start` button on the toolbar to start your
    virtual machine.

11. On the language selection dialog, set the system language to `English`.

12. On the keyboard layout selection dialog, set `English (US)`.

13. On the network configuration dialog, leave the default values as is.

14. Set proxy parameters. Ignore the dialog if you don't have a proxy server.

15. On the mirror configuration page, leave the default URL.

16. On the disk storage layout screen ensure that `Use an entire disk` is
    selected and proceed.

17. Aknowledge the file system summary and confirm destructive operations.

18. Type your full name. Set a system host name to identify the virtual machine
    on your network (e.g., `ubuntu-20-04`). Select a login name, create a
    password for a new account on your virtual system.

19. Check `Install OpenSSH server`, ignore all the proposed snaps, proceed.

20. Reboot the virtual machine at the end of the installation process. When
    asked, remove the disk image by clicking on a disk icon in the lower right
    corner and selecting `Remove disk from virtual drive`. It is also possible
    that the OS will remove the disk automatically for you. Press `Enter` if
    the system is not rebooting.

21. You can work directly in the VirtualBox window, or you can connect to your
    server through the SSH protocol.

    On your host machine (from Bash on Windows or any terminal on *nix
    platforms) use the following command to connect to your virtual server.

        ssh -p 2222 <the user name specified during installation>@127.0.0.1

    Consider starting using `tmux` at this point.

22. Uncomment every line that starts with `deb-src` except the line that ends
    with the word `partner` in the package manager's configuration file
    `/etc/apt/sources.list` and save it.

        sudo vim /etc/apt/sources.list

23. Update package index and install updates to the system

        sudo apt update && sudo apt upgrade

24. Install `git`, `make`, `gcc`, and `ncurses-dev`.

        sudo apt install git make gcc ncurses-dev

25. Upload the `kernel-project` directory to the virtual machine.

        scp -P 2222 -r kernel-project <the user name specified during installation>@127.0.0.1:~/

26. Go to the directory with sources of the process information utility
    "tasks".

        cd ~/kernel-project/tasks

27. Compile the user space program.

        make

28. Ensure that it doesn't work.

        ./tasks

29. Go to your home directory.

        cd ~/

30. Set your name and e-mail address in Git.

        git config --global user.name "your full name"
        git config --global user.email "your e-mail address"

31. Clone the Linux kernel repository for Ubuntu 16.10. Note that Git will try
    to fetch around 200 megabytes of data from the Canonical servers (the company
    behind the Ubuntu OS).

        git clone --depth 1 git://git.launchpad.net/~ubuntu-kernel/ubuntu/+source/linux/+git/jammy ubuntu-jammy

32. Install prerequisites for building the kernel.

        sudo apt build-dep linux linux-image-$(uname -r)
        sudo apt install libncurses-dev linux-tools-common fakeroot

33. Go inside the kernel source tree.

        cd ubuntu-*

34. Copy system call entry points into the kernel source tree.

        cp -R ~/kernel-project/ubuntu-*/task_info ~/ubuntu-*/

35. Add a kernel configuration entry to disable or enable the new subsystem.

        vim ~/ubuntu-*/init/Kconfig

    Add the following

        config TASK_INFO
            bool "task_info syscalls"
            default y
            help
              This option enables the task_info system calls which can be
              used to query task information directly from the kernel.

    at the end of the file. Adjust tabs and spaces.

36. Add a forward declaration for the `task_info` structure and two function
    prototypes for `get_pids` and `get_task_info` at the end of
    `include/linux/syscalls.h`.

        vim ~/ubuntu-*/include/linux/syscalls.h

    Add

        struct task_info;

    after

        enum landlock_rule_type;

    Add the following

        /* task_info */
        asmlinkage long sys_get_pids(int pid_list_length, pid_t __user *output_pid_list);
        asmlinkage long sys_get_task_info(pid_t pid, struct task_info __user *output_task_info);

    at the end of the file before the closing `#endif`

37. Add generic table entries for two system calls before the first instance of
    `#undef __NR_syscalls` in `include/uapi/asm-generic/unistd.h`. Do not forget
    to increment the counter for the first `#define __NR_syscalls` by two.

        vim ~/ubuntu-*/include/uapi/asm-generic/unistd.h

    Add

        /* task_info/get_pids.c */
        #define __NR_get_pids 449
        __SYSCALL(__NR_get_pids, sys_get_pids)
        /* task_info/get_task_info.c */
        #define __NR_get_task_info 450
        __SYSCALL(__NR_get_task_info, sys_get_task_info)

    before

        #undef __NR_syscalls

    Don't forget to adjust numbers `449` and `450` if necessary. Use successive
    values after the last system call in the file.

    Increment the value of the next instance of __NR_syscalls by two.

    For example, change

        #define __NR_syscalls 449

    to

        #define __NR_syscalls 451

38. Add x86 table entries for two system calls at the end of the
    `arch/x86/entry/syscalls/syscall_32.tbl` file.

        vim ~/ubuntu-*/arch/x86/entry/syscalls/syscall_32.tbl

    Add the following

        449    i386    get_pids        sys_get_pids
        450    i386    get_task_info   sys_get_task_info

    at the end of the file. Adjust tabs and spaces.

    Don't forget to adjust numbers `449` and `450` if necessary. Use successive
    values after the last system call in the file.

39. Add x86-64 table entries for two system calls in
    `arch/x86/entry/syscalls/syscall_64.tbl`. Do it before the
    `# x32-specific system call num...` comment.

        vim ~/ubuntu-*/arch/x86/entry/syscalls/syscall_64.tbl

    Add system call numbers for x86-64 architecture

        449    common    get_pids             sys_get_pids
        450    common    get_task_info        sys_get_task_info

    before

        #                                                                                
        # Due to a historical design error, certain syscalls are numbered differently...

    Don't forget to adjust numbers `449` and `450` if necessary. Use successive
    values after the last system call in the file. Adjust tabs and spaces
    appropriately.

40. Add two fallback stubs at the end of `kernel/sys_ni.c`.

        vim ~/ubuntu-*/kernel/sys_ni.c

    Add the following

        /* task_info */
        COND_SYSCALL(sys_get_pids);
        COND_SYSCALL(sys_get_task_info);

    at the end of the file

41. Modify the kernel build system to compile the new calls. Add the
    directory `task_info/` at the end of the line
    `core-y += kernel/ mm/ fs/ ipc/ security/ crypto/ block/` in the main
    `Makefile`.

        vim ~/ubuntu-*/Makefile

    Change the line

        core-y += kernel/ certs/ mm/ fs/ ipc/ security/ crypto/ block/

    to

        core-y += kernel/ certs/ mm/ fs/ ipc/ security/ crypto/ block/ task_info/

42. Add your AUCA login with a plus symbol (e.g., `+toksaitovd`) after the
    version number at the top of `debian.master/changelog` to identify your new
    kernel. You work will be graded based on a correct value here. Incorrect
    values will give you zero points for the work. Ensure to specify your login
    name correctly.

        vim ~/ubuntu-*/debian.master/changelog

    Change the version at the top (the version can be different)

        linux (5.15.0-52.58) jammy; urgency=medium

    to

        linux (5.15.0-52.58+toksaitovd) jammy; urgency=medium

43. Ensure that you are in the root directory of the kernel source tree.

        cd ~/ubuntu-*

44. Make Debian build scripts executable.

        chmod a+x debian/scripts/*
        chmod a+x debian/scripts/misc/*

45. Clean the build directory.

        fakeroot debian/rules clean

46. Create default configuration files for the kernel.

        fakeroot debian/rules editconfigs

    For each prompt, ensure that the `task_info` option is selected, save the
    configuration and exit.

47. Start the kernel build process. A successful build will produce multiple
    `.deb` packages in the parent directory. Time the the process.

        time fakeroot debian/rules binary-perarch \
                                   binary-indep   \
                                   binary-headers \
                                   binary-generic

    The compilation process can take a lot of time. Compiled objects can use up
    to 10 gigabytes of disk space. The Debian build system not only builds the
    kernel but also packs everything into a set of installable `.deb` packages.
    All compilation errors will appear at this stage.

    To restart an unsuccessful build, fix problems in your sources and start the
    build system again.

        time fakeroot debian/rules binary-perarch \
                                   binary-indep   \
                                   binary-headers \
                                   binary-generic

    To restart just the unsuccessful kernel build, fix problems in your sources
    and start the build system again in the following way

        time fakeroot debian/rules binary-generic

    To recompile the kernel with new changes after a successful build, remove a
    stamp file to notify the build system that kernel sources were changed.
    After that, you can start the build process again.

        rm debian/stamps/stamp-build-generic

        time fakeroot debian/rules binary-perarch \
                                   binary-indep   \
                                   binary-headers \
                                   binary-generic

48. The first problem that you will encounter is a complain that the variable
    `nr_threads` is undeclared in `get_pids.c`. Use the Elixir Linux Cross
    Reference for Linux kernel version [5.15.*](https://elixir.bootlin.com/linux/v5.15.52/source)
    to search where `nr_threads` is declared. You should also find where the
    same variable was included for the [old version](https://elixir.bootlin.com/linux/v4.8/source)
    of the kernel used in Ubuntu 16.10. Remove the old included header (if
    necessary) and replace it with the new one in `get_pids.c`. Note, that
    the file that you include must be an `.h` file. The variable should also
    be declared as an `externvar` and not as a member in the header.

49. The second problem is that `linux/cputime.h` does not exist in the 5.15.*
    version of the kernel anymore that is included in `get_task_info.c`. To find
    where `cputime.h` code was moved into, search in the commit history of the
    Linux [kernel](https://github.com/torvalds/linux). I recommend to use the
    following string `Move cputime functionality from` to narrow down your
    search. Fix the include after that. It also looks like that the `cputime_t`
    was removed and replaced with the `u64` typedef. Replace the type in
    `get_task_info.c`.

50. The third problem is that `cputime_to_usecs` is not used to convert time
    values in the new version of the kernel anymore. To help us find what
    should we use, we will refer to the Linux Cross Reference [again](https://elixir.bootlin.com/linux/v4.8/source).
    Find the `cputime_to_usecs` usage in 4.8 sources. Compare the same places
    in sources in the 5.15.* version on the same site. Apply the same change to
    your code.
    
51. Finally, the logic to extract the process state information (outlined after the `/* state */` comment in
    `get_task_info.c`) was changed, and a getter `task_state_index(...)` was [created](https://github.com/torvalds/linux/commit/1d48b080bcce0a5e7d7aa2dbcdb35deefc188c3f) to aquire
    this information from the process control block (`struct task_struct` in Linux). Replace the code after
    `/* state */` to set the `local_task_info.state` variable with the code to set this variable with the value
    returned by `task_state_index(...)`.

51. Go to the parent directory.

        cd ..

52. Ensure that you have a number of newly created `.deb` packages.

        ls *.deb

53. Install the new kernel and all its supporting files.

        sudo dpkg -i *.deb

    In case of dependency errors, run the following

        sudo apt -f install

    and restart the installation process

        sudo dpkg -i *.deb

    If the versions of the kernels are the same, you may
    have to remove the old one first, and then install
    your custom one.

         sudo apt remove linux-image-5.4.0-91-generic
         sudo dpkg -i *.deb

54. Reboot the machine to start using the new kernel.

        sudo shutdown -r now

55. Reconnect to your machine.

        ssh -p 2222 <the user name specified during installation>@127.0.0.1

56. Go to the directory with sources of the process information utility "tasks".

        cd '~/kernel-project/tasks'

57. Adjust system call numbers that you have defined in the kernel. Change
    values for constants `__NR_get_pids` and `__NR_get_task_info` in `tasks.c`.

        vim tasks.c

58. Recompile the user space program.

        make clean
        make

59. Test the new system calls.

        ./tasks

    You should see a list of tasks that are currently running on the system.
    The program uses your new system calls to get information directly from the
    kernel without going through a `proc` virtual file system (which is how
    programs such as `ps` or `top` work). You can scroll through the list with
    the arrow keys, show or hide kernel threads with the `t` key, or exit by
    pressing `q`.

60. Go back to the kernel source tree.

        cd ~/ubuntu-*

### Submitting Work

In your private course repository, create a directory `project-02/part-02`. Put
the following files into it.

* `task_info/`
* `init/Kconfig`
* `include/linux/syscalls.h`
* `include/uapi/asm-generic/unistd.h`
* `arch/x86/entry/syscalls/syscall_32.tbl`
* `arch/x86/entry/syscalls/syscall_64.tbl`
* `kernel/sys_ni.c`
* `Makefile`
* `debian.master/changelog`

Create a `results.txt` file from the home directory of your virtual machine.

```bash
sha512sum *.deb > results.txt
```

Put the `results.txt` file into the `project-02/part-02` directory of your
private repository with all the other files.

Commit and push your work through Git. Submit the last commit ID to Canvas
before the deadline.

### Deadlines

Refer to canvas for information about the deadline.

### Resources

* [Linux Cross Reference](http://lxr.free-electrons.com)

* [Linux Documentation, Adding a New System Call](https://github.com/torvalds/linux/blob/6f0d349d922ba44e4348a17a78ea51b7135965b1/Documentation/process/adding-syscalls.rst)

* [Linux Filesytem Hierarchy, /proc](http://www.tldp.org/LDP/Linux-Filesystem-Hierarchy/html/proc.html)

### Documentation

    man tmux

    man 2 syscall
    man 2 syscalls
    man 2 getpid
    man 2 sysinfo

### Reading

* _Understanding the Linux kernel, Third Edition by Daniel P. Bovet and Marco
  Cesati, Chapters 3, 4, 7, 10_

* _Linux Kernel Development, Third Edition by Robert Love, Chapters 3-5, 7_
