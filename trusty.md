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

Python 3.4 is the default on Trusty, but Firefox's build process requires 3.5 or newer.

    sudo apt install python3.5

Patches have been bundled together in a single **trusty.patch**. For the most part
it's similar to the Stretch patches except using gcc 4.8.4 headers instead of 6.3.0.

    patch -p1 < path/to/firefox-armhf/trusty.patch

For clang to use old gcc headers, you must apply a *system-level* libstdc++ patch:

    (cd / && sudo patch -p1 < path/to/firefox-armhf/gcc-4.8-trusty.patch)

Get Firefox source:

    apt-get source firefox:armhf
    cd firefox-*

Build:

    PKG_CONFIG_PATH=/usr/lib/arm-linux-gnueabihf/pkgconfig \
      CONFIG_SITE=/etc/dpkg-cross/cross-config.amd64 \
      DEB_BUILD_OPTIONS=nocheck \
      dpkg-buildpackage -aarmhf -b -d

After configuration errors, replace mozconfig with `mozconfig.trusty`
and edit paths. It's similar to the included mozconfig except s/6.3.0/4.8.4/

    PKG_CONFIG_PATH=/usr/lib/arm-linux-gnueabihf/pkgconfig \
      CONFIG_SITE=/etc/dpkg-cross/cross-config.amd64 \
      DEB_BUILD_OPTIONS=nocheck \
      dpkg-buildpackage -aarmhf -b -d -nc

This will likely encounter the shlibsign error so work around that
just as described in **README.md**.

The final build step might complain about missing files from debian/tmp, which
is frustrating as the build process nukes the tmp subfolder. To get past that,
try pausing with Ctrl+Z followed by:

    mkdir -p debian/tmp/usr/lib
    cp -rpf obj-arm-linux-gnueabihf/dist/* debian/tmp/usr/lib

Then bring the incremental build back into the foreground with:

    fg
