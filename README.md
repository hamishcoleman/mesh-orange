A small ramdisk system running modern Debian
============================================

This project will create system images for installing a Debian-based
router.

The initial hardware target is to run on an Orange Pi Zero Single Board
Computer and use the TOP-GS07 RT5572 WiFi adapter for mesh traffic.

In the future, it will create a fully working mesh node, with cjdns
and IPFS software installed.


Documentation Index
-------------------

There documentory README files in most directories with explanations as
to what that directory is used for.  These files are located where they
are so as to stay close to the code and config that they are documenting.
Be sure to read this main README and all the other ones to get a full view
of the project.


TOP-GS07 RT5572 WiFi adapter Note
---------------------------------

During the testing of this adaptor, it was found to get very hot.  If
insufficient cooling is provided, this adaptor could overheat and fail
to work properly.  In a subtropical summer environment, it was found
that this would occur within a couple of minutes of use.  Removing the
plastic cover allows some cooling and was enough to complete at least
half an hour of successful light testing.

Building the image
------------------

Multiple board targets are supported, but they all use the same basic
process.  The following commands will build for the Orange Pi Zero:

    make build-depends

    make -C boards/sun8i-h2-plus-orangepi-zero image

First, any packages required to complete the build are installed -
this needs to be done for both the Debian environment and the specific
board environment, which is why there are two lines.

The last command builds the image.  Once the disk image is completed,
it will placed in the `output` dir.

Starting a clean build
----------------------

During development, not all the makefile dependancies are always working.
Whilst this should always be considered a bug, it is important to know
how to do a clean build from scratch.

One simple answer is to completely delete your repository and do a new
clone from upstream, however that can be a little excessive.  The following
commands will remove all the build artifacts:

    make clean reallyclean

The "clean" target removes some files one-by-one from the build dir,
wheras the "reallyclean" target just nukes each build dir - so this is
kind of a belt-and-suspender approach.


Using the image
---------------

Once the image is built, and you have the disk image file from above,
write to the raw sdcard with a command line tool like `dd` or a
graphical tool like [Etcher](https://etcher.io).

For `dd`:

    lsblk -d -o NAME,SIZE,LABEL
    echo "Verify and run: sudo dd if=output/$IMAGE of=$DISK"

`Warning`: Don't overwrite the wrong disk!

Booting and using the system
----------------------------

A wireless access point will be automatically started on all detected
wifi adaptors - including those hot-plugged after bootup.  Any internet
connection plugged into the ethernet port will be shared out over the
access point.

`Note`: There is currently a bug with stopping/restarting the hostapd,
so if you unplug an adaptor, it will not automatically work again until
a reboot is done.

During testing the following default settings are used:

* wifi ssid: **test2**
* wifi passphrase: **bbbbbbbb**
* user: **root**
* pass: **root**

An ssh server is started on bootup, so the simplest way to login is to
connect to the wifi and use the root password.

During development and debugging, it is very helpful to use a serial
console to see the boot messages and login.  It may also be useful
to read the section below on running this image in an emulator.

`Note`: uncompressing the (currently approximately 120Meg uncompressed) initrd
will take a noticeable amount of time.  The network will not be setup until
that has been done, so nothing will happen for a while.  For some reason, this
also applies to the kernel messages.

Tests on one orange pi zero show that the time from power on until a login
prompt on the serial console is about 1 minute.


Applying persistent configurations
----------------------------------

Custom configurations are saved as tar.gz archives in conf.d at the root of the
fat16 file system. After the start of userspace and before systemd is launched,
content in each archive is extracted in alphabetical order to /etc of the ramdisk,
so files in an archive towards the end of the list overwrite those from an earlier
archive in case of a filename conflict, and systemd will only see the final
(highest priority) configurations. For example:

    /conf.d/00-tomesh-base.tar.gz
            10-tomesh-wlan-mesh-top-gs07-rt5572.tar.gz
            11-tomesh-wlan-hostap-tplink-tl-wn722n.tar.gz
            50-node-config-save.tar.gz
            90-user-custom-configs.tar.gz

This allows local mesh communities to customize nodes by distributing archives
of systemd.network configuration files, or any files that are to be read from
/etc. The user simply mounts the fat filesystem like a USB key and drops tar.gz
files into conf.d and backs them up across software updates.

If the node needs to save persistent configurations while running, such as
remembering a MAC address, it can save the current /etc directory into a
50-node-config-save.tar.gz archive by running `config_save`. The user may
also manually add configurations with higher priority archive names.

Test the Debian image using Qemu
--------------------------------

It is possible to boot the Debian system up in an emulator.  This is
useful during development to speed up testing, but is also useful for
exploring the system and learning its features before committing to the
purchase of any hardware.

### Testing with the armhf architecture

The armhf architecture is an environment that is closer to the real-life
Single Board Computers that are expected to be used - and it uses exactly
the same binaries that would be used on those.  However, it is usually
a slower emulator.

    make build-depends

    make -C boards/qemu_armhf test

Once the build has completed, it will boot up inside the emulator.  The
console of the emulator is connected to your terminal window.

To exit the emulator, use `Ctrl-A` then `x`.

### Testing with the i386 architecture

While this is not the expected architecture for most physical hardware,
it is a much faster emulation.  Additionally, all the debian packages,
configuration and customisation is the same as the armhf architecture.

    make build-depends

    make -C boards/qemu_i386 test

To exit the emulator, use `Ctrl-A` then `x`.

Debian ramdisk builder
----------------------

The debian directory contains the Debian builder - see the README in
that dir for more details.
