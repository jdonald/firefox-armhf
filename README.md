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
    curl -sL https://deb.nodesource.com/setup_11.x | sudo -E bash -
    sudo apt install -y gir1.2-atk-1.0:armhf gir1.2-freedesktop:armhf gir1.2-gdkpixbuf-2.0:armhf \
      gir1.2-notify-0.7:armhf gir1.2-pango-1.0:armhf libatk1.0-dev:armhf libgdk-pixbuf2.0-dev:armhf \
      libgirepository-1.0-1:armhf libgtk-3-dev:armhf gtk+-2.0:armhf libgtk2.0-dev:armhf \
      libpango1.0-dev:armhf libharfbuzz-dev:armhf libicu-dev:armhf libxt-dev:armhf libgtk-3-dev:armhf \
      libstartup-notification0-dev:armhf libasound2-dev:armhf libcurl4-openssl-dev:armhf \
      libdbus-glib-1-dev:armhf libiw-dev:armhf libnotify-dev:armhf libpulse-dev:armhf fakeroot \
      devscripts build-essential dpkg-cross crossbuild-essential-armhf libnss3-tools debhelper \
      autoconf2.13 cdbs clang-3.9 zip nodejs

Install **rustup**:

    curl https://sh.rustup.rs -sSf | sh
    # Edit ~/.bashrc, add Rust sourcing: . ~/.cargo/env
    rustup target add armv7-unknown-linux-gnueabihf

Add Firefox bionic source packages and fetch source:

    cat <<EOF | sudo tee /etc/apt/sources.list.d/firefox.list
    deb [arch=armhf] http://ports.ubuntu.com/ubuntu-ports/ bionic-updates main restricted
    deb-src [arch=armhf] http://ports.ubuntu.com/ubuntu-ports/ bionic-updates main restricted
    EOF

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

The WebRTC module has a bug that can manifest as an error on sbfx instructions. Fix this
to use the appropriate ASFLAGS:

    patch -p1 < webrtc.patch

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

Monitor the build for the course of two hours. When errors arise, troubleshoot then resume.
For example if you see this problem:

    /bin/bash: ./obj-arm-linux-gnueabihf/dist/bin/shlibsign: cannot execute binary file: Exec format error

This means mozbuild made the mistake of running the armhf build of the signing tool. Point it back to the host binary:

    rm obj-arm-linux-gnueabihf/dist/bin/shlibsign 
    ln -s /usr/bin/shlibsign obj-arm-linux-gnueabihf/dist/bin/

Then resume with the **-nc** flag. Next, if you run into the following error minutes later:

    dh_install: Cannot find (any matches for) "usr/lib/firefox/browser/chrome" (tried in "." and "debian/tmp")
    dh_install: firefox missing files: usr/lib/firefox/browser/chrome
    ...
    dh_install: Cannot find (any matches for) "usr/lib/firefox/plugin-container" (tried in "." and "debian/tmp")
    dh_install: firefox missing files: usr/lib/firefox/plugin-container

This means that the whole usr/lib folder never got populated. Create it and copy build artifacts into there:

    mkdir -p usr/lib
    cp -rpf obj-arm-linux-gnueabihf/dist/* usr/lib/

Resume here and it will take a while for all localizations. If you reach the end you'll see the error:

    gpg: skipped "Olivier Tilloy <olivier.tilloy@canonical.com>": No secret key
    gpg: dpkg-sign.8YBNCg8D/firefox_64.0+build3-0ubuntu0.18.04.1_armhf.buildinfo: clear-sign failed: No secret key

    dpkg-buildpackage: error: failed to sign .buildinfo file

Although the message may not sound positive, at that point you'll find a set of deb packages in your home directory.
