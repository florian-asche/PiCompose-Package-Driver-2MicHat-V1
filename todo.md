# To do for PiCompose-Package-Driver-2MicHat-V1

Open points and work that was postponed. Remove an entry in the change that settles it.

1. **Finish the test of the overlay of Seeed:** On Raspberry Pi OS Trixie (kernel 6.18) with the HAT the card `seeed2micvoicec` appears, plays and records, `/dev/spidev0.0` exists and the volume profile set is used after a reboot (package 1.0.2). Not checked: the playback speed and pitch (the overlay sets a 24 MHz clock and makes the codec the clock master), whether the volume is comfortable at a sink volume of 100 % (both outputs at +6 dB), and the LEDs on the SPI bus.

2. **Pin the download:** The overlay is built from the default branch of seeed-linux-dtoverlays at the time of the build. A pinned commit would make builds repeatable. Decide whether that is wanted.

3. **Package for the 4-Mics and 6-Mics HATs:** They need the AC108 driver, which is not in the kernel, and are not supported by this package. A separate repository with its own package (DKMS driver of HinTak/seeed-voicecard) is planned.

4. **Test the upgrade:** On a system with the earlier versions, check that the old lines (`dtoverlay=wm8960-soundcard,...`, `dtoverlay=seeed-*-voicecard`, `dtoverlay=i2s-mmap`, `dtparam=i2s=on`) and the DKMS modules do not disturb the new overlay. The package removes the overlay lines, but leaves `dtoverlay=i2s-mmap` and `dtparam=i2s=on`.
