# Package details
export VENDOR ?= bcicen
export APK_BUILD_TEMPLATE ?= APKBUILD.github-binary
export DOWNLOAD_URL ?= $(PACKAGE_REPO_URL)/releases/download/v$(PACKAGE_VERSION)/$(PACKAGE_NAME)-$(PACKAGE_VERSION)-$(OS)-$(ARCH)
export APKBUILD_DEPENDS += libc6-compat

include ../../tasks/Makefile.package
include ../../tasks/Makefile.apk
include ../../tasks/Makefile.deb

install:
	$(call download_binary)

test:
	$(PACKAGE_EXE) -v
