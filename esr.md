# Cross-compiling Firefox ESR for armhf with clang (stretch)

The main **README.md** focuses on building the latest release of
Firefox for Stretch, while this document is specific to Firefox
ESR while still targeting Stretch.

In a Debian Stetch Docker container, install all the dependencies
as listed in **README.md**, including rustup and its armv7-unknown-linux-gnueabihf target.

In addition, libffi-dev:armhf seems to be required for ESR.

    sudo apt install libffi-dev:armhf

Next, add, add sources from debian-security to your `/etc/apt/sources.list`

    deb-src http://security.debian.org/ stretch/updates main contrib non-free

Then:

    sudo apt update
    apt-get source firefox-esr:armhf
    cd firefox-esr-*

Going back to Firefox 60.5 restores bugs that were fixed some time ago.
Also as this is not from Ubuntu sources, I ended up hitting
issues that may be due to hacks in Debian-specific patches. In any case,
I lumped all the workarounds together in `esr.patch`.

    patch -p1 < path/to/firefox-armhf/esr.patch

It seems the PYTHON and SHELL variables are practically required now.
At this point there are so many environment variables that it makes
sense to export all of them into the local shell.

    export PKG_CONFIG_PATH=/usr/lib/arm-linux-gnueabihf/pkgconfig
    export CONFIG_SITE=/etc/dpkg-cross/cross-config.amd64
    export DEB_BUILD_OPTIONS=nocheck
    export PYTHON=/usr/bin/python
    export SHELL=/bin/bash

Then attempt to build:

    dpkg-buildpackage -aarmhf -b -d

It'll fail due to an architecture mismatch in mozconfig which
is now called `debian/firefox-esr.mozconfig`. Use the provided
custom one.

    cp path/to/firefox-armhf/firefox-esr.mozconfig debian/

Then continue the build:

    dpkg-buildpackage -aarmhf -b -d -nc
