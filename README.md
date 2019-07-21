# Cross-compiling Firefox for armhf with clang

The first run in spring 2018 was done in an Ubuntu 14.04 Trusty container,
but this flow has since been revised for a Debian Stretch amd64 container
targeting Raspbian.

The instructions below are for a recent major release such as Firefox
65.0 as this was last revised in February 2019. To build Firefox ESR 60.5,
see **esr.md**. Trusty-specific notes have been moved to **trusty.md**.
There is also a **xenial.md**.

While this is glaringly out-of-date relative to Firefox 68.0.1, Firefox
ESR 60.8, and beyond, as of June 2019 they released Raspbian 10 Buster.
That distro plays nicely with the gcc-based Firefox Bionic binaries and
relieves the needs of most users.

If you attempt the build procedure below please expect to run into
missing packages or other problems that you'll have to troubleshoot. Also
keep an eye on the list of GitHub issues for what others have come across.

If you're interested in maintaining this for newer releases of Firefox,
pull requests are welcome.

    sudo dpkg --add-architecture armhf
    sudo apt update
    curl -sL https://deb.nodesource.com/setup_11.x | sudo -E bash -
    sudo apt install -y gir1.2-atk-1.0:armhf gir1.2-freedesktop:armhf gir1.2-gdkpixbuf-2.0:armhf \
      gir1.2-notify-0.7:armhf gir1.2-pango-1.0:armhf libatk1.0-dev:armhf libgdk-pixbuf2.0-dev:armhf \
      libgirepository-1.0-1:armhf libgtk-3-dev:armhf libgtk2.0-dev:armhf \
      libpango1.0-dev:armhf libharfbuzz-dev:armhf libicu-dev:armhf libxt-dev:armhf \
      libstartup-notification0-dev:armhf libasound2-dev:armhf libcurl4-openssl-dev:armhf \
      libdbus-glib-1-dev:armhf libiw-dev:armhf libnotify-dev:armhf libpulse-dev:armhf fakeroot \
      devscripts build-essential dpkg-cross crossbuild-essential-armhf \
      libreadline-dev:armhf libbz2-dev:armhf libjsoncpp-dev:armhf \
      libnss3-tools debhelper autoconf2.13 cdbs clang-3.9 zip python nodejs

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

We need to patch `build_gecko.rs` to pass a cross-compile flag to its clang options.
`debian/rules` has some apparmor conditions that are too Ubuntu-specific. The
WebRTC module seems to have been patched but is missing essential ASFLAGS.
Firefox 64.0 added a major change:

    # * Explicitly set HOME=/tmp

which breaks existing build flows. Apply the complete **armhf.patch** to fix all
of the above:

    patch -p1 < path/to/firefox-armhf/armhf.patch

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
    gpg: dpkg-sign.8YBNCg8D/firefox_65.0+build2-0ubuntu0.18.04.1_armhf.buildinfo: clear-sign failed: No secret key

    dpkg-buildpackage: error: failed to sign .buildinfo file

Although the message may not sound positive, at that point you'll find a set of deb packages in your home directory.
On a good day you might see a clean finish:

    dpkg-buildpackage: info: binary-only upload (no source included)
