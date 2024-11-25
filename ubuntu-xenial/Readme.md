COM 341, Operating Systems
==========================
# Project #2: [Process Control Block](https://elixir.bootlin.com/linux/v6.8/source/include/linux/sched.h#L748

![Compiling a Linux Kernel](http://i.imgur.com/t0c7Wav.gif)

### General Information

In Project #2, you will have to add two system calls to the Linux kernel. The system calls will allow a small task information utility to run in the user space querying information directly from the kernel without parsing output from the `proc` file system.

### Notes

Consider working in a terminal multiplexer such as `tmux`. In `tmux`, you can disconnect from the machine during a long-running process and go back to check the progress of the task at any later time.

Install a terminal multiplexer

    sudo apt install tmux

Type the following to start a new virtual terminal on your remote server

    tmux

To split the current pane into two vertically

    <CTRL+B> "

To split the current pane into two horizontally

    <CTRL+B> %

To switch to the pane above, below, to the left, or to the right of the current pane

    <CTRL+B><Up>
    <CTRL+B><Down>
    <CTRL+B><Left>
    <CTRL+B><Right>

To detach from a virtual terminal

    <CTRL+B><D>

To attach back

    tmux a

To exit from a virtual terminal

    exit

### Steps

1. Go to the directory with the [Vagrantfile](https://github.com/auca/com.341/blob/master/Other/Vagrantfile) of our course and start it with the usual `vagrant up` command. If you have any problems with the environment, you can use the course [video](https://youtu.be/VIsQ5v4JYF4) to set it up again.

2. Backup the old files from the home directory of the Vagrant machine to your host environment with the `scp` command or any other file transfer tool. We will be recreating the Vagrant machine with a bigger disk size for Project #2. If you want to keep your old lab, Project #1, or Midterm Exam files, transfer them to your host machine or somewhere safe from the Vagrant system.

3. Destroy the current Vagrant machine with the `vagrant destroy` command.

4. Open the current `Vagrantfile` in a text editor and change the disk size to `64GB` if not already set.

    Add the following line to the Vagrantfile

        config.vm.disk :disk, size: "64GB", primary: true

    after the lines

        config.vm.provider "virtualbox" do |vb|
            vb.gui    = true
            vb.cpus   = cpus
            vb.memory = memory
        end
        config.vm.provider "vmware_desktop" do |vmware|
            vmware.gui    = true
            vmware.cpus   = cpus
            vmware.memory = memory
        end

    Add the following lines

        # Take all the disk space available
        sudo pvresize /dev/sda3
        sudo lvresize -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
        sudo resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv

    after the line

        config.vm.provision "shell", inline: <<-SHELL

    and before the lines

        # Update and upgrade the system
        sudo apt-get update
        sudo apt-get upgrade -y

5. Adjust the number of CPUs and memory size in the Vagrantfile if necessary. The default values are `4` CPUs and `4096` MB of memory. You can lower or increase them based on your hardware. If the hardware permits, you can increase the values to speed up the compilation process of Project #2 considerably.

    Change the following lines in the Vagrantfile

        cpus   = 4      # Lower or increase based on your hardware
        memory = "4096" # Increase if your hardware permits

6. Start the new Vagrant machine with the `vagrant up` command. Ensure the installation process completes successfully with all the necessary packages installed. Reboot the machine at the end with the `vagrant reload` command.

7. Create a save point for your virtual machine named `before-starting-project-2`. You can use it to restore the machine to the current state with the `vagrant snapshot restore before-starting-project-2` command if something goes wrong during the project.

        vagrant snapshot save before-starting-project-2

8. Connect to your virtual machine with the `vagrant ssh` command or any other method you prefer described in the course video. We recommend NOT to copy back the old files from the host environment to the virtual machine at this point. It is better to have a clean environment for the project. Copy the files back only after you have successfully completed the project.

9. Consider starting using `tmux` at this point. See the notes above for more information about `tmux`.

10. Make the package manager to allow downloading package sources by adding the `deb-src` text after the lines `Types: deb` in the `/etc/apt/sources.list.d/ubuntu.sources` file.

        sudo vim /etc/apt/sources.list.d/ubuntu.sources

    Change the first line of the block

        Types: deb
        URIs: http://ports.ubuntu.com/ubuntu-ports/
        Suites: noble noble-updates noble-backports
        Components: main restricted universe multiverse
        Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

    to

        Types: deb deb-src
        URIs: http://ports.ubuntu.com/ubuntu-ports/
        Suites: noble noble-updates noble-backports
        Components: main restricted universe multiverse
        Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

    Change the first line of the next block

        Types: deb
        URIs: http://ports.ubuntu.com/ubuntu-ports/
        Suites: noble-security
        Components: main restricted universe multiverse
        Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

    to

        Types: deb deb-src
        URIs: http://ports.ubuntu.com/ubuntu-ports/
        Suites: noble-security
        Components: main restricted universe multiverse
        Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

11. Update package index and install updates to the system. Reboot the machine if a new kernel was installed with the `vagrant reload` command in the host environment.

        sudo apt update && sudo apt upgrade

12. Install `git`, `make`, `gcc`, and `ncurses-dev` if they are not installed.

        sudo apt install git make gcc ncurses-dev

13. Upload the `project-2-*` directory to the virtual machine from the host environment or clone the repository from the GitHub in the Vagrant environment.

        # From your Host machine
        scp -r project-2-* <vagrant machine name in the '.ssh/config' set from the 'vagrant ssh-config' output>:~/

    or

        # From your Vagrant machine
        git clone <the URL of the repository generated by the GitHub Classrom system>

14. Go to the directory with sources of the process information utility "tasks".

        cd ~/project-2-*/tasks

15. Compile the user space program.

        make

16. Ensure that it doesn't work.

        ./tasks

17. Go to your home directory.

        cd ~/

18. Set your name and e-mail address in Git if you haven't done it yet.

        git config --global user.name "your full name"
        git config --global user.email "your e-mail address"

19. Get the Linux kernel sources for Ubuntu 24.04.

        apt source linux-image-unsigned-$(uname -r)

20. Install prerequisites for building the kernel.

        sudo apt build-dep linux linux-image-unsigned-$(uname -r)
        sudo apt install libncurses-dev gawk flex bison openssl libssl-dev dkms libelf-dev libudev-dev libpci-dev libiberty-dev autoconf llvm linux-tools-common fakeroot

21. Go inside the kernel source tree.

        cd linux-*/

22. Copy system call entry points into the kernel source tree.

        cp -R ~/project-2-*/ubuntu-xenial/task_info ./

23. Add a kernel configuration entry to disable or enable the new subsystem.

        vim ./init/Kconfig

    Add the following

        config TASK_INFO
        	bool "task_info syscalls"
        	default y
        	help
        	  This option enables the task_info system calls which can be
        	  used to query task information directly from the kernel.

    at the end of the file. Adjust tabs and spaces.

24. Enable the system only for `amd64` and `arm64` system in the `./debian.master/config/annotations` file.

        vim ./debian.master/config/annotations

    Add the following

        CONFIG_TASK_INFO                                policy<{'amd64': 'y', 'arm64': 'y', 'armhf': 'n', 'ppc64el': 'n', 'riscv64': 'n', 's390x': 'n'}>

    at the end of the file. Adjust tabs and spaces.

25. In the same file, remove all the flavors that we are not going to build for. If you have an x86-64 machine, you can remove all the flavors except `amd64`. If you have an ARM machine (Apple Silicon Chips), you can remove all the flavors except `arm64`.

        vim ./debian.master/config/annotations

    Adjust the comment line

        # FLAVOUR: amd64-generic arm64-generic arm64-generic-64k armhf-generic ppc64el-generic riscv64-generic s390x-generic

    by deleting all the flavors that we are not going to build for. For example, if you have an x86-64 machine, you can remove all the flavors except `amd64`.

        # FLAVOUR: amd64-generic

    On ARM machines, you can remove all the flavors except `arm64`.

        # FLAVOUR: arm64-generic

    You can find the architecture of your machine by running the following command

        uname -a


26. If you are on an ARM64 architecture, you need to adjust the `debian.master/rules.d/arm64.mk` file. Remove the `flavours = generic-64k` line from the file. On AMD64, you can skip this step.

        vim ./debian.master/rules.d/arm64.mk

    Remove `generic-64k` from the line

        flavours        = generic generic-64k

    It should look like

        flavours        = generic

    at the end.

27. Add a forward declaration for the `task_info` structure and two function prototypes for `get_pids` and `get_task_info` at the end of `include/linux/syscalls.h`.

        vim ./include/linux/syscalls.h

    Add

        struct task_info;

    after

        struct mnt_id_req;

    Add the following

        /* task_info */
        asmlinkage long sys_get_pids(int pid_list_length, pid_t __user *output_pid_list);
        asmlinkage long sys_get_task_info(pid_t pid, struct task_info __user *output_task_info);

    after the line

        asmlinkage long sys_lsm_list_modules(u64 *ids, u32 *size, u32 flags);

28. Add generic table entries for two system calls before the first instance of `#undef __NR_syscalls` in `include/uapi/asm-generic/unistd.h`. Do not forget to increment the counter for the first `#define __NR_syscalls` by two.

        vim ./include/uapi/asm-generic/unistd.h

    Add

        /* task_info/get_pids.c */
        #define __NR_get_pids 462
        __SYSCALL(__NR_get_pids, sys_get_pids)
        /* task_info/get_task_info.c */
        #define __NR_get_task_info 463
        __SYSCALL(__NR_get_task_info, sys_get_task_info)

    before

        #undef __NR_syscalls

    Don't forget to adjust numbers `462` and `463` if necessary. Use successive values after the last system call in the file.

    Increment the value of the next instance of __NR_syscalls by two.

    For example, change

        #define __NR_syscalls 462

    to

        #define __NR_syscalls 464

29. Add x86 table entries for two system calls at the end of the `arch/x86/entry/syscalls/syscall_32.tbl` file.

        vim ./arch/x86/entry/syscalls/syscall_32.tbl

    Add the following

        462	i386	get_pids		sys_get_pids
        463	i386	get_task_info		sys_get_task_info

    at the end of the file. Adjust tabs and spaces.

    Don't forget to adjust numbers `462` and `463` if necessary. Use successive values after the last system call in the file.

29 Add x86-64 table entries for two system calls in `arch/x86/entry/syscalls/syscall_64.tbl`. Do it before the `# Due to a historical design error...` comment.

        vim ./arch/x86/entry/syscalls/syscall_64.tbl

    Add system call numbers for x86-64 architecture

        462	common	get_pids		sys_get_pids
        463	common	get_task_info		sys_get_task_info

    before

        #
        # Due to a historical design error, certain syscalls are numbered differently...

    Don't forget to adjust numbers `462` and `463` if necessary. Use successive values after the last system call in the file. Adjust tabs and spaces appropriately.

30. Add two fallback stubs at the end of `kernel/sys_ni.c`.

        vim ./kernel/sys_ni.c

    Add the following

        /* task_info */
        COND_SYSCALL(sys_get_pids);
        COND_SYSCALL(sys_get_task_info);

    after the line

        COND_SYSCALL(lsm_list_modules);

31. Modify the kernel build system to compile the new calls. Add the directory `task_info/` after the line `obj-y += kernel/` in the `Kbuild` file.

        vim ./Kbuild

    Add the line

        obj-y			+= task_info/

    after the line

        obj-y			+= kernel/

32. Add your AUCA login with a plus symbol (e.g., `+toksaitovd`) after the version number at the top of `debian.master/changelog` to identify your new kernel. Remove the underscore symbol from the login name if you have it. Your work will be graded based on a correct value here. Incorrect values will give you zero points for the work. Ensure to specify your login name correctly.

        vim ./debian.master/changelog

    Change the version at the top (the version can be different)

        linux (6.8.0-49.49) noble; urgency=medium

    to

        linux (6.8.0-49.49+toksaitovd) noble; urgency=medium

    Do not forget to adjust the login name appropriately.

33. Make Debian build scripts executable.

        chmod a+x debian/rules
        chmod a+x debian/scripts/*
        chmod a+x debian/scripts/misc/*

34. Clean the build directory.

        fakeroot debian/rules clean

35. Create default configuration files for the kernel.

        fakeroot debian/rules editconfigs

    For each prompt, ensure that the `task_info` option is selected, save the configuration and exit. Ignore any error for architectures that we are not building for. You can also disable generation of debug information to speed up the compilation process and minimize the size of the resulting kernel. Various drivers and subsystems that we don't need such as sound and graphics can also be disabled. Do this if you have a limited amount of time to compile the kernel or a limited amount of disk space. Note that disabling some drivers may make your system unbootable.

36. Start the kernel build process. A successful build will produce multiple
    `.deb` packages in the parent directory. Time the process.

        time fakeroot debian/rules binary-headers binary-generic binary-perarch

    The compilation process can take a lot of time. Compiled objects can use up to 40 gigabytes of disk space. The Debian build system not only builds the kernel but also packs everything into a set of installable `.deb` packages. All compilation errors will appear at this stage.

    Keep track of errors by watching the compilation log. The compilation process does not stop on error. Stop the process immediately if you see an error by pressing `CTRL+C` key combination.

    To restart an unsuccessful build, fix problems in your sources and start the build system again.

        time fakeroot debian/rules binary-headers binary-generic binary-perarch

    To restart just the unsuccessful kernel build, fix problems in your sources and start the build system again in the following way

        time fakeroot debian/rules binary-generic

37. The first problem you will encounter is a complaint that the variable `nr_threads` is undeclared in `get_pids.c`. Use the Elixir Linux Cross Reference for Linux kernel version [6.8.\*](https://elixir.bootlin.com/linux/v6.8/source) to search where `nr_threads` is declared. You should also find where the same variable was included for the [old version](https://elixir.bootlin.com/linux/v4.8/source) of the kernel used in Ubuntu 16.10. Remove the old included header (if necessary) and replace it with the new one in `get_pids.c`. Note, that the file that you include must be an `.h` file. The variable should also be declared as an `externvar` and not as a member in the header.

38. The second problem is that `linux/cputime.h` does not exist in the 6.8.* version of the kernel anymore that is included in `get_task_info.c`. To find where `cputime.h` code was moved into, search in the commit history of the Linux [kernel](https://github.com/torvalds/linux). I recommend using the following string `Move cputime functionality from` to narrow down your search. Fix the include after that. It also looks like that the `cputime_t` was removed and replaced with the `u64` typedef. Replace the type in `get_task_info.c`.

39. The third problem is that `cputime_to_usecs` is not used to convert time values in the new version of the kernel anymore. To help us find what should we use, we will refer to the Linux Cross Reference [again](https://elixir.bootlin.com/linux/v4.8/source). Find the `cputime_to_usecs` usage in 4.8 sources. Compare the same places in sources in the [6.8.](https://elixir.bootlin.com/linux/v6.8/source) version on the same site. Finally, apply the change to your code.

40. Finally, the logic to extract the process state information (outlined after the `/* state */` comment in `get_task_info.c`) was changed, and a getter `task_state_index(...)` was [created](https://github.com/torvalds/linux/commit/1d48b080bcce0a5e7d7aa2dbcdb35deefc188c3f) to acquire this information from the process control block (`struct task_struct` in Linux). Replace the code after `/* state */` to set the `local_task_info.state` field with the code to set this variable with the value returned by the getter mentioned above.

41. Fix the code issues and try rebuilding the kernel again.

42. Go to the parent directory.

        cd ..

43. Ensure that you have a number of newly created `.deb` packages.

        ls *.deb

44. In the host environment, create a save point for your virtual machine named `after-compiling-kernel`. You can use it to restore the machine to the current state with the `vagrant snapshot restore after-compiling-project-2-kernel` command if something goes wrong during the project.

        vagrant snapshot save after-compiling-project-2-kernel

45. Install the new kernel and all its supporting files.

        sudo dpkg -i *.deb

    In case of dependency errors, run the following

        sudo apt -f install

    and restart the installation process

        sudo dpkg -i *.deb

    If the package manager complains about the conflict with the existing kernel, remove the old kernel and install the new one.

        sudo apt remove linux-image-6.8.0-49.49-generic # or whatever version `uname -a` reports
        sudo apt autoremove
        sudo dpkg -i *.deb

    If you have any trouble removing the old kernel as it is the only one installed, you can force the removal of the package by adding the line `exit 0` at the beginning of the `prerm` check script of the package.

        sudo vim /var/lib/dpkg/info/linux-image-6.8.0-49-generic.prerm

    Add the line

        exit 0

    at the beginning of the file. Save and exit the editor. Try to remove the old package again.

        sudo dpkg --force-all -r linux-image-6.8.0-49-generic

    and restart the installation process

        sudo dpkg -i *.deb

46. Reboot the machine to start using the new kernel.

        sudo shutdown -r now

47. Reconnect to your machine.

        vagrant ssh

48. Go to the directory with sources of the process information utility "tasks".

        cd ~/project-*/tasks

49. Adjust the system call numbers that you have defined in the kernel. Change
    values for constants `__NR_get_pids` and `__NR_get_task_info` in `tasks.c`.

        vim tasks.c

50. Recompile the user space program.

        make clean
        make

51. Test the new system calls.

        ./tasks

    You should see a list of tasks currently running on the system.

    If you press `t`, you will probably discover invalid values in the core and virtual memory columns. We will leave it to you as an optional exercise to figure out the problem. But be warned, you will probably have to modify and rebuild the kernel again.

52. Go back to the kernel source tree.

### Submitting Work

Create a directory `ubuntu-noble` and put the following files into it:

* `ubuntu-noble/task_info/get_pids.c`
* `ubuntu-noble/task_info/get_task_info.c`
* `ubuntu-noble/task_info/Makefile`
* `ubuntu-noble/vmlinuz-6.8.0-*-generic`

Figure out on your own on where to find and how to extract the last file from your *.deb packages. Create all the necessary directories and subdirectories to match the structure of the files in the repository.

### Deadlines

Refer to Moodle for information about the deadline.

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
