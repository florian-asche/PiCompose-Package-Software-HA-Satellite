# PiCompose-Package-Software-HA-Satellite

[![Version](https://github.com/florian-asche/PiCompose-Package-Software-HA-Satellite/actions/workflows/version.yml/badge.svg?branch=main)](https://github.com/florian-asche/PiCompose-Package-Software-HA-Satellite/actions/workflows/version.yml) [![GitHub Release Version](https://img.shields.io/github/v/release/florian-asche/PiCompose-Package-Software-HA-Satellite?label=version)](https://github.com/florian-asche/PiCompose-Package-Software-HA-Satellite/releases) [![GitHub License](https://img.shields.io/github/license/florian-asche/PiCompose-Package-Software-HA-Satellite)](https://github.com/florian-asche/PiCompose-Package-Software-HA-Satellite/blob/main/LICENSE)

Package `picompose-software-ha-satellite` for the [PiCompose APT repository](https://github.com/florian-asche/PiCompose-Repository). It turns a [PiCompose](https://github.com/florian-asche/PiCompose) system into a Home Assistant voice satellite. It is the stage `03-stage-linux-voice-assistant` of the PiCompose image as a package.

The package was called `ha-satellite` before. It is not published under that name any more, so install the new name.

## Contents

The package ships five Docker Compose projects. They are deployed by the compose manager of [picompose-core](https://github.com/florian-asche/PiCompose-Package-Core). The package does not depend on it yet, so it has to be installed by other means (the PiCompose image does that).

| Project | Image | Default |
| --- | --- | --- |
| `lva` | [linux-voice-assistant](https://github.com/OHF-Voice/linux-voice-assistant) | enabled, redeployed every day at 4 AM with a new image |
| `snapcast` | Snapcast client for audio playback | disabled (`DISABLED=true`) |
| `local-audio` | [Music Assistant Local Audio](https://github.com/music-assistant/local-audio-addon), a Sendspin player for Music Assistant | enabled, redeployed every day at 4 AM with a new image |
| `bt-proxy` | [bt-proxy](https://github.com/denvera/bt-proxy), an ESPHome-compatible Bluetooth proxy | enabled, redeployed every day at 4 AM with a new image |
| `lpc` | [linux-peripherals-controller](https://github.com/OHF-Voice/linux-peripherals-controller), LEDs, buttons and HATs for the voice assistant | disabled (`DISABLED=true`) |

The files are installed to `/usr/share/picompose/compose/<project>/` (`docker-compose.yml`, `picompose.conf`, `.env.example`). The `postinst` replaces `/compose/<project>/docker-compose.yml` and `/compose/<project>/.env.example` on every install and upgrade, so do not change them. `/compose/<project>/.env` is copied from `.env.example` only if it does not exist yet, so your settings survive upgrades; compare it with the new `.env.example` to find new options. A project whose containers are running is redeployed afterwards (`docker compose down`, then `up --detach`), so it picks up the new files; a failure of that does not fail the installation. `picompose.conf` is copied only if it does not exist yet, so your changes to it survive upgrades too.

The `local-audio` project runs the image with its own s6-overlay as the user `LOCAL_AUDIO_USER_ID` (1000 by default). The options of the player are the `SENDSPIN_*` variables of the `.env`, and the settings of the project (image, tag, user, sound server) are `LOCAL_AUDIO_*`. The player announces itself over mDNS through the Avahi of the host (`/var/run/dbus` is mounted read-only), so the bundled D-Bus and Avahi are not used. Set `SENDSPIN_SERVER` to dial out to a server instead. Unless `SENDSPIN_NAME` is set, the player is named after the host. The control socket of the player (`sendspin-cli status`, `pause`, `vol`, ...) is `control.sock` in the directory `LOCAL_AUDIO_CONTROL_DIR` on the host (default `/run/sendspin-cli`, mode 0600, owned by the user of the container). The `lva` project mounts the same directory (`SENDSPIN_CONTROL_DIR`) at the same path. The voice assistant looks for `control.sock` in it (`--sendspin-control-dir`) and controls the player unless `ENABLE_SENDSPIN_BRIDGE` is `0`: it ducks the player while you talk and for announcements and shows its state in the media player. This needs a linux-voice-assistant image with the Sendspin bridge (the option `--enable-sendspin-bridge`), an older image ignores the socket. The socket belongs to the user of the `local-audio` container, so `LVA_USER_ID` has to match `LOCAL_AUDIO_USER_ID`. Ducking changes the group volume of the Sendspin group, not only this device. The player listens on port 8928 (`SENDSPIN_PORT`); to run several instances, copy the project and give every instance its own `SENDSPIN_PORT`, compose project and `LOCAL_AUDIO_CONTROL_DIR`. A server has to pair with the player before it can play: `SENDSPIN_ALLOW_UNPAIRED` is `true` in the `.env` for now, see the `.env` for the options.

## Notes

- `bt-proxy` turns the Bluetooth adapter of the host into an ESPHome Bluetooth proxy for Home Assistant ([issue 476](https://github.com/OHF-Voice/linux-voice-assistant/issues/476) of linux-voice-assistant). It needs the network of the host (mDNS), the BlueZ of the host over `/var/run/dbus` and `privileged: true` (as documented by the project). Home Assistant discovers it like an ESPHome device and the API is plaintext. It listens on port 6054 (`BT_PROXY_PORT`), because 6053 is used by the `lva` project on the same host. Its options are the `BT_PROXY_*` variables of the `.env`, passed as command line options by the entrypoint. Unless `BT_PROXY_NAME` and `BT_PROXY_FRIENDLY_NAME` are set, the names are `ble-<mac>` and `BT-Proxy - <mac>`, with the MAC address of the default network interface. On a fresh install the Bluetooth adapter is soft-blocked by rfkill. The entrypoint unblocks it through `/dev/rfkill` of the privileged container and powers it on (`bluetoothctl power on` over D-Bus) before it starts the proxy, unless `BT_PROXY_RFKILL_UNBLOCK` is `false`. The unblock is kept by `systemd-rfkill` on the host, also after a reboot. It was not tested on a Raspberry Pi with a Bluetooth adapter.

- `lpc` drives the LEDs, buttons and HATs of the device for the `lva` project. It connects to the peripheral WebSocket API of the voice assistant (`localhost:6055`) and appears in Home Assistant as an ESPHome device of its own. It needs `DEVICE_TYPE` in its `.env` (the known types are in the [configuration of LPC](https://github.com/OHF-Voice/linux-peripherals-controller/blob/main/docs/configuration.md)) and is disabled until it is enabled in its `picompose.conf`, because it does not start without a device type. It needs the network of the host (mDNS) and `privileged: true` for GPIO, SPI, USB and HID. Its ESPHome API listens on port 6056 (`PORT`), because 6053 is used by `lva`, 6054 by `bt-proxy` and 6055 by the peripheral API of the voice assistant. LPC has no release yet, so the `.env.example` sets `LPC_IMAGE_TAG="nightly"`. The device configuration (`DEVICE_CONFIG`) is read from the volume `config`, the child lock, the light and the encryption key are kept in the volume `data`. It was not tested on a Raspberry Pi.

- `local-audio` needs the network of the host (mDNS and the port of the player, 8928 by default) and an Avahi daemon on the host for discovery. The service sets `security_opt: apparmor=unconfined`, because the Docker profile `docker-default` of an AppArmor host (for example Ubuntu) blocks the announcement (`could not register _sendspin._tcp`). On the PiCompose image AppArmor was not enforced when checked, so the option changes nothing there. Both the announcement through the Avahi of the host and the audio output were not tested on a Raspberry Pi.
- The default of `SNAPCAST_REMOTE_HOST` in the Snapcast `.env` is the placeholder `tcp://<IP-OF-YOUR-MUSICASSISTANT-SERVER>` and has to be set before the project is enabled.
- A new version of the package replaces `docker-compose.yml` and `.env.example`. An existing `.env` and `picompose.conf` are never changed, so new options have to be copied from the `.env.example` by hand. The `.env` of `local-audio` uses the `SENDSPIN_*` options and `LOCAL_AUDIO_CONTROL_DIR` (and the `.env` of `lva` `SENDSPIN_CONTROL_DIR` and `ENABLE_SENDSPIN_BRIDGE`).
- Hardware needs a driver package of its own, see the packages `picompose-driver-*`.

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
lintian ../picompose-software-ha-satellite_*.deb
```

## License

This project is released under the [MIT License](LICENSE).
