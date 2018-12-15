# Cross-compiling Firefox for armhf with clang

The first run in spring 2018 was done in an Ubuntu 14.04 Trusty container,
but this flow has since been revised for a Debian Stretch x86\_64 container
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
      debhelper autoconf2.13 cdbs clang-3.9 zip

Install **rustup**:

    curl https://sh.rustup.rs -sSf | sh
    # Edit ~/.bashrc, add Rust sourcing: . ~/.cargo/env
    rustup target add armv7-unknown-linux-gnueabihf

Add Firefox bionic source packages and fetch source:

    cat | sudo tee /etc/apt/sources.list.d/firefox.list
    deb [arch=armhf]  http://ports.ubuntu.com/ubuntu-ports/ bionic-updates main restricted
    deb-src [arch=armhf] http://ports.ubuntu.com/ubuntu-ports/ bionic-updates main restricted
    ^D

    sudo apt update
    apt-get source firefox:armhf
    cd firefox-*

Apply Rust build system patch:

    # See build_gecko.rs.patch

Firefox 64 has this major difference according to changelog:

    # * Explicitly set HOME=/tmp

which seems to break existing build flows. Edit debian/rules to remove the line.

Then run **dpkg-buildpackage**:

    PKG_CONFIG_PATH=/usr/lib/arm-linux-gnueabihf/pkgconfig \
      CONFIG_SITE=/etc/dpkg-cross/cross-config.amd64 \
      DEB_BUILD_OPTIONS=nocheck \
      dpkg-buildpackage -aarmhf -b -d

After configuration errors, replace `mozconfig` and edit with correct paths, then resume build:

    PKG_CONFIG_PATH=/usr/lib/arm-linux-gnueabihf/pkgconfig \
      CONFIG_SITE=/etc/dpkg-cross/cross-config.amd64 \
      DEB_BUILD_OPTIONS=nocheck \
      dpkg-buildpackage -aarmhf -b -d -nc

Monitor the build for the course of two hours. When errors arise, troubleshoot then resume with the -nc flag.
