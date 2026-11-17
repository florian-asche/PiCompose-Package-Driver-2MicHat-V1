# Abweichungen von der Stage `02-stage-audiodriver-2michat-v1`

Dieses Dokument beschreibt, was das Paket `picompose-driver-2michat-v1` anders macht als die gleichnamige Stage im [PiCompose](https://github.com/florian-asche/PiCompose)-Repository. Verglichen wurde mit dem Stand von `main` (Commit `784abfb`, 17.11.2026). Gepflegt wird es auf Deutsch, auf Wunsch des Entwicklers.

Pflege: Jede Änderung, die eine Abweichung neu schafft, ändert oder beseitigt, aktualisiert dieses Dokument. Prüfe dabei gegen den aktuellen `main` von PiCompose.

## Anders gelöst

- **Quellen und Kernel-Version.** Die Stage sucht beim Bau des Images alle installierten Kernel (`/boot/vmlinuz-*`), lädt für jede Major.Minor-Version den passenden Branch von `HinTak/seeed-voicecard` und baut das Modul mit DKMS. Das Paket lädt **beim Bau des Pakets** genau einen Branch (`v6.12`, Variable `SEEED_BRANCH` in `debian/rules`), ebenfalls als Tarball. Läuft das Image mit einem anderen Kernel, muss der Branch angepasst werden. Der Quelltext liegt nicht im Repository, der Download braucht Netzwerkzugriff beim Bau und ist nicht auf einen Commit festgelegt. Schlägt er fehl (HTTP-Fehler), bricht der Bau ab, wie die Stage mit `exit 1`.
- **DKMS.** Die Stage ruft `dkms add`, `dkms build` und `dkms install --force` für jeden Kernel einmal beim Bau des Images auf, im Verzeichnis `/usr/src/seeed-voicecard-<kernelversion>`. Das Paket installiert den ganzen Quellbaum nach `/usr/src/seeed-voicecard-<PACKAGE_VERSION aus dkms.conf>` (derzeit `0.3`) und lässt `dh_dkms` die DKMS-Schritte in der `postinst` ausführen, auf dem Gerät. Die Stage legt außerdem ein Marker-Verzeichnis unter `/var/lib/dkms/...` an, das Paket nicht.
- **Overlays.** Die Stage kopiert die vorkompilierten `seeed-*-voicecard.dtbo` aus dem Tarball nach `/boot/overlays`. Das Paket installiert dieselben vorkompilierten Dateien nach `/boot/firmware/overlays` (Raspberry Pi OS Trixie).
- **Kartenkonfiguration.** Die Stage kopiert `*.conf` und `*.state` aus dem Tarball nach `/etc/voicecard` (auch `dkms.conf`). Das Paket installiert nur die `asound_*.conf` und `*.state`, als Conffiles.
- **Kernelmodule laden.** Die Stage ergänzt `/etc/modules` um `snd-soc-seeed-voicecard`, `snd-soc-ac108` und `snd-soc-wm8960`. Das Paket legt `/etc/modules-load.d/picompose-seeed-voicecard.conf` mit denselben drei Modulen an.
- **`config.txt`.** Gleiche Erkennung der Datei wie in der Stage. Die Stage aktiviert `dtparam=i2c_arm`, `dtparam=i2s` und `dtparam=spi` nur, wenn sie auskommentiert vorhanden sind (und ersetzt an jeder Stelle), und ergänzt `dtoverlay=i2s-mmap`. Das Paket aktiviert die Zeile am Zeilenanfang und **hängt sie an, wenn sie fehlt**. Die `postrm` entfernt `dtoverlay=i2s-mmap` und die `dtoverlay=seeed-*-voicecard`-Zeilen und löscht `/var/lib/seeed-voicecard/success`, die Stage kennt kein Entfernen.
- **Pakete der Stage** (`git`, `build-essential`, `dkms`, `curl`, `linux-headers-rpi-v8`, `i2c-tools`, `libasound2-plugins`, `alsa-utils`): `dkms`, `linux-headers-rpi-v8`, `i2c-tools`, `libasound2-plugins` und `alsa-utils` sind `Depends`. `curl` und `ca-certificates` sind `Build-Depends`. `git` und `build-essential` entfallen.
- **Ausgaben.** Die Debug-Ausgaben des Chroot-Skripts (`ls /proc/version` und andere) gibt es nicht.
- Die Unit-Datei liegt in `/usr/lib/systemd/system/` statt in `/etc/systemd/system/`. Aktiviert wird sie durch `dh_installsystemd` im Paket (beim Installieren aktiviert, aber nicht gestartet) statt durch `systemctl enable` im Chroot der Stage. Der Inhalt der Unit ist unverändert. Das gilt für `seeed-voicecard.service` und `configure_audio.service`.
- `51-disable-acp.conf` ist als Conffile gekennzeichnet.

## Zusätzlich im Paket

- `Depends` auf `wireplumber`, `pipewire-bin` (`pw-cli` im Erkennungsskript) und `kmod` (`modprobe`).
- `Provides`, `Conflicts` und `Replaces` für `picompose-audio-driver`.
- Das Paket installiert den Quellbaum unverändert, auch das vorkompilierte ALSA-Plugin von Upstream, wie die Stage mit `cp -a`. Lintian meldet das für ein Paket der Architektur `all`, die Hinweise sind in `debian/picompose-driver-2michat-v1.lintian-overrides` begründet überschrieben.

## Unverändert

- `seeed-voicecard-v2`, `seeed-voicecard.service`, `configure_audio.sh`, `configure_audio.service` und `51-disable-acp.conf` sind Byte für Byte die Dateien der Stage. Der Dienst startet das System nach der Erkennung einmal neu, wie in der Stage.
