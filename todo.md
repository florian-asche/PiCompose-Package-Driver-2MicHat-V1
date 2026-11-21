# To do for PiCompose-Package-Driver-2MicHat-V1

Open points and work that was postponed. Remove an entry in the change that settles it.

1. **Test the overlay of Seeed on hardware:** Install on Raspberry Pi OS Trixie with the HAT, reboot and check that the card `seeed2micvoicec` appears, plays and records at the right speed (the overlay sets a 24 MHz clock and makes the codec the clock master), and that `/dev/spidev0.0` exists for the LEDs. The version before this one used `wm8960-soundcard` of the kernel: on a test system (kernel 6.18) the card appeared and had a playback and a capture device, sound was not checked. Nothing was tested on a Raspberry Pi with a HAT after the change to the overlay of Seeed.

2. **Check the volume of the WM8960 card:** The package does not set the volume, and the ALSA state of Seeed is deliberately not used. Check on hardware whether the card has a usable volume after a boot, and whether the sink volume reaches the codec. If not, add a profile set like the one of picompose-driver-2michat-v2 (udev property `ACP_PROFILE_SET`) with the real mixer elements of the WM8960 (`Headphone`, `Speaker`, `Playback`).

3. **Pin the download:** The overlay is built from the default branch of seeed-linux-dtoverlays at the time of the build. A pinned commit would make builds repeatable. Decide whether that is wanted.

4. **Package for the 4-Mics and 6-Mics HATs:** They need the AC108 driver, which is not in the kernel, and are not supported by this package. A separate repository with its own package (DKMS driver of HinTak/seeed-voicecard) is planned.

5. **Test the upgrade:** On a system with the earlier versions, check that the old lines (`dtoverlay=wm8960-soundcard,...`, `dtoverlay=seeed-*-voicecard`, `dtoverlay=i2s-mmap`, `dtparam=i2s=on`) and the DKMS modules do not disturb the new overlay. The package removes the overlay lines, but leaves `dtoverlay=i2s-mmap` and `dtparam=i2s=on`.
