# To do for PiCompose-Package-Driver-2MicHat-V1

Open points and work that was postponed. Remove an entry in the change that settles it.

1. **Test the mixer state after a reboot:** On a test system (kernel 6.18, overlay `wm8960-soundcard`) playback and microphone worked after the six controls of the state file were set by hand with `amixer`, and `alsactl -f <state> restore seeed2micvoicec` set them. Install the package, reboot, and check that the controls are still on (`amixer -c seeed2micvoicec cget name='Left Output Mixer PCM Playback Switch'`), that `/var/lib/alsa/asound.state` has them, and that playback and microphone work without a manual step. Also check the first install on a system where the card is not there yet. The overlay of Seeed was not tested on hardware, and whether the clock of `wm8960-soundcard` fits the HAT (playback speed, pitch) was not checked by measuring.

2. **Check the volume of the WM8960 card:** The mixer service switches the routing on, but the package does not set the volumes, and the ALSA state of Seeed is deliberately not used. Check on hardware whether the card has a usable volume after a boot, and whether the sink volume reaches the codec. If not, add a profile set like the one of picompose-driver-2michat-v2 (udev property `ACP_PROFILE_SET`) with the real mixer elements of the WM8960 (`Headphone`, `Speaker`, `Playback`).

3. **Check that the overlay ships in the images:** Check with `dtoverlay -h wm8960-soundcard` on the Raspberry Pi OS Trixie of the images that the overlay exists and has `alsaname`. It was checked in the source of the kernel branch `rpi-6.12.y`, not on an installed system.

4. **Package for the 4-Mics and 6-Mics HATs:** They need the AC108 driver, which is not in the kernel, and are not supported by this package. A separate repository with its own package (DKMS driver of HinTak/seeed-voicecard) is planned.

5. **Test the upgrade:** On a system with the earlier version, check that the old `dtoverlay=seeed-*-voicecard` and `dtoverlay=i2s-mmap` lines and the DKMS modules do not disturb the new overlay. The package removes the Seeed overlay lines, but leaves `dtoverlay=i2s-mmap` and `dtparam=i2s=on`.

6. **Card section with old values:** If the card is not there when the package is installed and `/var/lib/alsa/asound.state` already has a section for it (for example from an earlier installation), the `postinst` leaves it alone and the routing stays off. Decide whether the section should be replaced in that case.
