# Maintainer: local

pkgname=quickshell-velunara-git
pkgver=0.2.1.r816.g9b52f7b
pkgrel=1
pkgdesc='Flexible toolkit for making desktop shells with QtQuick'
arch=(x86_64 aarch64)
url='https://github.com/Velunara/quickshell'
license=(LGPL-3.0-only)
depends=(cpptrace
         gcc-libs
         glib2
         glibc
         hicolor-icon-theme
         jemalloc
         libdrm
         libglvnd
         libpipewire
         libxcb
         mesa
         pam
         polkit
         qt6-base
         qt6-declarative
         qt6-svg
         qt6-wayland
         wayland)
makedepends=(cli11
             cmake
             cpptrace
             ninja
             pkgconf
             qt6-shadertools
             spirv-tools
             vulkan-headers
             wayland-protocols)
optdepends=('bluez: Bluetooth service integration'
            'networkmanager: Network service integration'
            'upower: UPower service integration')
provides=(quickshell)
conflicts=(quickshell)
options=(!debug)
source=()
sha256sums=()

# Set QUICKSHELL_USE_PREBUILT=0 to force a fresh CMake build.
# Set QUICKSHELL_CRASH_HANDLER=ON to build with cpptrace crash handling enabled.
_use_prebuilt="${QUICKSHELL_USE_PREBUILT:-auto}"
_crash_handler="${QUICKSHELL_CRASH_HANDLER:-OFF}"
_cmake_build_dir="$startdir/pkgbuild-build"

pkgver() {
	cd "$startdir"

	local base
	base="$(awk -F'"' '/project\(quickshell VERSION/ { print $2; exit }' CMakeLists.txt)"

	if git rev-parse --is-inside-work-tree >/dev/null 2>&1; then
		printf '%s.r%s.g%s' \
			"$base" \
			"$(git rev-list --count HEAD)" \
			"$(git rev-parse --short HEAD)"
	else
		printf '%s' "$base"
	fi
}

build() {
	if [[ "$_use_prebuilt" == auto && -x "$startdir/build/src/quickshell" ]]; then
		_use_prebuilt=1
	fi

	if [[ "$_use_prebuilt" == 1 ]]; then
		if [[ ! -x "$startdir/build/src/quickshell" ]]; then
			error 'QUICKSHELL_USE_PREBUILT=1 was set, but build/src/quickshell is not executable'
			return 1
		fi
		return
	fi

	local cmake_options=(
		-D CMAKE_BUILD_TYPE=RelWithDebInfo
		-D DISTRIBUTOR='Velunara PKGBUILD'
		-D CRASH_HANDLER="$_crash_handler"
		-D CMAKE_INSTALL_PREFIX=/usr
		-D INSTALL_QML_PREFIX=lib/qt6/qml
	)

	cmake -G Ninja -S "$startdir" -B "$_cmake_build_dir" -W no-dev "${cmake_options[@]}"
	cmake --build "$_cmake_build_dir"
}

package() {
	cd "$startdir"

	if [[ "$_use_prebuilt" == auto && -x "$startdir/build/src/quickshell" ]]; then
		_use_prebuilt=1
	fi

	if [[ "$_use_prebuilt" == 1 ]]; then
		install -Dm0755 build/src/quickshell "$pkgdir/usr/bin/quickshell"
		install -d "$pkgdir/usr/bin"
		ln -s quickshell "$pkgdir/usr/bin/qs"
		install -Dm0644 assets/org.quickshell.desktop "$pkgdir/usr/share/applications/org.quickshell.desktop"
		install -Dm0644 assets/quickshell.svg "$pkgdir/usr/share/icons/hicolor/scalable/apps/org.quickshell.svg"
	else
		DESTDIR="$pkgdir" cmake --install "$_cmake_build_dir"
	fi

	install -Dm0644 LICENSE "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}
