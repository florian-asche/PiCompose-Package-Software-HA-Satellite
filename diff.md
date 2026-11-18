# Abweichungen von der Stage `03-stage-linux-voice-assistant`

Dieses Dokument beschreibt, was das Paket `picompose-software-ha-satellite` anders macht als die gleichnamige Stage im [PiCompose](https://github.com/florian-asche/PiCompose)-Repository. Verglichen wurde mit dem Stand von `main` (Commit `784abfb`, 17.11.2026). Gepflegt wird es auf Deutsch, auf Wunsch des Entwicklers.

Pflege: Jede Änderung, die eine Abweichung neu schafft, ändert oder beseitigt, aktualisiert dieses Dokument. Prüfe dabei gegen den aktuellen `main` von PiCompose.

## Nicht im Paket

- Die Stage `04-stage-finish` (`.bashrc` und Aufräumen des Images) gehört nicht dazu.

## Anders gelöst

- **Kopieren der Projekte.** Die Stage legt `/compose/lva` und `/compose/snapcast` beim Bau des Images an und kopiert die Dateien hinein. Das Paket liefert sie nach `/usr/share/picompose/compose/<projekt>/` und kopiert ein Projekt in der `postinst` nur, wenn `/compose/<projekt>` noch nicht existiert. Folge: Ein Upgrade des Pakets ändert ein vorhandenes Projekt nie, auch nicht die `docker-compose.yml`.

## Zusätzlich im Paket

- `Depends` auf `picompose-core`, das den Compose-Manager liefert, der die Projekte startet.

## Unverändert

- `docker-compose.yml`, `picompose.conf` und `.env` von `lva` und `snapcast` sind Byte für Byte die Dateien der Stage, mit den Rechten 644.
