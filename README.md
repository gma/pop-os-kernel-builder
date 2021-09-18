Pop!_OS Kernel Builder
======================

The scripts in this project simplify the process of **patching the kernel that comes with Pop!_OS**.

Compiling kernel packages requires a very large number of dependencies to be installed. I don't generally want all those packages on my machine, so I prefer to install the dependencies and build the kernel in a [chroot].

To break that down a bit, these scripts will:

- Use debootstrap to **create an Ubuntu chroot**.
- Copy your local APT `sources.list` files into the chroot (we want to download the kernel from the Pop!_OS package repository — this is why you should run this script on a machine running Pop!_OS).
- **Install the build dependencies** into the chroot.
- **Download the source** for the latest available kernel.
- **Apply your `.patch` files** that you've put in the `./patches` folder.
- **Build the kernel** for you.

Installation
------------

    $ git clone https://github.com/gma/pop-os-kernel-builder.git
    $ sudo apt install debootstrap equivs

Usage Synopsis
--------------

    $ sudo ./build-kernel <label>

The `label` argument is required, and will be appended to the version number of the resulting Debian packages. As [explained on the Ubuntu wiki], we need to modify the package version so that we can tell that we're running our modified kernel (the label appears in the output of `uname -a`).

Once the `build-kernel` script completes (which will take a while) you'll have a lot of `.deb` files in the project directory. There's usually no need to install them all, but we'll get to that in a moment.

Pop!_OS has a meta package called `pop-desktop` that depends on `linux-system76` which, in turn, depends on other kernel meta packages. These meta packages specify which versions of Pop's default kernel packages should be installed. So if we were to install our kernel packages and then run `apt upgrade` then `apt` would immediately want to replace our kernel with the standard one, as the dependencies of `linux-system76` would no longer be met.

To get around this problem we can also replace these meta packages with some that depend on our custom packages, and then mark these packages as on "hold" with `apt-mark`. These meta packages are created by the `build-meta-packages` script, which is automatically run by `build-kernel`.

To install all this stuff, use the `install-packages` script. It will use the `.deb` files that have been created by `build-kernel` and `build-meta-packages`, and then mark them as "on hold".

Example
-------

If you were modifying the kernel with a patch known as "foo", you might do this:

    $ cd pop-os-kernel-builder
    $ cp /path/to/the-foo-mod.patch ./patches
    $ sudo ./build-kernel foo
    $ sudo ./install-packages foo

Note that `build-kernel` will attempt to apply your patch from within the Linux kernel source's folder, using the command `patch -p1 < filename.patch`.

The `-p1` will discard the first component of the directory names of the files referenced in the patch. If you find your patch doesn't apply, this might be why. You could edit the script to use `-p0`, but if that doesn't work it may be that your patch won't apply to the current kernel source without a bit of manual work.

If you need to make a patch yourself, you could do it from two folders containing the kernel source like this:

    $ cd /usr/src
    $ diff -aur linux-x.y.z-orig linux-x.y.z-patched > my.patch

Make sure that the patch is in a file whose name ends with `.patch`, or your patch will be ignored.

Why?
----

These days there isn't much call for those of us who run Linux on the desktop to patch the kernel in a distribution like Pop!_OS.

In my case, I need to apply a [few tweaks] in order to support a hardware modification to my ThinkPad X230 that converts it to support a 1920x1080 screen.

[chroot]: https://en.wikipedia.org/wiki/Chroot
[explained on the Ubuntu wiki]: https://wiki.ubuntu.com/Kernel/BuildYourOwnKernel
[few tweaks]: https://gist.github.com/gma/fcacfcf6ae7bb560f09bd525e760ec9f
