# Cross-compiling Firefox for armhf with clang (trusty)

The main **README.md** describes how to cross-compile from an amd64
Debian Stretch container. Read that first, then for specifics on
building for Ubuntu 14.04 Trusty see these add-on notes. A Trusty
binary is the most portable (run it on Xenial or even Stretch).

Instead of adding Bionic sources, make sure your existing sources.list
contains the expected Trusty ports.

    deb [arch=armhf] http://ports.ubuntu.com/ubuntu-ports/ trusty main restricted
    deb-src [arch=armhf]  http://ports.ubuntu.com/ubuntu-ports/ trusty main restricted
    deb [arch=armhf]  http://ports.ubuntu.com/ubuntu-ports/ trusty-updates main restricted
    deb-src [arch=armhf] http://ports.ubuntu.com/ubuntu-ports/ trusty-updates main restricted
    deb [arch=armhf] http://ports.ubuntu.com/ubuntu-ports/ trusty universe
    deb-src [arch=armhf] http://ports.ubuntu.com/ubuntu-ports/ trusty universe
    deb [arch=armhf] http://ports.ubuntu.com/ubuntu-ports/ trusty-updates universe
    deb-src [arch=armhf] http://ports.ubuntu.com/ubuntu-ports/ trusty-updates universe
    deb [arch=armhf] http://ports.ubuntu.com/ubuntu-ports/ trusty multiverse
    deb-src [arch=armhf] http://ports.ubuntu.com/ubuntu-ports/ trusty multiverse
    deb [arch=armhf] http://ports.ubuntu.com/ubuntu-ports/ trusty-updates multiverse
    deb-src [arch=armhf] http://ports.ubuntu.com/ubuntu-ports/ trusty-updates multiverse

Then sudo apt update and install the many packages listed in README in addition
to **rustup** and **node** version 11.

Specific to Trusty: Apply a *system-level* libstdc++ patch. See `gcc-4.8.trusty.patch`

    apt-get source firefox:armhf
    cd firefox-*

Apply Trusty-specific Rust build system patch: `build_gecko.rs-trusty.patch`
It's basically the same as `build_gecko.rs.patch` except gcc 4.8.4 is substituted for 6.3.0

Then build:

    PKG_CONFIG_PATH=/usr/lib/arm-linux-gnueabihf/pkgconfig \
      CONFIG_SITE=/etc/dpkg-cross/cross-config.amd64 \
      DEB_BUILD_OPTIONS=nocheck \
      dpkg-buildpackage -aarmhf -b -d

After configuration errors, replace mozconfig with `mozconfig.trusty`
Again it's very similar to the included mozconfig except s/6.3.0/4.8.4/

    PKG_CONFIG_PATH=/usr/lib/arm-linux-gnueabihf/pkgconfig \
      CONFIG_SITE=/etc/dpkg-cross/cross-config.amd64 \
      DEB_BUILD_OPTIONS=nocheck \
      dpkg-buildpackage -aarmhf -b -d -nc
