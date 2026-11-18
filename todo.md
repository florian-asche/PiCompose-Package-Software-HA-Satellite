# To do for PiCompose-Package-Software-HA-Satellite

Open points and work that was postponed. Remove an entry in the change that settles it.

1. **Test on Raspberry Pi OS Trixie:** Deploy both projects on a real image and check the audio socket paths. Nothing was tested on hardware.
2. **`Depends` on `picompose-core`:** It was removed for now, so the package no longer pulls in the compose manager. Add it back once the image is built from the packages and `picompose-core` is released to `trixie`.
