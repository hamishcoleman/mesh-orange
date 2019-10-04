Automated builder for various Raspberry Pi boards
=================================================

Please note!  This project will not be able to support Raspberry Pi Zero,
Compute Module 1, Raspberry Pi 1A or Raspberry Pi 1B boards without major
changes to the debian-minimal-builder submodule - this is due to hardware
incompatibilities and is unlikely to change.

Supported Boards
----------------

This works on the 2b, 3b and 3b+ - it should also work for the cm3 (but I have
not tested that) as they are all using ARMv7+ CPUs that are compatible
with the Debian "armhf" architecture definition.

The other Raspberry Pi systems (the older ones and the Zeros) all use
an ARMv6+VFP cpu, which is incompatible with the Debian "armhf" - do
not be fooled by the fact that Raspbian uses an architecture called
"armhf", they have simply (and confusingly) redefined their architecture
to match their needs.

Power Usage
-----------

During testing, I measured the power used for several different hardware
configurations.

| Setup                                                 |  Model   | Bootup | Idle   | Notes |
|-------------------------------------------------------|----------|-------:|-------:| ----- |
| Raspbian lite, Gig Ethernet plugged in                | 4B v1.1 4G | >900mA |  774mA |
| Raspbian lite, Gig Ethernet unplugged                 | 4B v1.1 4G |        |  670mA |
| Ramdisk, Gig Ethernet plugged in                      | 4B v1.1  | <900mA |  700mA |
| Ramdisk, Ethernet plugged in                          | 3B+ v1.3 |  903mA |  530mA | 1 |
| Ramdisk, Ethernet plugged in                          | 3B v1.2  |  705mA |  295mA |
| Ramdisk, Ethernet plugged in                          | 2B v1.1  | >370mA |  254mA |
| Ramdisk, Ethernet unplugged                           | 2B v1.1  |        |  215mA |

Note 1: When doing actual work, 3B+ was able to easily burst past the 903mA needed to bootup.

* Bootup is the current that must be available for a successful boot
  without crashing.  "Not crashing" is defined as can login and run status
  commands with no error or freeze for 5 minutes.  Note that this is a bare
  minimum value - add at least 10% for safe normal use.
* Idle is the approximate current that the board uses after completing booting.
* All tests were done with a power supply that never allows the device under
  test to draw more current than allowed.  This ensures that the correct 5v
  supply is always maintained, but is different to your usual Micro-USB plug
  pack.  Due to this difference, it is possible that a smaller capacity
  plugpack could be successful - but that the device would then be operating
  out of specified limits.

Raspbian vs Debian in the Raspberry Pi World
--------------------------------------------

The most annoying bit about this is that the world now has two
Debian-ish architectures named armhf that are only compatible in one
direction (Raspbian armhf binaries should work on Debian armhf systems -
just slightly slower) and no clear naming to tell them apart, thus
making multiarch impossible to use to fix this.

The upshot is that the build system could build images for the other
Raspberry Pi systems, but it would need the debian builder to build
one based on Raspbian.

