# Cross-compiling Firefox for armhf with clang (xenial)

For targeting Ubuntu 16.04 Xenial, start an `ubuntu:xenial`
Docker container then for the most follow the main **README.md**
using the same instructions as Stretch.

In fact the standard Ubuntu repositories may have Xenial sources a
full version behind, so for Firefox 65.0 I still had to add Bionic
sources before running `sudo apt-get source firefox:armhf`.

One minor difference: use `mozconfig.xenial`:

    cp path/to/firefox-armhf/mozconfig.xenial mozconfig

then edit to paths to match your home directory. The only difference
between this and the Stretch configuration is the --enable-crashreporter
flag. Without it, the final packaging step may complain about missing
`usr/lib/firefox/crashreporter`.
