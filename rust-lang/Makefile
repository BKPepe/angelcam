include $(TOPDIR)/rules.mk

PKG_NAME := rust-lang
PKG_LICENSE := Apache-2.0/MIT
PKG_VERSION := 1.60.0
PKG_RELEASE := 1

PKG_SOURCE_PROTO:=git
PKG_SOURCE_URL:=https://github.com/rust-lang/rustup.git
PKG_SOURCE_VERSION:=1.24.3
PKG_MIRROR_HASH:=ba8ebb610aaf76127a475ae899dee44e57e81341965f37b3c534196c933fc2df

HOST_BUILD_DIR := $(BUILD_DIR_HOST)/$(PKG_NAME)-$(PKG_VERSION)

ADDITIONAL_TARGETS :=

ifeq ($(CONFIG_TARGET_PROFILE),"DEVICE_cznic_turris-omnia")
  ADDITIONAL_TARGETS += armv7-unknown-linux-musleabihf
else
  ADDITIONAL_TARGETS += $(shell echo $(CONFIG_ARCH)-unknown-linux-$(CONFIG_TARGET_SUFFIX))
endif

include $(INCLUDE_DIR)/host-build.mk
include $(INCLUDE_DIR)/package.mk

Build/Compile:=:

define Host/Prepare
	$(call Host/Prepare/Default)
	chmod 755 $(HOST_BUILD_DIR)/rustup-init.sh
endef

define Host/Compile
	test -e $$$$HOME/.cargo/bin/rustup || $(HOST_BUILD_DIR)/rustup-init.sh -y --profile minimal
	$$$$HOME/.cargo/bin/rustup toolchain install $(PKG_VERSION)
	$$$$HOME/.cargo/bin/rustup target add --toolchain $(PKG_VERSION) $(ADDITIONAL_TARGETS)
endef

define Host/Install
	test -e $(STAGING_DIR_HOST)/bin/cargo || ln -s $$$$HOME/.cargo/bin/cargo $(STAGING_DIR_HOST)/bin/
	test -e $(STAGING_DIR_HOST)/bin/rustc || ln -s $$$$HOME/.cargo/bin/rustc $(STAGING_DIR_HOST)/bin/
	test -e $(STAGING_DIR_HOST)/bin/rustup || ln -s $$$$HOME/.cargo/bin/rustup $(STAGING_DIR_HOST)/bin/
endef

$(eval $(call HostBuild))
$(eval $(call Build/DefaultTargets))
