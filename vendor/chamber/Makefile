# Package details
export VENDOR ?= segmentio
export APK_BUILD_TEMPLATE ?= APKBUILD.github-binary
export DOWNLOAD_URL ?= $(PACKAGE_REPO_URL)/releases/download/v$(PACKAGE_VERSION)/$(PACKAGE_NAME)-v$(PACKAGE_VERSION)-$(OS)-$(ARCH)
export CHECK_COMMAND_ARGUMENTS ?= version

include ../../tasks/Makefile.package
include ../../tasks/Makefile.apk
include ../../tasks/Makefile.deb

install:
	$(call download_binary)

test:
	$(PACKAGE_EXE) version
