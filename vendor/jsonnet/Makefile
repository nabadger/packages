include ../../tasks/Makefile.package
include ../../tasks/Makefile.apk

ifeq ($(OS), darwin)
  OS=darwin
else
  OS=linux
endif


export VENDOR = google
export PACKAGE_NAME = jsonnet
export PACKAGE_REPO_NAME = jsonnet
export DOWNLOAD_URL = $(PACKAGE_REPO_URL)/releases/download/v$(PACKAGE_VERSION)/$(PACKAGE_NAME)-bin-v${PACKAGE_VERSION}-$(OS).tar.gz

## APK build specific requirements
export APK_BUILD_TEMPLATE = APKBUILD.github-binary

install:
	$(call download_tarball)

test:
	$(PACKAGE_EXE) --version
