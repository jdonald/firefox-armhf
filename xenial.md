# Cross-compiling Firefox for armhf with clang (xenial)

For targeting Ubuntu 16.04 Xenial, start an `ubuntu:xenial`
Docker container then for the most follow the main **README.md**
using the same instructions as Stretch.

In fact the standard Ubuntu repositories may have Xenial sources a
full version behind, so for Firefox 65.0 I still had to add Bionic
sources before running `sudo apt-get source firefox:armhf`.
