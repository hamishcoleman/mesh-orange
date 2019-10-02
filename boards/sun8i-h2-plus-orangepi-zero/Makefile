#
# Build a Orange Pi Zero install
#

DEBIAN_VER = buster
DEBIAN_ARCH = armhf

# which uboot and device tree is this being built for
ARMBIAN_UBOOT_PKG = linux-u-boot-orangepizero-default=5.90
ARMBIAN_KERNEL_PKG = linux-image-next-sunxi=5.90
UBOOT = linux-u-boot-orangepizero_5.90_armhf
BOARD = sun8i-h2-plus-orangepi-zero
# FIXME - it would be nice if the uboot name was related to the dtb name

MODULE_VER = 4.19.57-sunxi

SRC_SPL = $(ARMBIAN)/usr/lib/$(UBOOT)/u-boot-sunxi-with-spl.bin
SRC_DTBDIR = $(ARMBIAN)/usr/lib/linux-image-next-sunxi
SRC_FDT = $(SRC_DTBDIR)/$(BOARD).dtb
SRC_DTBO = $(SRC_DTBDIR)/overlay

INITRD_PARTS += $(BUILD)/modules.lzma
CLEAN_FILES += $(BUILD)/modules.lzma

# Empty dirs
MODULES += \
    lib \
    lib/modules \
    lib/modules/$(MODULE_VER) \
    lib/modules/$(MODULE_VER)/kernel \
    lib/modules/$(MODULE_VER)/kernel/drivers \

# Index
MODULES += \
    lib/modules/$(MODULE_VER)/modules.dep.bin \
    lib/modules/$(MODULE_VER)/modules.builtin.bin \

# Wifi
MODULES += \
    lib/modules/$(MODULE_VER)/kernel/drivers/mmc \
    lib/modules/$(MODULE_VER)/kernel/drivers/mmc/core \
    lib/modules/$(MODULE_VER)/kernel/drivers/mmc/core/pwrseq_simple.ko \
    lib/modules/$(MODULE_VER)/kernel/drivers/net \
    lib/modules/$(MODULE_VER)/kernel/drivers/net/wireless \
    lib/modules/$(MODULE_VER)/kernel/drivers/net/wireless/xradio \
    lib/modules/$(MODULE_VER)/kernel/drivers/net/wireless/xradio/xradio_wlan.ko \
    lib/modules/$(MODULE_VER)/kernel/net \
    lib/modules/$(MODULE_VER)/kernel/net/mac80211 \
    lib/modules/$(MODULE_VER)/kernel/net/mac80211/mac80211.ko \
    lib/modules/$(MODULE_VER)/kernel/net/wireless \
    lib/modules/$(MODULE_VER)/kernel/net/wireless/cfg80211.ko \

# Networking - Firewall and VPN
MODULES += \
    lib/modules/$(MODULE_VER)/kernel/drivers \
    lib/modules/$(MODULE_VER)/kernel/drivers/net \
    lib/modules/$(MODULE_VER)/kernel/drivers/net/tun.ko \
    lib/modules/$(MODULE_VER)/kernel/net/ipv4 \
    lib/modules/$(MODULE_VER)/kernel/net/ipv4/netfilter \
    lib/modules/$(MODULE_VER)/kernel/net/ipv4/netfilter/ip_tables.ko \
    lib/modules/$(MODULE_VER)/kernel/net/netfilter \
    lib/modules/$(MODULE_VER)/kernel/net/netfilter/x_tables.ko \
    lib/modules/$(MODULE_VER)/kernel/net/wireguard \
    lib/modules/$(MODULE_VER)/kernel/net/wireguard/wireguard.ko \

# USB
MODULES += \
    lib/modules/$(MODULE_VER)/kernel/drivers/usb \
    lib/modules/$(MODULE_VER)/kernel/drivers/usb/gadget \
    lib/modules/$(MODULE_VER)/kernel/drivers/usb/gadget/function \
    lib/modules/$(MODULE_VER)/kernel/drivers/usb/gadget/function/u_ether.ko \
    lib/modules/$(MODULE_VER)/kernel/drivers/usb/gadget/function/u_serial.ko \
    lib/modules/$(MODULE_VER)/kernel/drivers/usb/gadget/function/usb_f_acm.ko \
    lib/modules/$(MODULE_VER)/kernel/drivers/usb/gadget/function/usb_f_eem.ko \
    lib/modules/$(MODULE_VER)/kernel/drivers/usb/gadget/function/usb_f_mass_storage.ko \
    lib/modules/$(MODULE_VER)/kernel/drivers/usb/gadget/libcomposite.ko \

# cpufreq
MODULES += \
    lib/modules/$(MODULE_VER)/kernel/drivers/cpufreq \
    lib/modules/$(MODULE_VER)/kernel/drivers/cpufreq/cpufreq-dt.ko \
    lib/modules/$(MODULE_VER)/kernel/drivers/thermal \
    lib/modules/$(MODULE_VER)/kernel/drivers/thermal/thermal_sys.ko \

MODULES += \
    lib/modules/$(MODULE_VER)/kernel/drivers/iio \
    lib/modules/$(MODULE_VER)/kernel/drivers/iio/adc \
    lib/modules/$(MODULE_VER)/kernel/drivers/iio/adc/sun4i-gpadc-iio.ko \
    lib/modules/$(MODULE_VER)/kernel/drivers/iio/industrialio.ko \
    lib/modules/$(MODULE_VER)/kernel/drivers/thermal \
    lib/modules/$(MODULE_VER)/kernel/drivers/thermal/sun8i_ths.ko \

CLEAN_FILES += $(TAG) $(ARMBIAN) $(BOOT)

# Directories
ARMBIAN = $(BUILD)/armbian

BUILD_DEPENDS = \
    xz-utils \

TOP_DIR ?= $(abspath ../..)
export TOP_DIR

all: image

include $(TOP_DIR)/mk/common.mk
include $(TOP_DIR)/boards/common-partitions.mk
include $(TOP_DIR)/boards/common-allwinner.mk
include $(TOP_DIR)/boards/common-firmware-armbian.mk
include $(TOP_DIR)/boards/common-cjdns-hack.mk
include $(TOP_DIR)/boards/common-initrd.mk

test:
	echo this board must be tested by flashing the output
	false

# Everything below this line is HW specific Armbian u-Boot startup code

$(TAG)/armbian:
	$(TOP_DIR)/scripts/get_deb $(DEBIAN_ARCH) armbian $(ARMBIAN) $(ARMBIAN_UBOOT_PKG) $(ARMBIAN_KERNEL_PKG)
	$(call tag,armbian)

# Add the kernel specific binaries to this cpio file
$(BUILD)/modules.cpio: $(TAG)/armbian
	( \
            cd $(ARMBIAN); \
            for i in $(MODULES); do echo $$i; done |sort |uniq | cpio -H newc -R 0:0 -o \
	) > $@
CLEAN_FILES += $(BUILD)/modules.cpio

$(BOOT): $(TAG)/boot
$(TAG)/boot:
	mkdir -p $(BOOT)
	$(call tag,boot)

$(BOOT)/.next: $(TAG)/boot
	touch $@

# TODO
# - rewrite or cleanup these two files for our specific use cases

$(BOOT)/boot.scr: $(TAG)/boot
$(BOOT)/boot.scr: boot.scr
	cp $< $@
CLEAN_FILES += boot.scr

$(BOOT)/armbianEnv.txt: $(TAG)/boot
$(BOOT)/armbianEnv.txt: armbian/lib/config/bootenv/sunxi-default.txt
	cp $< $@

$(BOOT)/zImage: $(TAG)/boot $(TAG)/armbian
	cp $(ARMBIAN)/boot/vmlinuz-* $(BOOT)/zImage
	cp $(ARMBIAN)/boot/config-* $(BOOT)

dtb: $(TAG)/dtb
$(TAG)/dtb: $(TAG)/boot
	mkdir -p $(BOOT)/dtb
	$(call tag,dtb)

$(SRC_FDT): $(TAG)/armbian

$(BOOT)/dtb/$(BOARD).dtb: $(TAG)/dtb
$(BOOT)/dtb/$(BOARD).dtb: $(SRC_FDT)
	cp $< $@

dtbo: $(TAG)/dtbo
$(TAG)/dtbo: $(TAG)/dtb
	mkdir -p $(BOOT)/dtb/overlay
	$(call tag,dtbo)

$(BOOT)/dtb/overlay/%.dtbo: $(TAG)/armbian $(TAG)/dtbo
	touch $@ # FIXME
	#cp $< $@

$(BOOT)/uInitrd: $(TAG)/boot
$(BOOT)/uInitrd: $(BUILD)/combined.uInitrd
	cp $< $@
CLEAN_FILES += $(BUILD)/combined.uInitrd

BOOT_FILES = \
    $(BOOT)/boot.scr $(BOOT)/armbianEnv.txt \
    $(BOOT)/.next \
    $(BOOT)/zImage \
    $(BOOT)/uInitrd \

BOOT_DTB_FILES = \
    $(BOOT)/dtb/$(BOARD).dtb \

BOOT_DTBO_FILES = \
    $(BOOT)/dtb/overlay/sun8i-h3-usbhost0.dtbo \
    $(BOOT)/dtb/overlay/sun8i-h3-usbhost2.dtbo \
    $(BOOT)/dtb/overlay/sun8i-h3-usbhost3.dtbo \

boot: $(BOOT_FILES) $(BOOT_DTB_FILES) $(BOOT_DTBO_FILES)

# Everything below this line is packing the built boot dir into a disk image

$(SRC_SPL): $(TAG)/armbian

$(DISK_IMAGE): $(SRC_SPL) $(BUILD)/mtoolsrc $(BOOT_FILES) $(BOOT_DTB_FILES) $(BOOT_DTBO_FILES)
	$(call image_normal)
	$(call allwinner_normal,$@.tmp,$(SRC_SPL),$(BUILD)/mtoolsrc,z:,$(BOOT_FILES),$(BOOT_DTB_FILES))
	$(call uboot_copy_overlayfiles,$(BUILD)/mtoolsrc,z:,$(BOOT_DTBO_FILES))
	mv $@.tmp $@
