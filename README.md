#Writing a Kernel Module
California State University - Chico

By Bryan Dixon
***

##Introduction

The purpose of this assignment is for you to become more familiar with how kernel modules are written and aspects of the Linux operating system using these modules.
Logistics
The only “hand-in” will be electronic. Any clarifications and revisions to the assignment will be modified here and announced to the class via Piazza.

##Hand Out Instructions
For this assignment you will want to use a virtual machine (VM); however, if you are running a native install of 64bit Ubuntu 14.04.3 LTS you should be able to do this without using a VM. I would recommend using a VM as we are going to be modifying privileged code and you could potentially corrupt your native system if you aren’t using a VM.
There are no handout files for this assignment; however, on my webpage for this assignment there is a provided set of files for the hello world kernel module that you should build first to familiarize yourself with the basics of compiling & installing a compiled kernel module.

##Kernel Modules
There are two main ways to add code to the Linux kernel. One way is to choose or add code to compile into the kernel during the compilation process. The other method is to add the code to the Linux kernel while it is running, which is what a loadable kernel module is [8].

For more information on Linux kernel modules, I highly recommend reading this extremely good introduc- tion to Linux Loadable Kernel Modules and how they are commonly used:
[http://www.tldp.org/HOWTO/Module-HOWTO/x73.html](http://www.tldp.org/HOWTO/Module-HOWTO/x73.html)

##Your Task
For this assignment you will be doing the following:

1. Get the latest Linux kernel source for Ubuntu 14.04.3
2. Compile the latest Linux kernel source for Ubuntu 14.04.3
3. Compile a Hello World kernel module
4. Write a kernel module to create and modify a /proc file

In this document we will walk through the steps to do items 1-3 above. The code for the Hello World kernel module can be found on my website along with this writeup. Details on item 4 are found later in this document.


##Compile the Linux kernel from source

Your first step will be to download and install Ubuntu 14.04.1 64bit [6] onto your computer or VM (the Desktop and Server variants of Ubuntu will both work, but the Server variant is recommended because it requires less disk space). If you need help with this step please ask me to show you in lab or come to my office hours.
Once Ubuntu is installed, you will need to set up the installation’s build environment by running the follow- ing commands in a terminal window:

<pre>
  $ sudo apt-get update
  $ sudo apt-get dist-upgrade
  $ sudo apt-get install build-essential
  $ sudo apt-get build-dep linux-image-$(uname -r)
</pre>

These commands update the package list to make sure we have the most up-to-date list of packages, install the latest versions of all installed software, install some general-purpose build tools, and finally install the build dependencies for the Linux kernel itself. In this way, we insure all the necessary tools to download, build, and install the new Linux kernel are available on the system.

The sudo in the previous commands indicate we are invoking the given commands as the root user. sudo only works if your user account has sudoer privileges; if not, you will receive a message indicating the user is not in the sudoers file. This is usually not an issue in standard installation, but if you encounter this message, it is simple to give the current user permission to run commands with sudo [2].

Once the build environment is set up, you will need to download the source code for the Linux kernel you are currently running (we aren’t trying to compile and install a newer kernel, just re-compile the current kernel). Downloading the source code for the Linux kernel is a simple process, and very common for people who are running user-built (instead of package-maintained) Linux distros. Gentoo Linux is an example of such a distro if you are interested [3].

To get the source code for the currently running Linux kernel on Ubuntu 14.04, we will use apt-get, which will obtain the source for a specific binary package it provides:

<pre>
   ~$ mkdir kernel-assignment
   ~$ cd kernel-assignment
   ~/kernel-assignment$ apt-get source linux-image-$(uname -r)
</pre>

It is also common practice to obtain the Linux kernel source by checking out the current kernel source from the source tree on the official Linux git repository, but this assignment was tested with the apt-get approach so that’s what I’m giving for the instructions. The references contain a link to an Ubuntu wiki article about building your own kernel that has the git commands if you want to know what they are [5].

The apt-get command will take a few minutes to download the Linux source code. When the download process finishes, you should have some additional files and a source code folder in the current working directory:

<pre>
  ~/kernel-assignment$ ls -l
  total 121428
  drwxr-xr-x 26 user users      4096 Aug 21 12:00 linux-3.13.0
  -rw-r--r--  1 user users   7902814 Aug 13 15:58 linux_3.13.0-34.60.diff.gz
  -rw-r--r--  1 user users     11781 Aug 13 15:58 linux_3.13.0-34.60.dsc
  -rw-r--r--  1 user users 116419243 Feb  3  2014 linux_3.13.0.orig.tar.gz
</pre>

We are not going to modify the kernel’s configuration, so we can now move to building the new kernel. Ubuntu does this a bit differently than other kernels I’ve built, which usually have a make directive to make the configuration, which can be the default or modified by you, and a second make directive to build the kernel. In this case, we will build the kernel using the following commands:

<pre>
  $ cd linux-3.13.0
  $ fakeroot debian/rules clean
  $ fakeroot debian/rules binary-headers binary-generic
</pre>

The first step above changes the working directory to be the root of the kernel source tree, which is the linux-3.13.0 folder in this case. The build commands will take quite a while to run; on my fastest computer, it took an hour and half to build the .deb files. If you get errors during the compilation, please post about them in the class discussion board and see me in office hours or during lab so we can track down and fix the issue.
When the build process is complete (hopefully without any errors), there will be numerous .deb files in the parent directory of the kernel source tree (this parent directory will be the kernel-assignement directory we created when downloading the kernel source code):

<pre>
  ~/kernel-assignment/linux-3.13.0$ cd ..
  ~/kernel-assignment$ ls -l
  drwxr-xr-x 26 user users      4096 Jan 22 15:08 linux-3.13.0
  -rw-r--r--  1 user users   8405184 Dec 16 13:03 linux_3.13.0-34.60.diff.gz
  -rw-r--r--  1 user users     11783 Dec 16 13:03 linux_3.13.0-34.60.dsc
  -rw-r--r--  1 user users 116419243 Feb  3  2014 linux_3.13.0.orig.tar.gz
  -rw-r--r--  1 user users    174210 Jan 22 15:09 linux-cloud-tools-3.13.0-xxx.deb
  -rw-r--r--  1 user users   9058856 Jan 22 14:34 linux-headers-3.13.0-xxx.deb
  -rw-r--r--  1 user users    865124 Jan 22 15:09 linux-headers-3.13.0-xxx.deb
  -rw-r--r--  1 user users  15483680 Jan 22 15:09 linux-image-3.13.0-xxx.deb
  -rw-r--r--  1 user users  36949560 Jan 22 15:09 linux-image-extra-3.13.0-xxx.deb
  -rw-r--r--  1 user users    174282 Jan 22 15:09 linux-tools-3.13.0-44-xxx.deb
</pre>

The .deb files contain the compiled kernel, which we can now install and run (after rebooting). To install the new kernel, we will use the following commands:

<pre>
  ~/kernel-assignment$ sudo dpkg -i linux-headers-*.deb
  ~/kernel-assignment$ sudo dpkg -i linux-image-*.deb
  ~/kernel-assignment$ sudo reboot
</pre>

After the computer reboots, we’ll be using the new kernel you compiled. You can now brag about your 1337 or leet status as a CS major and the fact you have compiled the Linux kernel from source.

##Compile Hello World kernel module
Now let’s get the Hello World kernel module source and Makefile files from my web server and work on compiling a Linux kernel module. You will need to download the helloworld.tar file from my website:

[http://bryancdixon.com/site_media/Fall2014/CSCI340/helloworld.tar](http://bryancdixon.com/site_media/Fall2014/CSCI340/helloworld.tar)

You could download the file from the link above to your local computer, but I would recommend downloading it directly to your Ubuntu VM so you can make use of it with having to worry about copying the files onto the VM. To do this you can use the wget command with that URL as the argument to the command and it’ll download the helloworld.tar file to your current working directory.
Once you have the tar file you will want to extract it:

<pre>
  ~$ wget http://bryancdixon.com/site_media/Fall2014/CSCI340/helloworld.tar
  ~$ tar xvf helloworld.tar
  x helloworld/
  x helloworld/hello.c
  x helloworld/Makefile
</pre>

You should now have a helloworld folder in your current working directory. At this point you’ll want to likely take a look at the helloworld.c source file and the Makefile to familiarize yourself with the workings of these two files. These two files are also a good starting point for the final part of this assignment.

To build the Hello World kernel module you will need to change your working directory to be in the helloworld folder. Building kernel module is as simple as just typing make:

<pre>
  $ cd helloworld
  $ make
</pre>

This will build numerous files:

<pre>
  ~/helloworld$ ls
  hello.c  hello.ko  hello.mod.c  hello.mod.o  hello.o
  Makefile  modules.order  Module.symvers
</pre>

The only generated file that we care about is the hello.ko file, which is a kernel object file. We can now install the generated helloworld kernel module by using the insmod command:

<pre>
  ~/helloworld$ sudo insmod hello.ko
  ~/helloworld$ sudo rmmod hello
  ~/helloworld$ dmesg | tail
  ...
  [ 6814.354580] Hello world!
  [ 6819.571911] Cleaning up module.
</pre>

In the above example, I inserted the Hello World kernel module with the insmod command, immediately removed it with the rmmod command, and finally inspected the dmesg output to find the printk statements that printed ”Hello World!” when the module was installed and the cleanup message when the module was removed. In the above output, this process completed successfully.
You may see a warning in the dmesg output:

<pre>
  ~$ dmesg | tail
  ...
  [  258.556284] hello: module verification failed: signature and/or
  required key missing - tainting kernel
  [  258.558168] Hello world!
  [  265.987671] Cleaning up module.
</pre>

This warning can be safely ignored.

##Write your own kernel module

Now for the hard part: using the skills you’ve gained in this assignment so far, resources provided later in the hints section, and some details from lab you’ll now need to write your own Linux kernel module to provide us a system statistic in a /proc system file [1].

When we insert your module for grading it should create a new entry in the /proc filesystem called:

<pre>
  /proc/num_pagefaults
</pre>

We should then be able to cat or examine the contents of that file and it should provide us the number of page faults that the operating system has handled since it booted. As an example:

<pre>
  ~$ ls -al /proc/num_pagefaults
  ls: /proc/num_pagefaults: No such file or directory
  ~$ insmod ./pagefault.ko
  ~$ ls -al /proc/num_pagefaults
  -r--r--r--  1 root root 37 August 22 21:38 /proc/num_pagefaults
  ~$ cat /proc/num_pagefaults
  658103
  ~$ cat /proc/num_pagefaults
  658295
  ~$ cat /proc/num_pagefaults
  658485
  ~$ rmmod page_fault_module
  ~$ ls -al /proc/num_pagefaults
  ls: /proc/num_pagefaults: No such file or directory
  ~$
</pre>

It’s worth thinking of this problem in a few steps:

1. Read about how the /proc filesystem works
2. Figure out how you write information to a /proc file
3. Write a kernel module that successfully prints a fixed string when one cat’s the /proc/num pagefaults file.
4. Locate the kernel code that generates page faults statistics
5. Write a kernel module that prints that statistic every time someone cat’s the /proc/num pagefaults file.

Good luck!

##Hints
There are quite a few hints for this assignment:

1. When compiling the Linux kernel, make sure the virtual disk for the VM has plenty of space. The default disk size of 20 GB is probably sufficient.
2. Take a snapshot of the VM once the base system is installed and configured. This snapshot will be quite handy if something gets fouled up during the process of building the kernel.
3. Take a snapshot of the VM before installing the new kernel. This snapshot will be quite handy if something gets fouled up during the process of installing the kernel.
4. Be sure to successfully complete the kernel compilation portion of the assignment before attempting to compile the Hello World kernel module. In particular, the header files for the Linux kernel must be installed for kernel modules to build correctly.
5. You will need to find the symbol that has been explicitly exported by the kernel to be accessible to kernel modules; not all functions and variables in the kernel code are accessible to kernel modules. The kernel uses the EXPORT SYMBOL macro to export a particular symbol, so you need to find the specific symbol that’s been exported as such that provides the page fault statistic we want.
6. You may need to declare your kernel module is licensed under the GPL open source license, as some kernel symbols are only accessible if you have declared your kernel module as being licensed under the GPL license. To declare it you only need to add a single line at the end of your kernel module code[4]:
<pre>    MODULE_LICENSE("GPL");</pre>
7. The Linux kernel already includes the page fault statistic in a /proc file, along with numerous other statistics. It is useful to see how this is already done and see if you can modify it to make a new /proc file that contains the current number of page faults only. The following command provides a good reference for the comparing the kernel statistics with those of your kernel module:
     ̃$ cat /proc/vmstat | grep pgfault
    pgfault 2301445
8. The /proc filesystem is not persistent storage in the same sense as an NTFS or ext3 filesystem, so don’t bother searching for file I/O tutorials. To quote a good reference on the proc system[7]:
/proc is very special in that it is also a virtual filesystem. It’s sometimes referred to as a process information pseudo-file system. It doesn’t contain ’real’ files but runtime system information (e.g. system memory, devices mounted, hardware configuration, etc). For this reason it can be regarded as a control and information centre for the kernel.
￼
Instead of performing standard file I/O, we need to register event handlers for various events. The seq file API[9] introduced in Linux 3.10 is quite useful for this. [version.c](https://github.com/torvalds/linux/blob/master/fs/proc/version.c) is a simple example that can be readily adapted to complete the pagefault portion of the assignment.
9. The only function you really need to get the pagefault stat is all vm events[10], which populates an array of longs with various statistical information. You just need to invoke the function, then index to the correct array element in the result; in our case, PGFAULT is the index we need. Examples of all vm events usage are available in the Linux kernel source[11].
You’llneedto#include <linux/mm.h>inyourkernelmoduletoaccesstheallvmevents function.
10. seq printf uses the same format specifiers as printf. The format specifier for an unsigned long is just a Google search away.

##Evaluation
You will be graded based on your success in completing various steps of this assignment. The scoring for this assignment is as follows:

* 20% If you successfully compile a new kernel from source
* 40% You also successfully compile the Hello World kernel module
* 70% You also successfully write your own kernel module that creates the correct /proc file with a fixed value reported.
* 100% You successfully get everything else working and your kernel module correctly writes the number of page faults to the /proc file requested.

##Hand In Instructions
You will need to submit your assignment by committing your code, and compiled .deb and .ko files to the GitHub private repo you were invited to. Your repo must be part of the CSUChico-CSCI340 organization on GitHub; if this is not the case, you have put your files in the wrong repository. If you do not have a repository in this organization, make sure you filled out the GitHub request on my website for this semester.

The files should be organized with the source files for your kernel module in a src/ folder in your repo, and the compiled dpkg (.deb) files and kernel object (.ko) files in the root directory.
Here is an example of a correctly structured repository:

<pre>
  /
  ....src/
  ........numpagefaults.c
  ........Makefile
  ....helloworld/
  ........hello.c
  ........Makefile
  ....hello.ko
  ....linux-cloud-tools-3.13.0-xxx.deb
  ....linux-headers-3.13.0-xxx.deb
  ....linux-headers-3.13.0-xxx.deb
  ....linux-image-3.13.0-xxx.deb
  ....linux-image-extra-3.13.0-xxx.deb
  ....linux-tools-3.13.0-44-xxx.deb
  ....numpagefaults.ko
</pre>

For ease of grading, you should name your .ko files as shown. All of the source code for your kernel module must be in the src/ folder as shown. You can name your source files any way you choose, but the compiled files must be named as shown.


##References
1. Steve Gribble CSE551 Spring 2007 - Programming Assignment #2 https://courses.cs. washington.edu/courses/cse551/07sp/programming/a2.html. Online; accessed 22- August-2014
2. Allowing other users to run sudo https://help.ubuntu.com/community/RootSudo# Allowing_other_users_to_run_sudo. Online; accessed 19-January-2015.
3. Gentoo Linux https://www.gentoo.org/. Online; accessed 21-August-2014.
4. The GNU General Public License http://www.gnu.org/copyleft/gpl.html Online; ac-
cessed 23-August-2014
5. Ubuntu Wiki “Build Your Own Kernel”. https://wiki.ubuntu.com/Kernel/
BuildYourOwnKernel. Online; accessed 21-August-2014.
6. Ubuntu “Download Ubuntu Server 14.04.1 LTS”. http://www.ubuntu.com/download/
server. Online; accessed 21-August-2014.
7. Linux Filesystem Hierarchy ”Chapter 1: Linux Filesystem Hierarchy - 1.14. /proc http://www. tldp.org/LDP/Linux-Filesystem-Hierarchy/html/proc.html Online; accessed 20- January-2015.
8. Linux Loadable Kernel Module HOWTO “Introduction to Linux Loadable Kernel Modules”. http: //www.tldp.org/HOWTO/Module-HOWTO/x73.html. Online; accessed 21-August-2014.
9. “Manage /proc file with seq file”. http://www.tldp.org/LDP/lkmpg/2.6/html/x861. html. Online; accessed 29-January-2015.
10. https://github.com/torvalds/linux/blob/33caee39925b887a99a2400dc5c980097c3573f9 mm/vmstat.c#L50. Online; accessed 27-January-2015.
11. https://github.com/torvalds/linux/blob/9a3c4145af32125c5ee39c0272662b47307a8323 arch/s390/appldata/appldata_mem.c#L75. Online; accessed 27-January-2015.
