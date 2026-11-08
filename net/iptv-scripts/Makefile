# SPDX-License-Identifier: Apache-2.0

include $(TOPDIR)/rules.mk

PKG_NAME:=iptv-scripts
PKG_VERSION:=0.1.0
PKG_RELEASE:=1

PKG_MAINTAINER:=ming
PKG_LICENSE:=Apache-2.0

include $(INCLUDE_DIR)/package.mk

define Package/iptv-scripts
  SECTION:=net
  CATEGORY:=Network
  TITLE:=IPTV device setup scripts
  DEPENDS:=+kmod-iptv
  PKGARCH:=all
endef

define Package/iptv-scripts/description
 UCI configuration, boot script and apply helper that create the itv
 device from kmod-iptv. Configure addressing and bridging on itv with
 the standard Network pages.
endef

define Package/iptv-scripts/conffiles
/etc/config/iptv
endef

define Build/Compile
endef

define Package/iptv-scripts/install
	$(INSTALL_DIR) $(1)/etc/config
	$(INSTALL_CONF) ./files/etc/config/iptv $(1)/etc/config/iptv

	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) ./files/etc/init.d/iptv $(1)/etc/init.d/iptv

	$(INSTALL_DIR) $(1)/usr/libexec
	$(INSTALL_BIN) ./files/usr/libexec/iptv-apply $(1)/usr/libexec/iptv-apply

	$(INSTALL_DIR) $(1)/etc/uci-defaults
	$(INSTALL_BIN) ./files/etc/uci-defaults/30-iptv-stb-ignore \
		$(1)/etc/uci-defaults/30-iptv-stb-ignore
endef

$(eval $(call BuildPackage,iptv-scripts))
