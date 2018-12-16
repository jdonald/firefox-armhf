# Cross-compiling Firefox for armhf with clang

The first run in spring 2018 was done in an Ubuntu 14.04 Trusty container,
but this flow has since been revised for a Debian Stretch amd64 container
targeting Raspbian.

Trusty-specific notes have been moved to **trusty.md**.

This is not regularly tested with continuous integration, so if you attempt
the procedure below please expect to run into missing packages or other problems
that you'll have to troubleshoot.

    sudo dpkg --add-architecture armhf
    sudo apt update
    sudo apt install gir1.2-atk-1.0:armhf gir1.2-freedesktop:armhf gir1.2-gdkpixbuf-2.0:armhf \
      gir1.2-notify-0.7:armhf gir1.2-pango-1.0:armhf libatk1.0-dev:armhf libgdk-pixbuf2.0-dev:armhf \
      libgirepository-1.0-1:armhf libgtk-3-dev:armhf gtk+-2.0:armhf \
      libgtk2.0-dev:armhf libpango1.0-dev:armhf libharfbuzz-dev:armhf \
      libicu-dev:armhf libxt-dev:armhf libgtk-3-dev:armhf libstartup-notification0-dev:armhf \
      libasound2-dev:armhf libcurl4-openssl-dev:armhf libdbus-glib-1-dev:armhf libiw-dev:armhf \
      libnotify-dev:armhf libpulse-dev:armhf \
      fakeroot devscripts build-essential dpkg-cross crossbuild-essential-armhf libnss3-tools \
      debhelper autoconf2.13 cdbs clang-3.9 zip nodejs npm

Install **rustup**:

    curl https://sh.rustup.rs -sSf | sh
    # Edit ~/.bashrc, add Rust sourcing: . ~/.cargo/env
    rustup target add armv7-unknown-linux-gnueabihf

Update **nodejs** version:

    sudo npm install -g n
    sudo n 9

Add Firefox bionic source packages and fetch source:

    cat | sudo tee /etc/apt/sources.list.d/firefox.list
    deb [arch=armhf]  http://ports.ubuntu.com/ubuntu-ports/ bionic-updates main restricted
    deb-src [arch=armhf] http://ports.ubuntu.com/ubuntu-ports/ bionic-updates main restricted
    ^D

    sudo apt-key adv --recv-key --keyserver keyserver.ubuntu.com 3B4FE6ACC0B21F32
    sudo apt update
    apt-get source firefox:armhf
    cd firefox-*

Apply Rust cross-build patch:

    patch -p1 < build_gecko.rs.patch

Apply `rules.patch` to fix some conditions that are too Ubuntu-specific. Otherwise early
on you'll hit a strange error about being unable to find `usr.bin.firefox.in`

    patch -p1 < rules.patch

Firefox 64.0 has this major change according to the changelog:

    # * Explicitly set HOME=/tmp

which breaks existing build flows. Apply `rules.mk.patch` to remove the line.

Then run **dpkg-buildpackage**:

    PKG_CONFIG_PATH=/usr/lib/arm-linux-gnueabihf/pkgconfig \
      CONFIG_SITE=/etc/dpkg-cross/cross-config.amd64 \
      DEB_BUILD_OPTIONS=nocheck \
      dpkg-buildpackage -aarmhf -b -d

After configuration errors, replace `mozconfig` with the one from this repo.
Make sure you've edited it to use your correct homedir paths. Then resume the build:

    PKG_CONFIG_PATH=/usr/lib/arm-linux-gnueabihf/pkgconfig \
      CONFIG_SITE=/etc/dpkg-cross/cross-config.amd64 \
      DEB_BUILD_OPTIONS=nocheck \
      dpkg-buildpackage -aarmhf -b -d -nc

Monitor the build for the course of two hours. When errors arise, troubleshoot then resume with the **-nc** flag.
