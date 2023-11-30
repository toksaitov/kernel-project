COM 341, Operating Systems
==========================
# Project #2 Part #2: [Stable API Nonsense](https://www.kernel.org/doc/Documentation/process/stable-api-nonsense.rst)

![Compiling a Linux Kernel](http://i.imgur.com/t0c7Wav.gif)

### General Information

In Part #2, you have to port the code of the `task_info` directory created for the older Ubuntu 16.10 to the newer kernel used by the 22.04 LTS version of Ubuntu Linux.

### Steps

1. Download an installation image for [Ubuntu 22.04](https://releases.ubuntu.com/jammy). We recommend to select the server image under the name `ubuntu-22.04.1-live-server-amd64.iso`.

2. Open Oracle VirtualBox and open a new machine creation dialog by clicking on the 'New' button on the toolbar.

3. Type a descriptive name, set the type to `Linux` and the version to `Ubuntu (64-bit)`.

4. Set `memory size` to 1024 megabytes or more. Click on `Create`.

5. Set `file size` for a new virtual disk to (!) 60 gigabytes or more. Once again, click on `Create`.

6. Right-click on your new machine and select `Settings...`.

7. Go to the `System` tab and choose `processor`. Set the first slider to the number of cores on your system.

8. Switch to the `Storage` tab, and select an empty disk under the IDE controller. On the right side, click on the disk icon and select `Choose a disk file...`. Open the disk image from step #1.

9. Go to the `Network` tab, click on `Advanced`, and open the `Port Forwarding` window. Click on the plus icon and add the following rule to forward traffic from your host machine on port 2222 to an SSH server on the guest machine on port 22.

        Name      Protocol    Host IP      Host Port    Guest IP     Guest Port
        Rule 1    TCP         127.0.0.1    2222         10.0.2.15    22

    Other options are possible, but they depend on your network setup (e.g., you can select the `Bridged Adapter` in the `Attached to` drop-down as an alternative).

10. Save your settings and use the `Start` button on the toolbar to start your virtual machine.

11. On the language selection dialog, set the system language to `English`.

12. On the keyboard layout selection dialog, set `English (US)`.

13. On the network configuration dialog, leave the default values as is.

14. Set proxy parameters. Ignore the dialog if you don't have a proxy server.

15. On the mirror configuration page, leave the default URL.

16. On the disk storage layout screen, ensure that `Use an entire disk` is selected and proceed.

17. Aknowledge the file system summary and confirm destructive operations.

18. Type your full name. Set a system hostname to identify the virtual machine on your network (e.g., `ubuntu-22-04`). Select a login name, and create a password for a new account on your virtual system.

19. Check `Install OpenSSH server`, ignore all the proposed snaps, and proceed.

20. Reboot the virtual machine at the end of the installation process. When asked, remove the disk image by clicking on a disk icon in the lower right corner and selecting `Remove disk from virtual drive`. It is also possible that the OS will remove the disk automatically for you. Press `Enter` if the system is not rebooting.

21. You can work directly in the VirtualBox window, or you can connect to your server through the SSH protocol.

    On your host machine (from Bash on Windows or any terminal on *nix platforms), use the following command to connect to your virtual server.

        ssh -p 2222 <the user name specified during installation>@127.0.0.1

    Consider starting using `tmux` at this point.

22. Uncomment every line that starts with `deb-src` except the line that ends with the word `partner` in the package manager's configuration file `/etc/apt/sources.list` and save it.

        sudo vim /etc/apt/sources.list

23. Update package index and install updates to the system

        sudo apt update && sudo apt upgrade

24. Install `git`, `make`, `gcc`, and `ncurses-dev`.

        sudo apt install git make gcc ncurses-dev

25. Upload the `kernel-project` directory to the virtual machine.

        scp -P 2222 -r kernel-project <the user name specified during installation>@127.0.0.1:~/

26. Go to the directory with sources of the process information utility "tasks".

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

31. Get the Linux kernel sources for Ubuntu 22.04.

        apt-get source linux-image-unsigned-$(uname -r)

32. Install prerequisites for building the kernel.

        sudo apt build-dep linux linux-image-unsigned-$(uname -r)
        sudo apt install libncurses-dev gawk flex bison openssl libssl-dev dkms libelf-dev libudev-dev libpci-dev libiberty-dev autoconf llvm linux-tools-common fakeroot

34. Go inside the kernel source tree.

        cd linux-*/

35. Copy system call entry points into the kernel source tree.

        cp -R ~/kernel-project/ubuntu-*/task_info ./

36. Add a kernel configuration entry to disable or enable the new subsystem.

        vim ./init/Kconfig

    Add the following

        config TASK_INFO
        	bool "task_info syscalls"
        	default y
        	help
        	  This option enables the task_info system calls which can be
        	  used to query task information directly from the kernel.

    at the end of the file. Adjust tabs and spaces.

37. Add a forward declaration for the `task_info` structure and two function prototypes for `get_pids` and `get_task_info` at the end of `include/linux/syscalls.h`.

        vim ./include/linux/syscalls.h

    Add

        struct task_info;

    after

        enum landlock_rule_type;

    Add the following

        /* task_info */
        asmlinkage long sys_get_pids(int pid_list_length, pid_t __user *output_pid_list);
        asmlinkage long sys_get_task_info(pid_t pid, struct task_info __user *output_task_info);

    at the end of the file before the closing `#endif`

38. Add generic table entries for two system calls before the first instance of `#undef __NR_syscalls` in `include/uapi/asm-generic/unistd.h`. Do not forget to increment the counter for the first `#define __NR_syscalls` by two.

        vim ./include/uapi/asm-generic/unistd.h

    Add

        /* task_info/get_pids.c */
        #define __NR_get_pids 449
        __SYSCALL(__NR_get_pids, sys_get_pids)
        /* task_info/get_task_info.c */
        #define __NR_get_task_info 450
        __SYSCALL(__NR_get_task_info, sys_get_task_info)

    before

        #undef __NR_syscalls

    Don't forget to adjust numbers `449` and `450` if necessary. Use successive values after the last system call in the file.

    Increment the value of the next instance of __NR_syscalls by two.

    For example, change

        #define __NR_syscalls 449

    to

        #define __NR_syscalls 451

39. Add x86 table entries for two system calls at the end of the `arch/x86/entry/syscalls/syscall_32.tbl` file.

        vim ./arch/x86/entry/syscalls/syscall_32.tbl

    Add the following

        449	i386	get_pids		sys_get_pids
        450	i386	get_task_info		sys_get_task_info

    at the end of the file. Adjust tabs and spaces.

    Don't forget to adjust numbers `449` and `450` if necessary. Use successive values after the last system call in the file.

40. Add x86-64 table entries for two system calls in `arch/x86/entry/syscalls/syscall_64.tbl`. Do it before the `# x32-specific system call num...` comment.

        vim ./arch/x86/entry/syscalls/syscall_64.tbl

    Add system call numbers for x86-64 architecture

        449	common	get_pids		sys_get_pids
        450	common	get_task_info		sys_get_task_info

    before

        #
        # Due to a historical design error, certain syscalls are numbered differently...

    Don't forget to adjust numbers `449` and `450` if necessary. Use successive values after the last system call in the file. Adjust tabs and spaces appropriately.

41. Add two fallback stubs at the end of `kernel/sys_ni.c`.

        vim ./kernel/sys_ni.c

    Add the following

        /* task_info */
        COND_SYSCALL(sys_get_pids);
        COND_SYSCALL(sys_get_task_info);

    at the end of the file

42. Modify the kernel build system to compile the new calls. Add the directory `task_info/` at the end of the line `core-y += kernel/ mm/ fs/ ipc/ security/ crypto/ block/` in the main `Makefile`.

        vim ./Makefile

    Change the line

        core-y += kernel/ certs/ mm/ fs/ ipc/ security/ crypto/

    to

        core-y += kernel/ certs/ mm/ fs/ ipc/ security/ crypto/ task_info/

43. Add your AUCA login with a plus symbol (e.g., `+toksaitovd`) after the version number at the top of `debian.master/changelog` to identify your new kernel. Your work will be graded based on a correct value here. Incorrect values will give you zero points for the work. Ensure to specify your login name correctly.

        vim ./debian.master/changelog

    Change the version at the top (the version can be different)

        linux (5.15.0-89.99) jammy; urgency=medium

    to

        linux (5.15.0-89.99+toksaitovd) jammy; urgency=medium

44. Make Debian build scripts executable.

        chmod a+x debian/rules
        chmod a+x debian/scripts/*
        chmod a+x debian/scripts/misc/*

45. Clean the build directory.

        LANG=C fakeroot debian/rules clean

46. Create default configuration files for the kernel.

        LANG=C fakeroot debian/rules editconfigs

    For each prompt, ensure that the `task_info` option is selected, save the configuration and exit.

47. Start the kernel build process. A successful build will produce multiple
    `.deb` packages in the parent directory. Time the process.

        LANG=C time fakeroot debian/rules binary-headers binary-generic binary-perarch

    The compilation process can take a lot of time. Compiled objects can use up to 40 gigabytes of disk space. The Debian build system not only builds the kernel but also packs everything into a set of installable `.deb` packages. All compilation errors will appear at this stage.

    Keep track of errors by watching the compilation log. The compilation process does not stop on error. Stop the process immediately if you see an error by pressing `CTRL+C` key combination.

    To restart an unsuccessful build, fix problems in your sources and start the build system again.

        LANG=C time fakeroot debian/rules binary-headers binary-generic binary-perarch

    To restart just the unsuccessful kernel build, fix problems in your sources and start the build system again in the following way

        LANG=C time fakeroot debian/rules binary-generic

48. The first problem you will encounter is a complaint that the variable `nr_threads` is undeclared in `get_pids.c`. Use the Elixir Linux Cross Reference for Linux kernel version [5.15.\*](https://elixir.bootlin.com/linux/v5.15.52/source) to search where `nr_threads` is declared. You should also find where the same variable was included for the [old version](https://elixir.bootlin.com/linux/v4.8/source) of the kernel used in Ubuntu 16.10. Remove the old included header (if necessary) and replace it with the new one in `get_pids.c`. Note, that the file that you include must be an `.h` file. The variable should also be declared as an `externvar` and not as a member in the header.

49. The second problem is that `linux/cputime.h` does not exist in the 5.15.* version of the kernel anymore that is included in `get_task_info.c`. To find where `cputime.h` code was moved into, search in the commit history of the Linux [kernel](https://github.com/torvalds/linux). I recommend using the following string `Move cputime functionality from` to narrow down your search. Fix the include after that. It also looks like that the `cputime_t` was removed and replaced with the `u64` typedef. Replace the type in `get_task_info.c`.

50. The third problem is that `cputime_to_usecs` is not used to convert time values in the new version of the kernel anymore. To help us find what should we use, we will refer to the Linux Cross Reference [again](https://elixir.bootlin.com/linux/v4.8/source). Find the `cputime_to_usecs` usage in 4.8 sources. Compare the same places in sources in the [5.15.](https://elixir.bootlin.com/linux/v5.15.52/source) version on the same site. Finally, apply the change to your code.

51. Finally, the logic to extract the process state information (outlined after the `/* state */` comment in `get_task_info.c`) was changed, and a getter `task_state_index(...)` was [created](https://github.com/torvalds/linux/commit/1d48b080bcce0a5e7d7aa2dbcdb35deefc188c3f) to acquire this information from the process control block (`struct task_struct` in Linux). Replace the code after `/* state */` to set the `local_task_info.state` field with the code to set this variable with the value returned by the getter mentioned above.

52. Fix the code issues and try rebuilding the kernel again.

53. Go to the parent directory.

        cd ..

54. Ensure that you have a number of newly created `.deb` packages.

        ls *.deb

55. Install the new kernel and all its supporting files.

        sudo dpkg -i *.deb

    In case of dependency errors, run the following

        sudo apt -f install

    and restart the installation process

        sudo dpkg -i *.deb

    If the new kernel version is lower than the current one, you may have to remove the current one and install your custom one.

         sudo apt remove linux-image-5.15.0-89-generic # or whatever version `uname -a` reports
         sudo apt autoremove
         sudo dpkg -i *.deb

    You may also select your kernel in the Grub bootloader. You must reboot the machine and make the Grub menu appear by pressing the arrow keys during the boot process. In the Grub menu, you may select the kernel to boot in.

56. Reboot the machine to start using the new kernel.

        sudo shutdown -r now

57. Reconnect to your machine.

        ssh -p 2222 <the user name specified during installation>@127.0.0.1

58. Go to the directory with sources of the process information utility "tasks".

        cd '~/kernel-project/tasks'

59. Adjust the system call numbers that you have defined in the kernel. Change
    values for constants `__NR_get_pids` and `__NR_get_task_info` in `tasks.c`.

        vim tasks.c

60. Recompile the user space program.

        make clean
        make

61. Test the new system calls.

        ./tasks

    You should see a list of tasks currently running on the system.

    If you press `t`, you will probably discover invalid values in the core and virtual memory columns. We will leave it to you as an optional exercise to figure out the problem. But be warned, you will probably have to modify and rebuild the kernel again.

62. Go back to the kernel source tree.

### Submitting Work

In your private course repository, create a directory `project-2/part-2`. Please put the following files into it.

* `task_info/`
* `init/Kconfig`
* `include/linux/syscalls.h`
* `include/uapi/asm-generic/unistd.h`
* `arch/x86/entry/syscalls/syscall_32.tbl`
* `arch/x86/entry/syscalls/syscall_64.tbl`
* `kernel/sys_ni.c`
* `Makefile`
* `debian.master/changelog`
* `~/linux-image-unsigned-*.deb`

Create a `results.txt` file from the home directory of your virtual machine.

```bash
sha512sum *.deb > results.txt
```

Put the `results.txt` file into the `project-2/part-2` directory of your private repository with all the other files.

Commit and push your work through Git. Submit the last commit ID to Canvas before the deadline.

### Deadlines

Refer to canvas for information about the deadline.

### Resources

* [Linux Cross Reference](http://lxr.free-electrons.com)

* [Linux Documentation, Adding a New System Call](https://github.com/torvalds/linux/blob/6f0d349d922ba44e4348a17a78ea51b7135965b1/Documentation/process/adding-syscalls.rst)

* [Linux Filesystem Hierarchy, /proc](http://www.tldp.org/LDP/Linux-Filesystem-Hierarchy/html/proc.html)

### Documentation

    man tmux

    man 2 syscall
    man 2 syscalls
    man 2 getpid
    man 2 sysinfo

### Reading

* _Understanding the Linux kernel, Third Edition by Daniel P. Bovet and Marco Cesati, Chapters 3, 4, 7, 10_

* _Linux Kernel Development, Third Edition by Robert Love, Chapters 3-5, 7_
