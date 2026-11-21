# PiCompose-Package-Driver-2MicHat-V1

[![Version](https://github.com/florian-asche/PiCompose-Package-Driver-2MicHat-V1/actions/workflows/version.yml/badge.svg?branch=main)](https://github.com/florian-asche/PiCompose-Package-Driver-2MicHat-V1/actions/workflows/version.yml) [![GitHub Release Version](https://img.shields.io/github/v/release/florian-asche/PiCompose-Package-Driver-2MicHat-V1?label=version)](https://github.com/florian-asche/PiCompose-Package-Driver-2MicHat-V1/releases) [![GitHub License](https://img.shields.io/github/license/florian-asche/PiCompose-Package-Driver-2MicHat-V1)](https://github.com/florian-asche/PiCompose-Package-Driver-2MicHat-V1/blob/main/LICENSE)

Package `picompose-driver-2michat-v1` for the [PiCompose APT repository](https://github.com/florian-asche/PiCompose-Repository). It is the driver for the Seeed [ReSpeaker 2-Mics Pi HAT](https://wiki.seeedstudio.com/ReSpeaker_2_Mics_Pi_HAT/) (version 1, WM8960 codec, I2C address 0x1a). Version 2 of the HAT has another codec and is the package `picompose-driver-2michat-v2`. The 4-Mics and 6-Mics HATs have an AC108 codec, which the kernel does not have, so the package does not support them.

The PiCompose image built the DKMS driver of [HinTak/seeed-voicecard](https://github.com/HinTak/seeed-voicecard) for this HAT. The package does not: the kernel of Raspberry Pi OS has the driver for the codec (`snd-soc-wm8960`) and the overlay `wm8960-soundcard`, which connects it to the I2S pins of the Raspberry Pi. So no kernel module is built, nothing is downloaded and the package is small.

## Requirements

The package is for Raspberry Pi OS only. It changes the boot configuration of the Raspberry Pi (`config.txt`, `usercfg.txt`), which only has an effect on the Raspberry Pi firmware. The package does not check this when it is installed, so on any other system it installs but does nothing.

## Contents

| Part | What it does |
| --- | --- |
| Boot configuration | The `postinst` enables `dtparam=i2c_arm=on` and `dtparam=spi=on` (SPI is for the APA102 LEDs of the HAT, not for audio) and adds `dtoverlay=wm8960-soundcard,alsaname=seeed-2mic-voicecard` to the boot configuration (`config.txt`, `usercfg.txt` if it exists). It removes the `dtoverlay=seeed-*-voicecard` lines that earlier versions of the package added, because they use the same nodes. The `postrm` removes the overlay line again. Both are safe to run more than once. |
| Mixer routing | `/usr/share/picompose-driver-2michat-v1/seeed2micvoicec.state`, an ALSA state with six controls of the codec: `Left/Right Output Mixer PCM Playback Switch` and `Left/Right Input Mixer Boost Switch` on, `Left Input Boost Mixer LINPUT1` and `Right Input Boost Mixer RINPUT1` at 3. The `postinst` applies it once (marker `/var/lib/picompose-driver-2michat-v1/mixer-state-applied`): if the card is there it runs `alsactl restore` with the file and `alsactl store`, which writes the values into `/var/lib/alsa/asound.state`. If the card is not there yet (the overlay is loaded after the first reboot) it adds the section to `/var/lib/alsa/asound.state`, unless the file already has a section for the card. The volumes are not in the file. The `postrm` removes the marker on purge. |

The overlay is part of the firmware of Raspberry Pi OS and is not shipped by the package. `alsaname=seeed-2mic-voicecard` is the name of the card in the overlay of Seeed, so after a reboot the card appears as `seeed2micvoicec` as before. The WM8960 starts with the DAC not routed to the outputs and the microphones not routed to the ADC, so without this state the card is silent. The package does not set the volumes and does not ship a full ALSA state. It was not tested on hardware.

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
