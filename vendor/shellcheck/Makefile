include ../../tasks/Makefile.package
include ../../tasks/Makefile.apk
include ../../tasks/Makefile.deb

CURRENT_VERSION: QUERY=first(.[] | .name)
CURRENT_VERSION: API=tags

# Package details
export VENDOR ?= koalaman
export DOWNLOAD_URL ?= $(PACKAGE_REPO_URL)/releases/download/v$(PACKAGE_VERSION)/$(PACKAGE_NAME)-v${PACKAGE_VERSION}.linux.x86_64.tar.xz
export APK_BUILD_TEMPLATE ?= APKBUILD.github-binary

install:
	$(call download_tar_xz)

test:
	$(PACKAGE_EXE) --version

package/prepare::
	mv src/shellcheck-v$(PACKAGE_VERSION)/shellcheck src
