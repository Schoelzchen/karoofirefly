# Magicshine-Labor: Stand und Testplan

Übergabe für eine Claude-Code-Sitzung, die auf dem Mac läuft (Bluetooth, Webcam, Karoo per USB).
Ziel: die **Magicshine EVO 1700 UT** direkt per Bluetooth ansteuern, ihr Protokoll bestätigen und danach KarooFireFly (dieses Repository) dafür verbessern.

## Erkenntnisse aus der Recherche

Quellen: [lenne0815/karoo-magicshine-controls](https://github.com/lenne0815/karoo-magicshine-controls) (Messungen an einer EVO 1700 ohne UT, `MAGICSHINE_BLE_SNIFFER_NOTES.md`), [TimFourcode/evo1300](https://github.com/TimFourcode/evo1300), [JaJaJim/LightONKaroo](https://github.com/JaJaJim/LightONKaroo), AliOS-Quellcode (AIS-Dienst).

- **GATT:** Service `FFE1`, Characteristic `FFE0` (Write und Notify auf derselben Characteristic). Daneben `FFE2` (leer) und `FEB3` mit `FED4`–`FED8`.
  - `FEB3` ist der Alibaba IoT Service (AIS), ein Firmware-Update-Dienst.
  - **Nicht anfassen.**
  - Kein Standard-Akkudienst (`180F`).
- **Frame-Format:** `DE <Gesamtlänge> <Befehl> <Daten…> <XOR über Länge..letztes Datenbyte> ED`
- **Reine Abfragen** (ändern nichts):

  | Abfrage | Frame | Antwort / Inhalt |
  |---|---|---|
  | `A4` Akku | `DE06A400A2ED` | `B4`, Akku im Byte `data[8]` |
  | `A1` Info/Temperatur | `DE06A100A7ED` | `B1`, Temperatur nach Marker `17 03` |
  | `AD` Modellname | `DE06AD00ABED` | ASCII, z. B. `EVO_1700` |
  | `A5` Profile | `DE06A500A3ED` | |
  | `A9` Laufzeit | `DE06A900AFED` | EVO 1700 v1 antwortet nicht |
  | `AB` Speed-Sensor | `DE06AB00ADED` | |
  | `AC` Lichtsensor | `DE06AC00AAED` | |

- **Licht setzen (`A2`, 20 Byte):** `DE 14 A2 01 <Gruppe> <G0: an, Modus, Wert> <G1: an, Modus, Wert> 00×6 <Flag> <XOR> ED`
  - Modus: `01` = dauerhaft, `02` = SOS, `03` = Blitz. Wert = Helligkeit 0–100 (0x00–0x64).
  - Die EVO-Lampen haben zwei LED-Gruppen. Laut lenne0815/evo1300 ist G0 das Abblendlicht und G1 das Fernlicht. **Für die UT unbestätigt.**
  - Flag `BB` = „FLAG_SAVE“: Die Lampe speichert die Einstellung dauerhaft als Modus und verändert damit die Modi am Taster. Flag `00` (Prüfsumme neu berechnen) setzt das Licht nur vorübergehend (an der EVO 1300 bestätigt).
  - Bekannte Frames (alle mit `BB`):
    - Aus: `DE14A20101010100000000000000000000BB0DED`
    - Abblend 25 %: `DE14A20101010114000150000000000000BB48ED`
    - Fernlicht 25 %: `DE14A2010200010A010114000000000000BB11ED`
- **Akku:** EVO 1700 Firmware v1 meldet nur die Stufen 100 → 50 → 30 (komplette Entladung bei voller Leistung: 2:45 h). Deine UT zeigt in der Praxis 100 % und dann 20 %, also vermutlich ebenfalls nur Stufen.
- **Nur eine Bluetooth-Verbindung gleichzeitig:** Magicshine-App und Karoo-Extensions müssen getrennt sein.

## Schwächen in KarooFireFly (`app/src/main/kotlin/io/github/derstrassi/karoofirefly/ble/`)

1. `MagicshineProtocol.kt`: Jeder `A2`-Befehl wird mit `FLAG_SAVE` (`0xBB`) gesendet und überschreibt dadurch Lampenmodi.
2. `MagicshineDeviceConfig.buildCommand`: M2-Lampen werden nur über G1 angesteuert. Das Abblendlicht ist nicht erreichbar.
3. `MagicshineBleController.parseNotification`: Die Akku-Stufe wird als Prozent angezeigt.
4. Namensfilter: nur `M1-`/`M2-`/`M3-`. Der Name der UT ist unbekannt.

## Testplan auf dem Mac

Sicherheitsregeln:
- Lampe auf eine Wand richten, nicht ins Licht schauen. Max sitzt in der Nähe.
- Volle Leistung höchstens 30 s am Stück, Temperatur (`A1`) beobachten, ab 50 °C abbrechen.
- Zuerst nur Abfragen. `A2` zuerst **mit Flag `00`**, mit `BB` nur nach Rückfrage.
- Den Dienst `FEB3`/`FED4`–`FED8` nie beschreiben. Keine unbekannten Befehlscodes ausprobieren.

Schritte:
1. Python-Umgebung mit [`bleak`](https://pypi.org/project/bleak/) anlegen: `python3 -m venv .venv && .venv/bin/pip install bleak`.
2. Scannen: Name und UUIDs im Advertising notieren.
3. Verbinden: alle Services und Characteristics auflisten, dazu Device Information (`180A`) lesen, falls vorhanden.
4. Notify auf `FFE0` einschalten und die Abfragen `A4`, `A1`, `AD`, `A5`, `A9`, `AB`, `AC` senden. Alle Antworten roh protokollieren.
5. `A2` mit Flag `00`: G0 allein mit 10 / 25 / 50 %, dann G1 allein, dann beide, dann Aus. Nach jedem Schritt ein Webcam-Foto aufnehmen (`ffmpeg -f avfoundation -i "0" -frames:v 1 foto.jpg` oder `imagesnap`) und prüfen, welches Licht leuchtet.
6. Modi `02` (SOS) und `03` (Blitz) testen. Prüfen, ob sie ohne Wiederholung des Befehls bestehen bleiben.
7. Prüfen, ob nach den Tests mit Flag `00` die Modi am Lampentaster unverändert sind.
8. Ergebnisse in `docs/magicshine-evo1700ut-protokoll.md` festhalten.
9. Danach KarooFireFly anpassen (Punkte 1–4 oben, plus Laufzeit-Schätzer pro Modus), bauen, per `adb install` auf den Karoo bringen und mit `adb logcat` prüfen.
