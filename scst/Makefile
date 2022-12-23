#
# Copyright (C) 2007-2018 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#
# $Id$

include $(TOPDIR)/rules.mk
include $(INCLUDE_DIR)/kernel.mk

PKG_NAME:=scst
#PKG_VERSION:=trunk
PKG_VERSION:=3.3.x
PKG_RELEASE:=1

PKG_SOURCE_PROTO:=svn
PKG_SOURCE_VERSION:=HEAD
ifeq ($(PKG_VERSION),trunk)
PKG_SOURCE_URL:=https://svn.code.sf.net/p/scst/svn/trunk
else
PKG_SOURCE_URL:=https://svn.code.sf.net/p/scst/svn/branches/$(PKG_VERSION)
endif
PKG_SOURCE_SUBDIR:=$(PKG_NAME)-$(PKG_VERSION)
PKG_SOURCE:=$(PKG_SOURCE_SUBDIR).tar.bz2
PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_SOURCE_SUBDIR)

#In kernel 3.18/4.4+ already enabled Direct-IO and no need special
# parameters. If you use old kernel, you must add "+@KERNEL_DIRECT_IO"
# parameter to config_deps
#PKG_CONFIG_DEPENDS:=+@KERNEL_DIRECT_IO
PKG_BUILD_DEPENDS:=linux

include $(INCLUDE_DIR)/package.mk

OTHER_MENU:=Other modules

define Package/scst
  SECTION:=net
  CATEGORY:=Network
  DEPENDS:=+kmod-scst +kmod-iscsi-scst +kmod-scst-vdisk
  TITLE:=SCST open source iSCSI target
  URL:=http://scst.sourceforge.net/
endef

define Package/scst/description
 SCST is designed to provide unified, consistent interface between SCSI
 target drivers and Linux kernel and simplify target drivers development
 as much as possible.
endef

define KernelPackage/scst
  SUBMENU:=$(OTHER_MENU)
  TITLE:=SCST kernel module
  FILES:=$(PKG_BUILD_DIR)/scst/src/scst.ko
  #Use kconfig parameter "CONFIG_CRC_T10DIF=y" to built-in the kernel
  # dependency module or otherwise add "+kmod-lib-crc-t10dif" parameter
  # to depends
  KCONFIG:=CONFIG_CRC_T10DIF=y
  DEPENDS:=+kmod-scsi-core
endef

define KernelPackage/scst/description
 SCST module itself
endef

define KernelPackage/iscsi-scst
  SUBMENU:=$(OTHER_MENU)
  TITLE:=SCST iscsi support
  FILES:=$(PKG_BUILD_DIR)/iscsi-scst/kernel/iscsi-scst.ko
  DEPENDS:=+kmod-scst
endef

define KernelPackage/iscsi-scst/description
 iSCSI-SCST module itself
endef

define KernelPackage/scst-vdisk
  SUBMENU:=$(OTHER_MENU)
  TITLE:=SCST vdisk support
  FILES:=$(PKG_BUILD_DIR)/scst/src/dev_handlers/scst_vdisk.ko
  #Use kconfig parameter "CONFIG_LIBCRC32C=y" to built-in the kernel
  # dependency module or otherwise add "+kmod-lib-crc32c" parameter
  # to depends
  KCONFIG:=CONFIG_LIBCRC32C=y
  DEPENDS:=+kmod-scst
endef

define KernelPackage/scst-vdisk/description
 Device handler for virtual disks module (file, device or ISO CD image).
endef

#Added 'no-incompatible-pointer-types' exception because on
# x86 compiler we have error of that type.
NO_SIGN_COMPARE:='s!-Wno-sign-compare)! -Wno-sign-compare -Wno-incompatible-pointer-types)!g'
#NO_SIGN_COMPARE:='s!-Wno-missing-field-initializers)!-Wno-missing-field-initializers -Wno-sign-compare -Wno-incompatible-pointer-types)!g'
NO_DEBUG:='s!EXTRA_CFLAGS += -DCONFIG_SCST_DEBUG -g -fno-inline -fno-inline-functions!\#EXTRA_CFLAGS += -DCONFIG_SCST_DEBUG -g -fno-inline -fno-inline-functions!g'
NO_EXTRA_CHECKS:='s!EXTRA_CFLAGS += -DCONFIG_SCST_EXTRACHECKS!\#EXTRA_CFLAGS += -DCONFIG_SCST_EXTRACHECKS!g'
NO_DEBUG_O:='s!scst-y        += scst_debug.o!\#scst-y        += scst_debug.o!g'

define Build/Configure
        $(call Build/Configure/Default)
	$(SED) $(NO_SIGN_COMPARE) $(PKG_BUILD_DIR)/scst/src/Makefile
	$(SED) $(NO_DEBUG_O)      $(PKG_BUILD_DIR)/scst/src/Makefile
	$(SED) $(NO_EXTRA_CHECKS) $(PKG_BUILD_DIR)/scst/src/Makefile
	$(SED) $(NO_DEBUG)        $(PKG_BUILD_DIR)/scst/src/Makefile

	$(SED) $(NO_SIGN_COMPARE) $(PKG_BUILD_DIR)/scst/src/dev_handlers/Makefile
	$(SED) $(NO_EXTRA_CHECKS) $(PKG_BUILD_DIR)/scst/src/dev_handlers/Makefile
	$(SED) $(NO_DEBUG)        $(PKG_BUILD_DIR)/scst/src/dev_handlers/Makefile

	$(SED) $(NO_SIGN_COMPARE) $(PKG_BUILD_DIR)/iscsi-scst/kernel/Makefile
	$(SED) $(NO_EXTRA_CHECKS) $(PKG_BUILD_DIR)/iscsi-scst/kernel/Makefile
	$(SED) $(NO_DEBUG)        $(PKG_BUILD_DIR)/iscsi-scst/kernel/Makefile
endef

MAKE_FLAGS += \
	KVER=$(LINUX_VERSION) \
	KDIR=$(LINUX_DIR) \
	SRCARCH="$(SRCARCH)"

SRCARCH:=$(shell echo $(ARCH) | sed -e s'/-.*//' \
        -e 's/i.86/x86/' \
        -e 's/mips.*/mips/' \
        -e 's/mipsel.*/mips/' \
)

define Build/Compile
	$(call Build/Compile/Default,scst iscsi)
endef

define Package/scst/install
	$(INSTALL_DIR) $(1)/usr/sbin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/iscsi-scst/usr/iscsi-scstd $(1)/usr/sbin/
	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) ./files/$(PKG_NAME).init $(1)/etc/init.d/$(PKG_NAME)
endef

define Package/scst/postinst
#!/bin/sh
grep -q 'scst' $${IPKG_INSTROOT}/etc/config/ucitrack 2>/dev/null
[ $$? -ne 0 ] && {
	echo >>$${IPKG_INSTROOT}/etc/config/ucitrack ""
	echo >>$${IPKG_INSTROOT}/etc/config/ucitrack "config scst"
	echo >>$${IPKG_INSTROOT}/etc/config/ucitrack "       option init 'scst'"
}
exit 0
endef

$(eval $(call BuildPackage,scst))
$(eval $(call KernelPackage,scst))
$(eval $(call KernelPackage,scst-vdisk))
$(eval $(call KernelPackage,iscsi-scst))
