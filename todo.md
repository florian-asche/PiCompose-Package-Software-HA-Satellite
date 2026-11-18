# To do for PiCompose-Package-Software-HA-Satellite

Open points and work that was postponed. Remove an entry in the change that settles it.

1. **Default of SNAPCAST_REMOTE_HOST:** The `.env` of the Snapcast project has `tcp://192.168.33.5`, an address of the author's network, as in the image. Decide on a neutral default.

2. **Snapcast in its own package:** The package contains the whole stage `03-stage-linux-voice-assistant`, so Snapcast too. Decide whether Snapcast should be a package of its own.

3. **Test on Raspberry Pi OS Trixie:** Deploy both projects on a real image and check the audio socket paths. Nothing was tested on hardware.
