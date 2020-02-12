Pop!_OS Kernel Builder
======================

This script simplifies the process of patching the kernel that comes with Pop!_OS. It's assumed that you're running it on a machine that is running the same version of Pop!_OS (and the same version of the kernel) whose kernel you wish to patch.

Compiling kernel packages requires a very large number of dependencies to be installed. I don't generally want all those packages on my machine, so I prefer to install the dependencies into a [chroot], then build the packages in the [chroot].

To break that down a bit, this script will:

- Use debootstrap to **create an Ubuntu chroot**.  
  The version of Ubuntu installed into the chroot matches the release on which the currently running version of Pop!_OS is based.
- Copy your local APT `sources.list` files into the chroot.  
  Why? We want to download the kernel from the Pop!_OS package repository (this is why you should run this script on a machine running Pop!_OS).
- **Install the build dependencies** into the chroot.
- **Download the source** for the **currently running kernel**.
- **Apply your `.patch` files** that you've put in the `patches` folder.
- **Build the kernel packages** for you.

Installation
------------

    $ git clone https://github.com/gma/pop-os-kernel-builder.git
    $ sudo apt install debootstrap

Usage Synopsis
--------------

    $ sudo ./build-kernel [ubuntu-codename] [path]

The `ubuntu-codename` argument defaults to the name of the Ubuntu release that the Pop!_OS distribution that you're running is based upon. You can check this by running `lsb_release -c -s`.

The `path` argument is the name of the folder in which the chroot is created. It defaults to `./<ubuntu-codename>-chroot`.

Example
-------

Imagine that you want to patch the kernel, then compile some new packages. This will do what you need:

    $ cd pop-os-kernel-builder
    $ cp /path/to/my-kernel.patch ./patches
    $ sudo ./build-kernel

Now go and put the kettle on — this is going to take a while...

Why?
----

These days there isn't much call for those of us who run Linux on the desktop to patch the kernel in a distribution like Pop!_OS.

In my case, I need to apply a [few tweaks] in order to support the hardware modification to my ThinkPad X230 that converted it to use a 1920x1080 screen.

I'm going to want to do it whenever Pop releases a new kernel. That means I need a process that's easy for me to kick off, with highly reproducible results.

[chroot]: https://en.wikipedia.org/wiki/Chroot
[few tweaks]: https://gist.github.com/gma/fcacfcf6ae7bb560f09bd525e760ec9f
