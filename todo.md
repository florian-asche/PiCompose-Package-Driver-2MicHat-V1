# To do for PiCompose-Package-Driver-2MicHat-V1

Open points and work that was postponed. Remove an entry in the change that settles it.

1. **Test the DKMS build against the Raspberry Pi kernel:** The three modules compile against the Debian 6.12 headers (amd64). The link check (modpost) fails there on `simple_util_clean_reference`, which the Debian amd64 kernel does not export. Build the modules against the real kernel of Raspberry Pi OS Trixie (`linux-headers-rpi-v8`, arm64), in a Raspberry Pi OS container or on a Raspberry Pi.

2. **Follow kernel versions:** The package carries the branch `v6.12` of HinTak/seeed-voicecard. The image downloads the branch that matches the kernel it finds. Decide how the package follows new kernels (replace the sources, or carry several branches with several DKMS versions).

3. **Raspberry Pi 5:** Only `linux-headers-rpi-v8` is a dependency, as in the image. Check whether `linux-headers-rpi-2712` is needed for the Raspberry Pi 5.

4. **Review the license information:** `debian/copyright` states GPL-2 for the kernel module sources, because their files and `MODULE_LICENSE` say so, while the upstream repository ships the GPL-3 text. The device tree overlays and the Makefile state the MIT License. Check this against the upstream files.

5. **Review the service that reboots the system:** `seeed-voicecard.service` reboots the system once after it detected the HAT. That is the behavior of the image and is kept. Check whether it is acceptable for a package that is installed on a running system.

6. **Test on hardware:** Nothing was tested on a Raspberry Pi with a HAT.
