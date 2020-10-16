include ../../tasks/Makefile.package
include ../../tasks/Makefile.apk

ifeq ($(OS), darwin)
  OS=darwin
else
  OS=linux
endif

export VENDOR = jsonnet-bundler
export PACKAGE_NAME = jb
export PACKAGE_REPO_NAME = jsonnet-bundler
export DOWNLOAD_URL = $(PACKAGE_REPO_URL)/releases/download/v$(PACKAGE_VERSION)/$(PACKAGE_NAME)-$(OS)-$(ARCH)

## APK build specific requirements
export APK_BUILD_TEMPLATE = APKBUILD.github-binary

install:
	$(call download_binary)

test:
	$(PACKAGE_EXE) --version
