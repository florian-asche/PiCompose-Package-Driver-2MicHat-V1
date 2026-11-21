# To do for PiCompose-Package-Driver-2MicHat-V1

Open points and work that was postponed. Remove an entry in the change that settles it.

1. **Test on hardware:** Install on Raspberry Pi OS Trixie with the HAT, reboot and check that the card `seeed2micvoicec` appears, plays and records. The overlay `wm8960-soundcard` is made for the Waveshare WM8960 HAT. It differs from the overlay of Seeed (it targets `i2s_clk_producer`, the clock is set on the codec node, there is no `master` parameter), so check the clocking of the HAT (the Seeed HAT has its own oscillator). Nothing was tested on a Raspberry Pi with a HAT.

2. **Check the volume of the WM8960 card:** The package does not set the volume, and the ALSA state of Seeed is deliberately not used. Check on hardware whether the card has a usable volume after a boot, and whether the sink volume reaches the codec. If not, add a profile set like the one of picompose-driver-2michat-v2 (udev property `ACP_PROFILE_SET`) with the real mixer elements of the WM8960 (`Headphone`, `Speaker`, `Playback`).

3. **Check that the overlay ships in the images:** Check with `dtoverlay -h wm8960-soundcard` on the Raspberry Pi OS Trixie of the images that the overlay exists and has `alsaname`. It was checked in the source of the kernel branch `rpi-6.12.y`, not on an installed system.

4. **Package for the 4-Mics and 6-Mics HATs:** They need the AC108 driver, which is not in the kernel, and are not supported by this package. A separate repository with its own package (DKMS driver of HinTak/seeed-voicecard) is planned.

5. **Test the upgrade:** On a system with the earlier version, check that the old `dtoverlay=seeed-*-voicecard` and `dtoverlay=i2s-mmap` lines and the DKMS modules do not disturb the new overlay. The package removes the Seeed overlay lines, but leaves `dtoverlay=i2s-mmap` and `dtparam=i2s=on`.
