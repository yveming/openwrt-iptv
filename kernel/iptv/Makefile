# SPDX-License-Identifier: GPL-2.0-only

include $(TOPDIR)/rules.mk
include $(INCLUDE_DIR)/kernel.mk

PKG_NAME:=iptv
PKG_RELEASE:=2
PKG_LICENSE:=GPL-2.0-only AND Apache-2.0
PKG_MAINTAINER:=ming

include $(INCLUDE_DIR)/package.mk

define KernelPackage/iptv
  SUBMENU:=Network Devices
  TITLE:=IPTV interface support
  FILES:=$(PKG_BUILD_DIR)/iptv.ko
  AUTOLOAD:=$(call AutoLoad,46,iptv)
  KCONFIG:=
endef

define KernelPackage/iptv/description
  A single untagged Ethernet device carrying one unicast VLAN and one
  multicast VLAN over the uplink. The package also installs the UCI
  configuration, boot script and apply logic that create the device;
  neither part is useful without the other.
endef

define KernelPackage/iptv/conffiles
/etc/config/iptv
endef

define KernelPackage/iptv/install
	$(INSTALL_DIR) $(1)/etc/config
	$(INSTALL_CONF) $(CURDIR)/files/etc/config/iptv $(1)/etc/config/iptv

	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) $(CURDIR)/files/etc/init.d/iptv $(1)/etc/init.d/iptv

	$(INSTALL_DIR) $(1)/etc/uci-defaults
	$(INSTALL_BIN) $(CURDIR)/files/etc/uci-defaults/30-iptv-stb-ignore \
		$(1)/etc/uci-defaults/30-iptv-stb-ignore
endef

define Build/Prepare
	mkdir -p $(PKG_BUILD_DIR)
	$(CP) $(CURDIR)/src/* $(PKG_BUILD_DIR)/
endef

define Build/Compile
	$(KERNEL_MAKE) M="$(PKG_BUILD_DIR)" modules
endef

$(eval $(call KernelPackage,iptv))
