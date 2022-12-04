# Maintainer: Jan Alexander Steffens (heftig) <heftig@archlinux.org>
# Contributor: Ionut Biru <ibiru@archlinux.org>
# Contributor: Michael Kanis <mkanis_at_gmx_dot_de>


pkgbase=mutter
pkgname=(mutter mutter-docs)
pkgver=43.1+r24+g030e9b8b2
pkgrel=1
pkgdesc="A window manager for GNOME"
url="https://gitlab.gnome.org/GNOME/mutter"
arch=(x86_64)
license=(GPL)
depends=(dconf gobject-introspection-runtime gsettings-desktop-schemas
         libcanberra startup-notification libsm gnome-desktop libxkbcommon-x11
         gnome-settings-daemon libgudev libinput pipewire xorg-xwayland graphene
         libxkbfile libsysprof-capture lcms2 colord)
makedepends=(gobject-introspection git egl-wayland meson xorg-server
             wayland-protocols sysprof gi-docgen xorg-server-xvfb)
checkdepends=(wireplumber python-dbusmock zenity)
_commit=030e9b8b24cb4335455f4e51a39a2c854c5717b8
source=("git+https://gitlab.gnome.org/GNOME/mutter.git#commit=$_commit"
        "https://gitlab.gnome.org/GNOME/mutter/-/merge_requests/1441.patch"
        "https://gitlab.gnome.org/GNOME/mutter/-/merge_requests/1880.patch"
        "https://gitlab.gnome.org/GNOME/mutter/-/merge_requests/2671.patch"
        "https://gitlab.gnome.org/GNOME/mutter/-/merge_requests/2702.patch")
sha256sums=('SKIP'
            'f534708708ac8ce4adf021d50c70ab5f256f3d78df53194ccda9cecdee44e80b'
            '65981409a5fc5ebfa95c7178f588cb2564f405cf55cbc7a315ecd4ac6c892b1c'
            'eb9e2952046cce99a1a3d8010aa3175b011cafb37f0258ebe91f0a67bbf31729'
            '452f3b66a5853eda67c72cb1d6e00c256ddd159cf1e906e0374a5ec4e9861fe6')

pkgver() {
  cd mutter
  git describe --tags | sed 's/[^-]*-g/r&/;s/-/+/g'
}

prepare() {
  cd mutter
  local src
  for src in "${source[@]}"; do
    src="${src%%::*}"
    src="${src##*/}"
    [[ $src = *.patch ]] || continue
    echo "Applying patch $src..."
    patch -Np1 < "../$src"
  done
  #git revert -n a81d4aed7aa681672e181b6f6999057d1700df62 # fix stuttering when changing layout
}

build() {
  CFLAGS="${CFLAGS/-O2/-O3} -fno-semantic-interposition"
  LDFLAGS+=" -Wl,-Bsymbolic-functions"

  arch-meson mutter build \
    -D egl_device=true \
    -D wayland_eglstream=true \
    -D docs=true \
    -D installed_tests=false
  meson compile -C build
}

_check() (
  export XDG_RUNTIME_DIR="$PWD/rdir" GSETTINGS_SCHEMA_DIR="$PWD/build/data"
  mkdir -p -m 700 "$XDG_RUNTIME_DIR"
  glib-compile-schemas "$GSETTINGS_SCHEMA_DIR"

  pipewire &
  _p1=$!

  wireplumber &
  _p2=$!

  trap "kill $_p1 $_p2; wait" EXIT

  meson test -C build --print-errorlogs -t 3 || true
)

check() {
  dbus-run-session xvfb-run -s '-nolisten local +iglx -noreset' \
    bash -c "$(declare -f _check); _check"
}

_pick() {
  local p="$1" f d; shift
  for f; do
    d="$srcdir/$p/${f#$pkgdir/}"
    mkdir -p "$(dirname "$d")"
    mv "$f" "$d"
    rmdir -p --ignore-fail-on-non-empty "$(dirname "$f")"
  done
}

package_mutter() {
  provides=(libmutter-11.so)
  groups=(gnome)

  meson install -C build --destdir "$pkgdir"

  _pick docs "$pkgdir"/usr/share/mutter-*/doc
}

package_mutter-docs() {
  pkgdesc+=" (documentation)"
  depends=()

  mv docs/* "$pkgdir"
}

# vim:set sw=2 et:
