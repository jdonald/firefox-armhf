# Cross-compiling Firefox for armhf with clang (xenial)

For targeting Ubuntu 16.04 Xenial, start an `ubuntu:xenial`
Docker container then for the most follow the main **README.md**
using instructions similar to Stretch:

For Xenial armhf sources, add these lines to your `sources.list`.

    deb [arch=armhf] http://ports.ubuntu.com/ubuntu-ports/ xenial-updates universe
    deb-src [arch=armhf] http://ports.ubuntu.com/ubuntu-ports/ xenial-updates universe

Use the slightly modified `mozconfig.xenial`:

    cp path/to/firefox-armhf/mozconfig.xenial mozconfig

then edit to paths to match your home directory. Note the
--enable-crashreporter flag. Without it, the final packaging
step may complain about missing `usr/lib/firefox/crashreporter`.
