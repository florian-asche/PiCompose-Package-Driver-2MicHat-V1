# To do for PiCompose-Package-Driver-2MicHat-V1

Open points and work that was postponed. Remove an entry in the change that settles it.

1. **Package for the 4-Mics and 6-Mics HATs:** They need the AC108 driver, which is not in the kernel, and are not supported by this package. A separate repository with its own package (DKMS driver of HinTak/seeed-voicecard) is planned.

2. **Test the upgrade:** On a system with the earlier versions, check that the old lines (`dtoverlay=wm8960-soundcard,...`, `dtoverlay=seeed-*-voicecard`, `dtoverlay=i2s-mmap`, `dtparam=i2s=on`) and the DKMS modules do not disturb the new overlay. The package removes the overlay lines, but leaves `dtoverlay=i2s-mmap` and `dtparam=i2s=on`.
