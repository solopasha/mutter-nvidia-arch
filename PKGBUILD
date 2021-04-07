# Maintainer: solopasha at daron439 dot com

pkgname=mutter-nvidia-arch
_pkgname=mutter
pkgver=40.0
pkgrel=1
pkgdesc="A window manager for GNOME."
url="https://gitlab.gnome.org/GNOME/mutter"
arch=(x86_64)
license=(GPL)
depends=(dconf gobject-introspection-runtime gsettings-desktop-schemas
         libcanberra startup-notification zenity libsm gnome-desktop upower
         libxkbcommon-x11 gnome-settings-daemon libgudev libinput pipewire
         xorg-xwayland graphene libxkbfile)
makedepends=(gobject-introspection git egl-wayland meson xorg-server sysprof)
provides=(libmutter-8.so mutter)
conflicts=(mutter)
groups=(gnome)
_commit=21a09fb7928c17519d67ffd8c1ae80071f92fdbf
source=("git+https://gitlab.gnome.org/GNOME/mutter.git#commit=$_commit"
        "night-light.patch")
sha256sums=('SKIP'
            'SKIP')

pkgver() {
  cd $_pkgname
  git describe --tags | sed 's/-/+/g'
}

prepare(){
  cd $_pkgname
  patch -Np1 -i ../night-light.patch
}
build() {
  arch-meson $_pkgname build \
    -D egl_device=true \
    -D wayland_eglstream=true \
    -D installed_tests=false \
    -D profiler=false
  meson compile -C build
}

package() {
  DESTDIR="$pkgdir" meson install -C build
}
