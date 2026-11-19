# PiCompose-Package-Software-HA-Satellite

[![Version](https://github.com/florian-asche/PiCompose-Package-Software-HA-Satellite/actions/workflows/version.yml/badge.svg?branch=main)](https://github.com/florian-asche/PiCompose-Package-Software-HA-Satellite/actions/workflows/version.yml) [![GitHub Release Version](https://img.shields.io/github/v/release/florian-asche/PiCompose-Package-Software-HA-Satellite?label=version)](https://github.com/florian-asche/PiCompose-Package-Software-HA-Satellite/releases) [![GitHub License](https://img.shields.io/github/license/florian-asche/PiCompose-Package-Software-HA-Satellite)](https://github.com/florian-asche/PiCompose-Package-Software-HA-Satellite/blob/main/LICENSE)

Package `picompose-software-ha-satellite` for the [PiCompose APT repository](https://github.com/florian-asche/PiCompose-Repository). It turns a [PiCompose](https://github.com/florian-asche/PiCompose) system into a Home Assistant voice satellite. It is the stage `03-stage-linux-voice-assistant` of the PiCompose image as a package.

The package was called `ha-satellite` before. It is not published under that name any more, so install the new name.

## Contents

The package ships three Docker Compose projects. Both are deployed by the compose manager of [picompose-core](https://github.com/florian-asche/PiCompose-Package-Core). The package does not depend on it yet, so it has to be installed by other means (the PiCompose image does that).

| Project | Image | Default |
| --- | --- | --- |
| `lva` | [linux-voice-assistant](https://github.com/OHF-Voice/linux-voice-assistant) | enabled, redeployed every day at 4 AM with a new image |
| `snapcast` | Snapcast client for audio playback | disabled (`DISABLED=true`) |
| `local-audio` | [Music Assistant Local Audio](https://github.com/music-assistant/local-audio-addon), a Sendspin player for Music Assistant | enabled, redeployed every day at 4 AM with a new image |

The files are installed to `/usr/share/picompose/compose/<project>/` (`docker-compose.yml`, `picompose.conf`, `.env`). The `postinst` replaces `/compose/<project>/docker-compose.yml` on every install and upgrade, so do not edit it. `picompose.conf` and `.env` are copied only if they do not exist yet, so your changes survive upgrades. Edit these two files in `/compose`, for example `.env` for the name of the client or the address of the Snapcast server.

The `local-audio` image starts its processes with s6-overlay. The compose file replaces the entrypoint with a short script that renders the player configuration from the `LOCAL_AUDIO_*` options of the `.env` and starts only `sendspin-cli`. The player announces itself over mDNS through the Avahi of the host (`/var/run/dbus` is mounted read-only), so the bundled D-Bus and Avahi are not needed. Set `LOCAL_AUDIO_SERVER` to dial out to a server instead. The control socket of the player (`sendspin-cli status`, `pause`, `vol`, ...) is on the host as `/run/<container name>/control.sock`, with the container name from `LOCAL_AUDIO_CONTAINER_NAME` (default `local-audio`). It belongs to the user of the container and is mode 0600. The `lva` project mounts the directory of that socket (`/run/local-audio`, option `LVA_SENDSPIN_CONTROL_DIR`) to `/sendspin` and passes `SENDSPIN_CONTROL_SOCKET`, so the voice assistant ducks the player while you talk and pauses it for announcements. This needs a linux-voice-assistant image that knows the option `--sendspin-control-socket`, an older image ignores it. The socket is mode 0600 and belongs to the user of the `local-audio` container, so `LVA_USER_ID` has to match `LOCAL_AUDIO_USER_ID`. Ducking changes the group volume of the Sendspin group, not only this device. To run several instances, copy the project and give every instance its own container name, `LOCAL_AUDIO_PORT` and compose project, so that sockets, ports and data stay separate. Newer images require pairing, see the `.env` for the options.

## Notes

- `local-audio` needs the port 8928 (host network) and an Avahi daemon on the host for discovery. Both the announcement through the Avahi of the host and the audio output were not tested on a Raspberry Pi.
- The default of `SNAPCAST_REMOTE_HOST` in the Snapcast `.env` is the placeholder `tcp://<IP-OF-YOUR-MUSICASSISTANT-SERVER>` and has to be set before the project is enabled.
- A new version of the package updates the `docker-compose.yml` files but never changes an existing `picompose.conf` or `.env`. New options of a `.env` have to be added by hand.
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
