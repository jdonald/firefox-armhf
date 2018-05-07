# Cross-compiling Firefox for armhf with clang

These are notes for how I did it inside an Ubuntu 14.04 Trusty Docker container.

This is not regularly tested with continuous integration, so if you attempt
the procedure below please expect to run into missing packages or other problems
that you'll have to troubleshoot.

The idea is that these notes can serve as a reference for someone capable to
improve the official build flows, then we won't need this hacky procedure
anymore.

    sudo dpkg --add-architecture armhf

    # Make sure your /etc/apt/sources.list contains the following:
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

    sudo apt update

    sudo apt install gir1.2-atk-1.0:armhf gir1.2-freedesktop:armhf gir1.2-gdkpixbuf-2.0:armhf \
      gir1.2-notify-0.7:armhf gir1.2-pango-1.0:armhf libatk1.0-dev:armhf libgdk-pixbuf2.0-dev:armhf \
      libgirepository-1.0-1:armhf libgtk-3-dev:armhf gtk+-2.0:armhf
    sudo apt install libgtk2.0-dev:armhf libpango1.0-dev:armhf libharfbuzz-dev:armhf \
      libicu-dev:armhf libxt-dev:armhf libgtk-3-dev:armhf libstartup-notification0-dev:armhf \
      libasound2-dev:armhf libcurl4-openssl-dev:armhf libdbus-glib-1-dev:armhf libiw-dev:armhf \
      libnotify-dev:armhf libpulse-dev:armhf \
      fakeroot devscripts build-essential dpkg-cross crossbuild-essential-armhf libnss3-tools \
      debhelper autoconf2.13
    sudo apt install cdbs clang-3.9 zip

    # Install rustup:
    curl https://sh.rustup.rs -sSf | sh
    # Edit ~/.bashrc, add Rust sourcing: . ~/.cargo/env
    rustup target add armv7-unknown-linux-gnueabihf

    # Apply system-wide libstdc++ patch. See gcc-4.8.patch

    apt-get source firefox:armhf
    cd firefox-*
    # Apply Rust build system patch, C99 free() bugfixes patch
    # See build_gecko.rs.patch and c99-free.patch

    PKG_CONFIG_PATH=/usr/lib/arm-linux-gnueabihf/pkgconfig \
      CONFIG_SITE=/etc/dpkg-cross/cross-config.amd64 \
      DEB_BUILD_OPTIONS=nocheck
      dpkg-buildpackage -aarmhf -b -d

    # After configuration errors, replace mozconfig and edit with correct paths
    PKG_CONFIG_PATH=/usr/lib/arm-linux-gnueabihf/pkgconfig \
      CONFIG_SITE=/etc/dpkg-cross/cross-config.amd64 \
      DEB_BUILD_OPTIONS=nocheck
      dpkg-buildpackage -aarmhf -b -d -nc
    # While build is underway, apply libpng backend.mk patch, xpcom/string NEON patch
    # See libpng.patch and utf8neon.patch
