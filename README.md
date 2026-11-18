# PiCompose-Package-Software-HA-Satellite

[![Version](https://github.com/florian-asche/PiCompose-Package-Software-HA-Satellite/actions/workflows/version.yml/badge.svg?branch=main)](https://github.com/florian-asche/PiCompose-Package-Software-HA-Satellite/actions/workflows/version.yml) [![GitHub Release Version](https://img.shields.io/github/v/release/florian-asche/PiCompose-Package-Software-HA-Satellite?label=version)](https://github.com/florian-asche/PiCompose-Package-Software-HA-Satellite/releases) [![GitHub License](https://img.shields.io/github/license/florian-asche/PiCompose-Package-Software-HA-Satellite)](https://github.com/florian-asche/PiCompose-Package-Software-HA-Satellite/blob/main/LICENSE)

Package `picompose-software-ha-satellite` for the [PiCompose APT repository](https://github.com/florian-asche/PiCompose-Repository). It turns a [PiCompose](https://github.com/florian-asche/PiCompose) system into a Home Assistant voice satellite. It is the stage `03-stage-linux-voice-assistant` of the PiCompose image as a package.

The package was called `ha-satellite` before. It is not published under that name any more, so install the new name.

What differs from how the stage does it is documented in [diff.md](diff.md) (in German).

## Contents

The package ships two Docker Compose projects. Both are deployed by the compose manager of [picompose-core](https://github.com/florian-asche/PiCompose-Package-Core), on which the package depends.

| Project | Image | Default |
| --- | --- | --- |
| `lva` | [linux-voice-assistant](https://github.com/OHF-Voice/linux-voice-assistant) | enabled, redeployed every day at 4 AM with a new image |
| `snapcast` | Snapcast client for audio playback | disabled (`DISABLED=true`) |

The files are installed to `/usr/share/picompose/compose/<project>/` (`docker-compose.yml`, `picompose.conf`, `.env`). The `postinst` replaces `/compose/<project>/docker-compose.yml` on every install and upgrade, so do not edit it. `picompose.conf` and `.env` are copied only if they do not exist yet, so your changes survive upgrades. Edit these two files in `/compose`, for example `.env` for the name of the client or the address of the Snapcast server.

## Notes

- The package ships the files of the image without changes. The default of `SNAPCAST_REMOTE_HOST` in the Snapcast `.env` is an address of the author's network and has to be set.
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
