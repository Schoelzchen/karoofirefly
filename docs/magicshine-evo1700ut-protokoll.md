# Magicshine EVO 1700 UT: gemessenes Bluetooth-Protokoll

Messung am 2026-11-02 mit einem Mac (`bleak` 3.0.2). Phase 1: nur lesen. Geschrieben wurden ausschließlich die Abfragen `A4`, `A1`, `AD`, `A5`, `A9`, `AB`, `AC` sowie das Einschalten von Notify.

## Gerät

| Feld | Wert |
|---|---|
| Bluetooth-Name | `M2-B0 EVO1700_UT` |
| Hersteller (`2A29`) | `Magicshine` |
| Modell (`2A24`) | `EVO1700 UT` |
| Seriennummer (`2A25`) | 16 Hex-Zeichen (hier nicht veröffentlicht) |
| Firmware (`2A26`) | `1.0.0` |
| Hardware (`2A27`) | `1.0.0` |
| Software (`2A28`) | `1.0.1` |

## GATT-Datenbank

| Service | Characteristic | Eigenschaften | Gelesener Wert | Bedeutung |
|---|---|---|---|---|
| `180A` Device Information | `2A29`, `2A24`, `2A25`, `2A27`, `2A26`, `2A28`, `2A23`, `2A2A`, `2A50` | read | siehe oben, Rest Nullen | Geräteinformationen |
| `180F` Battery Service | `2A19` | read, notify | `64` (100 %) | **Standard-Akkustand.** Den hat die EVO 1700 ohne UT nicht. Auflösung noch offen |
| `FFE1` (Magicshine) | `FFE0` | read, write, notify | `DE13B400020800000A…A7ED` | Steuer- und Abfragekanal (bekannt) |
| | `FFE2` | read | `010001` | unbekannt |
| | `FFE3` | read, notify | `0064` | unbekannt (100?) |
| | `FFE4` | write | – | unbekannt, **nicht beschreiben** |
| | `FFE5` | read, indicate | leer | unbekannt |
| | `FFE6` | write | – | unbekannt, **nicht beschreiben** |
| | `FFE7` | read, indicate | `027100` | unbekannt |
| `ADB425D4-B1C6-11ED-AFA1-0242AC120002` | `8CE5DD01-0A4D-11E9-AB14-D663BD873D93` | read | `0200818002000EFF6400` | „Light Profile“, das evo1300 bei der HORI 1300 zur Fernlicht-Steuerung nutzt |
| | `8CE5DD02-…` | read, notify | `2F00010F00FF` | |
| | `8CE5DD03-…` | write, indicate | – | Steuerkanal, **nicht beschreiben** (vorerst) |
| | `8CE5DD04-…` | write, indicate | – | **nicht beschreiben** (vorerst) |
| `42820001-D3E8-480B-95BB-2A34850D4048` | `42820002`–`42820008` | read | ASCII: `33653`, `000000000000`, `600200003B77`, … | unbekannt, evtl. Di2-/Shimano-Anbindung (unbestätigt) |
| `FE59` Nordic Secure DFU | `8EC90003-F315-4F60-9FB8-838830DAEA50` | write, indicate | – | **Firmware-Update (Buttonless DFU). Niemals beschreiben** |

Folgerung: Die UT basiert auf einem **Nordic-Chip** (passt zu ANT+) und nutzt den Nordic-Secure-DFU-Bootloader, nicht den Alibaba-AIS-Dienst der EVO 1700. Secure DFU prüft Updates in der Regel mit einer ECDSA-Signatur des Herstellers. Eigene oder veränderte Firmware per Funk ist damit praktisch ausgeschlossen.

## Antworten auf die Abfragen

| TX | RX | Inhalt |
|---|---|---|
| `DE06A400A2ED` (A4) | `DE13B40004000000640000000000000000C7ED` | Daten `04 00 00 00 64 …`, Akku = `0x64` = 100 (EVO 1700: erstes Datenbyte `00`) |
| `DE06A100A7ED` (A1) | `DE0DB100001118071B0103ABED` | Daten `00 11 18 07 1B 01 03` |
| `DE06AD00ABED` (AD) | `DE13BD0045564F313730305F5554000000AAED` | ASCII `EVO1700_UT` |
| `DE06A500A3ED` (A5) | – | keine Antwort |
| `DE06A900AFED` (A9) | – | keine Antwort |
| `DE06AB00ADED` (AB) | `DE08BB0000FF4CED` | wie EVO 1700 |
| `DE06AC00AAED` (AC) | `DE08BC0000FF4BED` | wie EVO 1700 |

Alle Antworten haben gültige Länge, `ED`-Ende und XOR-Prüfsumme.

### Temperatur (A1 → B1)

EVO 1700 (lenne0815): Daten `00 11 17 03 17 01 01`. Der Wert an Index 4 (`0x17`) stieg während der Entladung bis 61 °C.
EVO 1700 UT: Daten `00 11 18 07 1B 01 03`. An Index 4 steht `0x1B` = 27 °C, was zur Raumtemperatur passt.

Der Marker `17 03`, den KarooFireFly zum Finden der Temperatur nutzt, kommt bei der UT nicht vor. Die Temperatur steht vermutlich **fest an Daten-Index 4**. Das muss noch bestätigt werden, indem man die Lampe kurz warmlaufen lässt.

### Beobachtung beim Lesen von FFE0

Direktes Lesen lieferte ein `B4`-Frame `DE13B400020800000A0000000000000000A7ED` (Daten `02 08 00 00 0A …`). Das unterscheidet sich von der A4-Antwort und ist wahrscheinlich ein alter Puffer. Die Bedeutung ist offen.

## Konsequenzen für KarooFireFly

- Namensfilter `M2-` passt, die Lampe wird erkannt und als zwei Lampengruppen (M2) behandelt.
- Temperatur-Parser: der Marker `17 03` passt nicht, auf Index 4 umstellen.
- Akku zusätzlich über den Standarddienst `180F`/`2A19` per Notify lesen und vergleichen, ob er feiner auflöst als `A4`.
