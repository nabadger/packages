include ../../tasks/Makefile.package
include ../../tasks/Makefile.apk
include ../../tasks/Makefile.deb

# Package details

export VENDOR ?= Versent
export DOWNLOAD_URL ?= $(PACKAGE_REPO_URL)/releases/download/v$(PACKAGE_VERSION)/$(PACKAGE_NAME)_$(PACKAGE_VERSION)_$(OS)_$(ARCH).tar.gz
export APK_BUILD_TEMPLATE ?= APKBUILD.github-binary

install:
	$(call download_tarball)

test:
	$(PACKAGE_EXE) --version
