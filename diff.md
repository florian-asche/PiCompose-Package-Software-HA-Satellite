# Abweichungen von der Stage `03-stage-linux-voice-assistant`

Dieses Dokument beschreibt, was das Paket `picompose-software-ha-satellite` anders macht als die gleichnamige Stage im [PiCompose](https://github.com/florian-asche/PiCompose)-Repository. Verglichen wurde mit dem Stand von `main` (Commit `784abfb`, 17.11.2026). Gepflegt wird es auf Deutsch, auf Wunsch des Entwicklers.

Pflege: Jede Änderung, die eine Abweichung neu schafft, ändert oder beseitigt, aktualisiert dieses Dokument. Prüfe dabei gegen den aktuellen `main` von PiCompose.

## Nicht im Paket

- Der Compose-Manager. Das Paket hat vorläufig kein `Depends` auf `picompose-core`, der Compose-Manager muss anders installiert sein (siehe `todo.md`).
- Die Stage `04-stage-finish` (`.bashrc` und Aufräumen des Images) gehört nicht dazu.

## Anders gelöst

- **Kopieren der Projekte.** Die Stage legt `/compose/lva` und `/compose/snapcast` beim Bau des Images an und kopiert die Dateien hinein. Das Paket liefert sie nach `/usr/share/picompose/compose/<projekt>/`. Die `postinst` überschreibt `/compose/<projekt>/docker-compose.yml` bei jeder Installation und jedem Upgrade, kopiert `picompose.conf` und `.env` aber nur, wenn die Datei noch nicht existiert. Folge: Ein Upgrade aktualisiert die Compose-Dateien, ändert aber nie die Einstellungen des Nutzers.

## Unverändert

- `docker-compose.yml` und `picompose.conf` von `lva` und `snapcast` sind Byte für Byte die Dateien der Stage, mit den Rechten 644.

## Geändert

- **`SNAPCAST_REMOTE_HOST`.** Die `.env` von `snapcast` hat statt der Adresse aus dem Netz des Autors (`tcp://192.168.33.5`) den Platzhalter `tcp://<IP-OF-YOUR-MUSICASSISTANT-SERVER>`. Er muss vor dem Aktivieren des Projekts gesetzt werden.
- **`.env` von `lva`.** Sie entspricht der `.env.example` des linux-voice-assistant (Stand `ca2917a`) und nicht mehr der Datei der Stage. Neu sind unter anderem `MAC_ADDRESS`, `ENABLE_ENCRYPTION`, `ENCRYPTION_KEY_FILE`, `AUDIO_INPUT_BLOCK_SIZE`, die `PERIPHERAL_*`-Optionen, `DOWNLOAD_DIR` und weitere Sound-Dateien. `AUDIO_OUTPUT_DEVICE` und `AUDIO_INPUT_CHANNELS` haben neue Kommentare und Beispielwerte (`pulse`, `1`). Alle neuen Optionen sind auskommentiert, das Verhalten ändert sich also nicht. Eine vorhandene `/compose/lva/.env` wird nicht ersetzt.
