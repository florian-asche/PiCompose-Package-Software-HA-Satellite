# To do for PiCompose-Package-Software-HA-Satellite

Open points and work that was postponed. Remove an entry in the change that settles it.

1. **Test on Raspberry Pi OS Trixie:** Deploy both projects on a real image and check the audio socket paths. Nothing was tested on hardware.
2. **`Depends` on `picompose-core`:** It was removed for now, so the package no longer pulls in the compose manager. Add it back once the image is built from the packages and `picompose-core` is released to `trixie`.
3. **Test `local-audio` on Raspberry Pi OS:** The announcement over mDNS through the host Avahi (D-Bus socket mounted into the container) failed on an Ubuntu development machine (`dns_sd error -65537`, `dbus-send` got no reply from the system bus, also with `apparmor=unconfined`). Check it on the image, and keep `SENDSPIN_SERVER` as the fallback if it does not work. The audio output (`pulse`) was not tested either, only `null`.
4. **Pairing of `local-audio`:** `:latest` is the release 0.2.0 (sendspin-cli 0.3.0) today. The next release (sendspin-cli 0.4.0) requires pairing and refuses `id`. `SENDSPIN_ALLOW_UNPAIRED` is `true` in the `.env` for now. Check the compose file and the `.env` once it is released.
5. **Better update method for `.env`:** The `postinst` replaces `docker-compose.yml` and `.env` on every `configure`, so user changes to the `.env` are lost (noted at the top of the README). Replace this with a method that keeps user settings, then restore "copy only if missing" for the `.env` and remove the note.
