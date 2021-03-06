# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#

include $(TOPDIR)/rules.mk

PKG_NAME:=node
PKG_CPE_ID:=cpe:/a:nodejs:node.js

ifeq ($(CONFIG_NODEJS_11),y)
PKG_VERSION:=v11.6.0
PKG_RELEASE:=1
PKG_HASH:=94f2be389c80ab939114f67c824db7ebd12df602358b7481c55431336bbff9b3
PATCH_DIR:=./patches/v11.x
else
ifeq ($(CONFIG_NODEJS_10),y)
PKG_VERSION:=v10.15.0
PKG_RELEASE:=1
PKG_HASH:=797ab34c74b83b21b7d6ea261b5ca235d34c61a7da5aebb32459a963097ede3e
PATCH_DIR:=./patches/v10.x
else
ifeq ($(CONFIG_NODEJS_6),y)
PKG_VERSION:=v6.16.0
PKG_RELEASE:=1
PKG_HASH:=0d0882a9da1ccc217518d3d1a60dd238da9f52bed0c7daac42b8dc3d83bd7546
PATCH_DIR:=./patches/v6.x
else
PKG_VERSION:=v8.15.0
PKG_RELEASE:=1
PKG_HASH:=968523333947cc3f769d73dedc6c9c60580826d8714bc0e62ca4589de6a7c633
PATCH_DIR:=./patches/v8.x
endif
endif
endif

PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.xz
PKG_SOURCE_URL:=https://nodejs.org/dist/${PKG_VERSION}

HOST_BUILD_DEPENDS:=python/host
PKG_BUILD_DEPENDS:=python/host
PKG_INSTALL:=1
PKG_USE_MIPS16:=0

HOST_BUILD_PARALLEL:=1
PKG_BUILD_PARALLEL:=1

PKG_MAINTAINER:=Hirokazu MORIKAWA <morikw2@gmail.com>
PKG_LICENSE:=MIT
PKG_LICENSE_FILES:=LICENSE

include $(INCLUDE_DIR)/host-build.mk
include $(INCLUDE_DIR)/package.mk

define Package/node
  SECTION:=lang
  CATEGORY:=Languages
  SUBMENU:=Node.js
  TITLE:=Node.js is a platform built on Chrome's JavaScript runtime
  URL:=https://nodejs.org/
  DEPENDS:=+libstdcpp +(!NODEJS_10&&!NODEJS_11):libopenssl +(!NODEJS_10&&!NODEJS_11):zlib \
           +USE_UCLIBC:libpthread +USE_UCLIBC:librt +NODEJS_ICU_SYSTEM:icu \
           +(NODEJS_10||NODEJS_11):libatomic
endef

define Package/node/description
  Node.js® is a JavaScript runtime built on Chrome's V8 JavaScript engine. Node.js uses
  an event-driven, non-blocking I/O model that makes it lightweight and efficient. Node.js'
   package ecosystem, npm, is the largest ecosystem of open source libraries in the world.
endef

define Package/node-npm
  SECTION:=lang
  CATEGORY:=Languages
  SUBMENU:=Node.js
  TITLE:=NPM stands for Node Package Manager
  URL:=https://www.npmjs.com/
  DEPENDS:=+node
endef

define Package/node-npm/description
 NPM is the package manager for Node.js
endef

#define Package/node-inspect
#  SECTION:=lang
#  CATEGORY:=Languages
#  SUBMENU:=Node.js
#  TITLE:=Node Inspect
#  URL:=https://www.npmjs.com/
#  DEPENDS:=+node
#endef

#define Package/node-inspect/description
# node-inspect re-implementing node debug against the new V8 debugger protocol.
#endef

define Package/node/config
	menu "Configuration"
		depends on PACKAGE_node

	choice
		prompt "Version Selection"
		default NODEJS_8
		help
		 Select node.js version.

		config NODEJS_6
			bool "6.x Maintenance LTS"

		config NODEJS_8
			bool "8.x Active LTS"

		config NODEJS_10
			bool "10.x Active LTS"

		config NODEJS_11
			bool "11.x Current Release"
	endchoice

	choice
		prompt "ICU Selection"
		default NODEJS_ICU_NONE
		help
		 Select i18n features

		config NODEJS_ICU_NONE
			bool "Disable"

		config NODEJS_ICU_SMALL
			bool "small-icu"

		config NODEJS_ICU_SYSTEM
			bool "system-icu"
	endchoice

	config NODEJS_DEBUG
		bool "Debug build"
		default n

	endmenu
endef

NODEJS_CPU:=$(subst powerpc,ppc,$(subst aarch64,arm64,$(subst x86_64,x64,$(subst i386,ia32,$(ARCH)))))

ifeq ($(CONFIG_NODEJS_10),y)
TARGET_LDFLAGS+=-latomic
endif

ifeq ($(CONFIG_NODEJS_11),y)
TARGET_LDFLAGS+=-latomic
endif

MAKE_VARS+= \
	DESTCPU=$(NODEJS_CPU)

ifeq ($(CONFIG_NODEJS_DEBUG),y)
	STRIP:="/bin/true"
endif

CONFIGURE_VARS:= \
	CC="$(TARGET_CC) $(TARGET_OPTIMIZATION)" \
	CXX="$(TARGET_CXX) $(TARGET_OPTIMIZATION)" \
	CC_host="$(HOSTCC)" \
	CXX_host="$(HOSTCXX)"

CONFIGURE_ARGS:= \
	--dest-cpu=$(NODEJS_CPU) \
	--dest-os=linux \
	--without-snapshot \
	$(if $(CONFIG_NODEJS_10),,$(if $(CONFIG_NODEJS_11),,--shared-zlib)) \
	$(if $(CONFIG_NODEJS_10),,$(if $(CONFIG_NODEJS_11),,--shared-openssl)) \
	$(if $(CONFIG_NODEJS_DEBUG),--debug) \
	$(if $(CONFIG_NODEJS_ICU_SMALL),, \
		--with-intl=$(if $(CONFIG_NODEJS_ICU_SYSTEM),system-icu,none)) \
	$(if $(findstring mips,$(NODEJS_CPU)), \
		$(if $(CONFIG_SOFT_FLOAT),--with-mips-float-abi=soft)) \
	$(if $(findstring +neon,$(CONFIG_CPU_TYPE)),--with-arm-fpu=neon) \
	$(if $(findstring +vfp",$(CONFIG_CPU_TYPE)),--with-arm-fpu=vfp) \
	$(if $(findstring +vfpv3",$(CONFIG_CPU_TYPE)),--with-arm-fpu=vfpv3-d16) \
	$(if $(findstring +vfpv4",$(CONFIG_CPU_TYPE)),--with-arm-fpu=vfpv3) \
	--prefix=/usr

HOST_CONFIGURE_VARS:=
HOST_CONFIGURE_ARGS:= \
	--dest-os=linux \
	--without-snapshot \
	--with-intl=none \
	--prefix=$(STAGING_DIR_HOSTPKG)

HOST_CONFIGURE_CMD:=python ./configure

define Build/Prepare
	$(if $(findstring arm,$(NODEJS_CPU)), $(if $(CONFIG_SOFT_FLOAT), \
		echo "You can't running Node.js on ARM CPU without hardware FPU."; \
		exit 1; \
	))
	$(Build/Prepare/Default)
endef

define Build/InstallDev
	$(INSTALL_DIR) $(1)/usr/include
	$(CP) $(PKG_INSTALL_DIR)/usr/include/* $(1)/usr/include/
endef

define Package/node/install
	$(INSTALL_DIR) $(1)/usr/bin
	$(INSTALL_BIN) $(PKG_INSTALL_DIR)/usr/bin/node $(1)/usr/bin/
ifeq ($(CONFIG_NODEJS_DEBUG),y)
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/node_g $(1)/usr/bin/
endif
endef

define Package/node-npm/install
	$(INSTALL_DIR) $(1)/usr/lib/node_modules
	$(CP) $(PKG_INSTALL_DIR)/usr/lib/node_modules/* $(1)/usr/lib/node_modules/
	$(INSTALL_DIR) $(1)/usr/bin
	$(LN) ../lib/node_modules/npm/bin/npm-cli.js $(1)/usr/bin/npm
ifneq ($(CONFIG_NODEJS_6),y)
	$(LN) ../lib/node_modules/npm/bin/npx-cli.js $(1)/usr/bin/npx
endif
endef

#define Package/node-inspect/install
#	$(INSTALL_DIR) $(1)/usr/lib/node_modules
#	$(CP) $(PKG_INSTALL_DIR)/usr/lib/node_modules/* $(1)/usr/lib/node_modules/
#	$(INSTALL_DIR) $(1)/usr/bin
#	$(LN) ../lib/node_modules/node-inspect/cli.js $(1)/usr/bin/node-inspect
#endef

$(eval $(call HostBuild))
$(eval $(call BuildPackage,node))
$(eval $(call BuildPackage,node-npm))
#$(eval $(call BuildPackage,node-inspect))
