# PiCompose-Package-Driver-2MicHat-V1

Package `picompose-driver-2michat-v1` for the [PiCompose APT repository](https://github.com/florian-asche/PiCompose-Repository). It is the driver for the Seeed [ReSpeaker 2-Mics Pi HAT](https://wiki.seeedstudio.com/ReSpeaker_2_Mics_Pi_HAT/) (version 1, WM8960 codec), and also for the 4-Mics and 6-Mics HATs, which the same service detects. Version 2 of the 2-Mics HAT has another codec and is the package picompose-driver-2michat-v2. It is the stage `02-stage-audiodriver-2michat-v1` of the [PiCompose](https://github.com/florian-asche/PiCompose) image as a package.

## Contents

| Part | Files |
| --- | --- |
| Kernel modules | The source is in `/usr/src/seeed-voicecard-0.3`. DKMS builds `snd-soc-wm8960`, `snd-soc-ac108` and `snd-soc-seeed-voicecard` for every installed kernel, and again for new kernels. `/etc/modules-load.d/picompose-seeed-voicecard.conf` loads them at boot. |
| Device tree overlays | `/boot/firmware/overlays/seeed-2mic-voicecard.dtbo`, `seeed-4mic-voicecard.dtbo`, `seeed-8mic-voicecard.dtbo`, built from the `.dts` files during the package build |
| Card configuration | `/etc/voicecard/` with the `asound_*.conf` and `*_asound.state` files of the HATs |
| Detection service | `/usr/bin/seeed-voicecard-v2`, `seeed-voicecard.service` (enabled, not started at install) |
| Boot configuration | The `postinst` enables `dtparam=i2c_arm=on`, `dtparam=i2s=on`, `dtparam=spi=on` and adds `dtoverlay=i2s-mmap`. The `postrm` removes `i2s-mmap` and the overlay lines again. Both are safe to run more than once. |
| WirePlumber | `/etc/wireplumber/wireplumber.conf.d/51-disable-acp.conf` turns off the ALSA card profile handling (ACP) |
| Volume setup | `/usr/bin/configure_audio.sh`, `configure_audio.service` (enabled, not started at install) |

## What the service does, and a reboot

`seeed-voicecard.service` runs once after the first boot (until `/var/lib/seeed-voicecard/success` exists). It waits for PipeWire, looks on the I2C bus which HAT is there, writes the matching `dtoverlay=` line into the boot configuration, replaces `/etc/asound.conf` and the ALSA state with the ones of the HAT, **and reboots the system**. This is the behavior of the PiCompose image and is kept without changes.

## Kernel version and DKMS

The module sources are the branch `v6.12` of [HinTak/seeed-voicecard](https://github.com/HinTak/seeed-voicecard) (commit `f04dab275d61cbc38af1f4d0e9c188043760ef84`), which matches the kernel 6.12 of Raspberry Pi OS Trixie. The image downloads the branch for the kernel version it finds while it is built. The package carries one branch, so it follows one kernel version. If Raspberry Pi OS moves to another kernel, the sources in `src/seeed-voicecard` have to be replaced with the matching branch.

The package depends on `linux-headers-rpi-v8`, as the image does, which covers the 64-bit kernel of the Raspberry Pi 3, 4 and Zero 2. DKMS builds the modules in the `postinst`, so the headers of the running kernel have to be installed.

Only what the build needs is kept from the upstream repository: the sources of the three modules, the Makefile, `dkms.conf`, the patches, the overlays and the card configuration. The prebuilt overlays and the `ac108` plugin binary of upstream are left out.

## One driver per system

All `picompose-driver-*` packages provide `picompose-audio-driver` and conflict with it. Installing one removes another, because a system has one audio device and all of them install `configure_audio.sh` and `configure_audio.service`.

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
