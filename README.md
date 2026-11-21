# PiCompose-Package-Driver-2MicHat-V1

[![Version](https://github.com/florian-asche/PiCompose-Package-Driver-2MicHat-V1/actions/workflows/version.yml/badge.svg?branch=main)](https://github.com/florian-asche/PiCompose-Package-Driver-2MicHat-V1/actions/workflows/version.yml) [![GitHub Release Version](https://img.shields.io/github/v/release/florian-asche/PiCompose-Package-Driver-2MicHat-V1?label=version)](https://github.com/florian-asche/PiCompose-Package-Driver-2MicHat-V1/releases) [![GitHub License](https://img.shields.io/github/license/florian-asche/PiCompose-Package-Driver-2MicHat-V1)](https://github.com/florian-asche/PiCompose-Package-Driver-2MicHat-V1/blob/main/LICENSE)

Package `picompose-driver-2michat-v1` for the [PiCompose APT repository](https://github.com/florian-asche/PiCompose-Repository). It is the driver for the Seeed [ReSpeaker 2-Mics Pi HAT](https://wiki.seeedstudio.com/ReSpeaker_2_Mics_Pi_HAT/) (version 1, WM8960 codec, I2C address 0x1a). Version 2 of the HAT has another codec and is the package `picompose-driver-2michat-v2`. The 4-Mics and 6-Mics HATs have an AC108 codec, which the kernel does not have, so the package does not support them.

The PiCompose image built the DKMS driver of [HinTak/seeed-voicecard](https://github.com/HinTak/seeed-voicecard) for this HAT. The package does not: the kernel of Raspberry Pi OS has the driver for the codec (`snd-soc-wm8960`), so only the overlay that connects it to the I2S pins of the Raspberry Pi is needed. No kernel module is built.

## Requirements

The package is for Raspberry Pi OS only. It copies the overlay to `/boot/firmware/overlays` and changes the boot configuration of the Raspberry Pi (`config.txt`, `usercfg.txt`), which only has an effect on the Raspberry Pi firmware. The package does not check this when it is installed, so on any other system it installs but does nothing.

## Contents

| Part | Files |
| --- | --- |
| Device tree overlay | `/usr/lib/picompose-driver-2michat-v1/respeaker-2mic-v1_0.dtbo`, built while the package is built, see below. The `postinst` copies it to `/boot/firmware/overlays/respeaker-2mic-v1_0.dtbo` (or `/boot/overlays` if that is where the overlays are), the `postrm` removes the copy. |
| Boot configuration | The `postinst` enables `dtparam=i2c_arm=on` and `dtparam=spi=on` (SPI is for the APA102 LEDs of the HAT, not for audio) and adds `dtoverlay=respeaker-2mic-v1_0` to the boot configuration (`config.txt`, `usercfg.txt` if it exists). It removes the overlay lines that earlier versions of the package added (`dtoverlay=wm8960-soundcard,alsaname=seeed-2mic-voicecard` and `dtoverlay=seeed-*-voicecard`), because they use the same nodes. The `postrm` removes the overlay line again. Both are safe to run more than once. |
| Volume | A profile set for the sound card, so the volume of the sink controls `Headphone` (3.5 mm jack) and `Speaker` of the codec and sets the digital volume `Playback` to 0 dB: `/usr/lib/udev/rules.d/89-picompose-driver-2michat-v1.rules` selects it, `/usr/share/alsa-card-profile/mixer/profile-sets/picompose-2michat-v1.conf` and `/usr/share/alsa-card-profile/mixer/paths/picompose-2michat-v1-output.conf` define it |

The overlay names the card `seeed2micvoicec`. It sets the clock of the codec to 24 MHz and makes the codec the master of the I2S clocks, which the overlay `wm8960-soundcard` of the kernel (made for the Waveshare HAT) does not. The codec starts with `Headphone` at 0 (-121 dB), which is nearly silent, and the standard path of PipeWire does not control it. The profile set makes WirePlumber (ACP) merge `Headphone` and `Speaker` into the volume of the sink, so the sink volume of 100 % puts both at their maximum (+6 dB). It takes effect when the card is detected, so after a reboot, or after `udevadm trigger --action=change --subsystem-match=sound` and a restart of WirePlumber. The package does not load an ALSA state. The overlay takes effect after a reboot. The profile set was tested by hand on one system, the package was not.

The 4-Mics and 6-Mics HATs have an AC108 codec, which the kernel does not have, so the package does not support them.

## Source of the overlay

The package build (`debian/rules`) fetches one commit of the repository [seeed-linux-dtoverlays](https://github.com/Seeed-Studio/seeed-linux-dtoverlays) (`SEEED_COMMIT` in `debian/rules`) and runs `make overlays/rpi/respeaker-2mic-v1_0-overlay.dtbo`, so building needs network access, and the same commit gives the same overlay. To follow upstream, set `SEEED_COMMIT` to a newer commit. No source is stored in this repository. The overlay is copied to `/boot/firmware/overlays` and not installed there by dpkg: that directory is on a FAT partition without hard links, where dpkg cannot replace a file during an upgrade.

## One driver per system

All `picompose-driver-*` packages provide `picompose-audio-driver` and conflict with it. Installing one removes another, because a system has one audio device.

## Versioning and changelog

The version of the package is the first entry of `debian/changelog`, which is part of the package. You do not edit it by hand, the workflows write it.

1. **Pull request:** Every pull request needs exactly one of the labels `major`, `minor` and `patch`. Tick one box under "Version bump" in the description or set the label by hand. The check "Version label" fails until there is exactly one.
2. **Merge to `main`:** The workflow "Version" raises the version, adds the pull request titles to `debian/changelog`, commits it to `main`, creates the tag `v<version>` as pre-release and starts the build, which publishes to the suite `trixie-nightly`.
3. **Release:** Turn the pre-release into a release in the GitHub UI. That publishes the same version to the suite `trixie`.
4. **Any other branch:** `<version>+dev<timestamp>.<sha>`, kept as workflow artifact for 14 days, nothing is published.

The repository needs the secret `HUB_DISPATCH_TOKEN` and a branch protection of `main` that requires the check "Version label" and lets `github-actions[bot]` push, see the [DEVELOPMENT.md](https://github.com/florian-asche/PiCompose-Repository/blob/main/DEVELOPMENT.md) of the APT repository. The repository also has to be listed in `sources.yml` of the APT repository.

## Local build

```sh
sudo apt install build-essential debhelper devscripts fakeroot lintian
dpkg-buildpackage -us -uc -b
lintian ../picompose-driver-2michat-v1_*.deb
```

## License

This project is released under the [MIT License](LICENSE).
