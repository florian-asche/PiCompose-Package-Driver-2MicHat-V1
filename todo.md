# To do for PiCompose-Package-Driver-2MicHat-V1

Open points and work that was postponed. Remove an entry in the change that settles it.

1. **Test the DKMS build against the Raspberry Pi kernel:** The three modules compile against the Debian 6.12 headers (amd64). The link check (modpost) fails there on `simple_util_clean_reference`, which the Debian amd64 kernel does not export. Build the modules against the real kernel of Raspberry Pi OS Trixie (`linux-headers-rpi-v8`, arm64), in a Raspberry Pi OS container or on a Raspberry Pi.

2. **Follow kernel versions:** The package build downloads one branch (`SEEED_BRANCH` in `debian/rules`, today `v6.12`). The image downloads the branch that matches each kernel it finds. Decide how the package follows new kernels.

3. **Pin the download:** The tarball of the branch is downloaded without a fixed commit or checksum, as in the image. A pinned commit or a checksum would make builds repeatable. Decide whether that is wanted.

4. **Raspberry Pi 5:** Only `linux-headers-rpi-v8` is a dependency, as in the image. Check whether `linux-headers-rpi-2712` is needed for the Raspberry Pi 5.

5. **Review the license information:** The package installs the upstream tarball unchanged (kernel module sources, prebuilt overlays and a prebuilt ALSA plugin). `debian/copyright` points to the upstream files instead of naming a license, because the sources are downloaded. Check what upstream states (the kernel modules say GPL version 2, the repository ships the GPL-3 text, the overlays and the Makefile say MIT) and whether `debian/copyright` should say it.

6. **Review the service that reboots the system:** `seeed-voicecard.service` reboots the system once after it detected the HAT. That is the behavior of the image and is kept. Check whether it is acceptable for a package that is installed on a running system.

7. **Test on hardware:** Nothing was tested on a Raspberry Pi with a HAT.
