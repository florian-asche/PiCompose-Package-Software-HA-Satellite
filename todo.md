# To do for PiCompose-Package-Software-HA-Satellite

Open points and work that was postponed. Remove an entry in the change that settles it.

2. **`Depends` on `picompose-core`:** It was removed for now, so the package no longer pulls in the compose manager. Add it back once the image is built from the packages and `picompose-core` is released to `trixie`.
4. **Pairing of `local-audio`:** `:latest` is the release 0.2.0 (sendspin-cli 0.3.0) today. The next release (sendspin-cli 0.4.0) requires pairing and refuses `id`. `SENDSPIN_ALLOW_UNPAIRED` is `true` in the `.env.example` for now. Check the compose file and the `.env.example` once it is released.
