include $(TOPDIR)/rules.mk

PKG_NAME := angelcam-connector
PKG_LICENSE := Apache-2.0
PKG_VERSION := 0.10.1
PKG_RELEASE := 1

PKG_BUILD_DEPENDS := rust-lang/host

PKG_SOURCE_PROTO:=git
PKG_SOURCE_URL:=https://github.com/angelcam/arrow-client.git
PKG_SOURCE_VERSION:=v$(PKG_VERSION)
PKG_MIRROR_HASH:=8651cb53d51642cac468948a0d40d2b04cec28e9ff046df254641b5a46cc8d87

include $(INCLUDE_DIR)/package.mk

ifeq ($(CONFIG_TARGET_PROFILE),"DEVICE_cznic_turris-omnia")
  RUST_TARGET := armv7-unknown-linux-musleabihf
else
  RUST_TARGET := $(shell echo $(CONFIG_ARCH)-unknown-linux-$(CONFIG_TARGET_SUFFIX))
endif

define Package/angelcam-connector
  TITLE := Angelcam Connector
  CATEGORY := Network
  SECTION := net
  DEPENDS := +libpcap +libopenssl
endef

define Package/angelcam-connector/description
  Angelcam Connector
endef

define CARGO_CONFIG
[target.$(RUST_TARGET)]
ar = "$(TARGET_AR)"
linker = "$(TARGET_CC)"
rustflags = [
  "-C", "link-args=$(TARGET_LDFLAGS)",
  "-C", "target-feature=-crt-static",
]
endef

export CARGO_CONFIG

define Build/Prepare
	$(call Build/Prepare/Default)
	mkdir $(PKG_BUILD_DIR)/.cargo
	echo "$$$$CARGO_CONFIG" > $(PKG_BUILD_DIR)/.cargo/config
endef

define Build/Compile
	(\
		cd $(PKG_BUILD_DIR) && \
		export OPENSSL_LIB_DIR=$(STAGING_DIR)/usr/lib && \
		export OPENSSL_INCLUDE_DIR=$(STAGING_DIR)/usr/include && \
		export CC_$(subst -,_,$(RUST_TARGET))=$(TARGET_CC) && \
		export CFLAGS_$(subst -,_,$(RUST_TARGET))="$(TARGET_CFLAGS)" && \
		cargo +1.60.0 build \
			--features "discovery threads" \
			--release \
			--target $(RUST_TARGET) \
	)
endef

define Package/angelcam-connector/postinst
#!/bin/sh
/etc/init.d/angelcam-connector enable
/etc/init.d/angelcam-connector start
endef

define Package/angelcam-connector/prerm
#!/bin/sh
/etc/init.d/angelcam-connector stop
/etc/init.d/angelcam-connector disable
endef

define Package/angelcam-connector/install
	$(INSTALL_DIR) $(1)/usr/bin
	$(INSTALL_DIR) $(1)/etc/angelcam-connector
	$(INSTALL_DIR) $(1)/usr/local/angelcam-connector
	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/target/$(RUST_TARGET)/release/arrow-client \
		$(1)/usr/bin/angelcam-connector
	$(INSTALL_BIN) ./files/init-script $(1)/etc/init.d/angelcam-connector
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/ca.pem $(1)/usr/local/angelcam-connector/
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/rtsp-paths $(1)/etc/angelcam-connector/
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/mjpeg-paths $(1)/etc/angelcam-connector/
	touch $(1)/etc/angelcam-connector/config.json
	touch $(1)/etc/angelcam-connector/config-skel.json
	chmod 600 $(1)/etc/angelcam-connector/config.json
	chmod 600 $(1)/etc/angelcam-connector/config-skel.json
endef

$(eval $(call BuildPackage,angelcam-connector))
