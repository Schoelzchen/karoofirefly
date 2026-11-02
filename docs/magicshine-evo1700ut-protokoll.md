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

## Firmware-Stand

- Die Magicshine-App hat am 2026-11-01 beim ersten Verbinden ein Firmware-Update angeboten, und Max hat es installiert. Alle Messungen hier beziehen sich auf den Stand **nach** diesem Update.
- Bluetooth Device Information: Firmware `1.0.0`, Hardware `1.0.0`, Software `1.0.1`.
- ANT+-Produktinfo laut Karoo: Modellnummer `522`, Software `0.101`, Hardware `1`, Seriennummer `831635…`.

## Phase 2: eigene A2-Frames mit Flag `00` (2026-11-02)

Gesendet wurden drei Frames mit Flag `00`:
- `DE14A2010101010000000000000000000000B6ED` (zweimal)
- `DE14A2010101010A00015000000000000000EDED`

Die Lampe hat **keinen** davon bestätigt (kein `B6`-Ack) und das Licht nicht verändert. Sie verwirft Frames mit Flag `00` offenbar.

Dass die Modi am Taster danach verändert waren (Abblendlicht nur noch ein schwacher Modus), stammt deshalb sehr wahrscheinlich **nicht** von diesen Frames. Vermutlich war die Lampe noch von einem früheren Test (LightONKaroo bzw. App) mit Flag `BB` in einem „Live-Zustand“, siehe unten. Die Modi waren vor Phase 2 nicht geprüft worden.

## Phase 3: Mitschnitt der offiziellen Magicshine-App (iPhone, PacketLogger)

Die App schreibt Lichtbefehle **nur** auf `FFE0`. Die Kanäle `FFE4`, `FFE6`, `8CE5DD03/04` nutzt sie nie.

### Beim Verbinden
1. Indications bzw. Notify einschalten, `FFE0` und `2A19` lesen.
2. `DE07A601EF4FED` (`A6 01`), Bedeutung offen, keine Antwort.
3. `A1` und `A4`, danach alle 20 s erneut.

### A2-Format (UT, laut App)

`DE 14 A2 01 01 <G0: an, Muster, Wert> <G1: an, Muster, Wert> 00 00 00 00 00 00 <Flag> <XOR> ED`

- G0 = Abblendlicht, G1 = Fernlicht. Die App schaltet immer genau eine Gruppe an. Laut App „beinhaltet“ das Fernlicht das Abblendlicht.
- Muster: `01` dauerhaft, `02` SOS, `03` Flash.
- Wert: Prozent als Byte (z. B. `0x18` = 24 %, `0x32` = 50 %).
- Das Byte nach `A2 01` ist immer `01`. KarooFireFly sendet hier `02`; ob die UT das akzeptiert, ist offen.
- **Flag `BB`:** Licht wird sofort live gesetzt. Antwort `DE07B60001B0ED`.
- **Flag `AA`:** beendet den Live-Zustand, die Lampe kehrt zu ihren eigenen Modi zurück. Die App sendet das beim Löschen eines Eintrags und beim Verlassen ohne Speichern. Danach hatte der Taster wieder je drei Modi. Antwort ebenfalls `B6`-Ack.
- **Flag `00`:** von der App nie gesendet, von der Lampe ignoriert.
- „Speichern“ in der App sendet nichts an die Lampe. Die eigenen Modi liegen also in der App, nicht in der Lampe.

Beispiele aus dem Mitschnitt:

| Zweck | Frame |
|---|---|
| Abblend 10 % | `DE14A2010101010A000000000000000000BB07ED` |
| Abblend 24 % | `DE14A20101010118000000000000000000BB15ED` |
| Abblend 50 % | `DE14A20101010132000000000000000000BB3FED` |
| Fernlicht 24 % | `DE14A20101000118010118000000000000BB0CED` |
| Fernlicht 50 % | `DE14A20101000118010132000000000000BB26ED` |
| Abblend Flash 24 % | `DE14A20101010318000118000000000000BB0EED` |
| Abblend SOS 24 % | `DE14A20101010218000118000000000000BB0FED` |
| Fernlicht Flash 24 % | `DE14A20101000218010318000000000000BB0DED` |
| Live-Zustand beenden | `DE14A20101010118000000000000000000AA04ED` |

### Temperatur
Bestätigt: Frame-Byte 8 der `B1`-Antwort (Daten-Index 4). Bei eingeschaltetem Licht stieg der Wert von 28 auf 33 °C, alle anderen Bytes blieben gleich.

### Offen
- Wie die Lampe per BLE ausgeht. Die App hat dafür keine Funktion.
- Ob ein `BB`-Zustand einen Neustart der Lampe übersteht.
- Bedeutung von `A6 01`.
- Auflösung des Akkus über `2A19` bei einer Entladung.

## Konsequenzen für KarooFireFly

- Frames auf das App-Format umstellen: Byte `01` nach `A2 01`, G0 = Abblend, G1 = Fernlicht.
- Beim Freigeben der Lampe (Fahrtende, Extension inaktiv) **`AA` senden**, damit der Taster wieder normal funktioniert. Ohne `AA` bleibt die Lampe im zuletzt gesetzten Live-Zustand. Das erklärt sehr wahrscheinlich die „veränderten Modi“.
- Temperatur aus Frame-Byte 8 lesen.
- Akku zusätzlich aus `2A19` lesen.

## Xingzhe „Bluetooth Light Profile“ (Service `ADB425D4-…`)

Die Characteristics `8CE5DD01`–`8CE5DD04` gehören zum öffentlich dokumentierten Light Profile des Radcomputer-Herstellers Xingzhe: <https://developer.imxingzhe.com/docs/device/light/> (Version 1.6.0). Kurzfassung der hier relevanten Teile:

| Characteristic | Name | Eigenschaften |
|---|---|---|
| `8CE5DD01` | Light Features | Read |
| `8CE5DD02` | Light Machine Status | Read, Notify |
| `8CE5DD03` | Light Control Point | Write (mit Response), Indicate |
| `8CE5DD04` | Light Network | Write, Indicate (laut Spezifikation nicht implementiert) |

Light Control Point: Format `<Opcode> <Parameter…>`, Antwort als Indication `80 <Opcode> <Result>` (`01` Erfolg, `02` nicht unterstützt, `03` ungültiger Parameter, `04` Fehler, `05` keine Berechtigung).

| Opcode | Bedeutung |
|---|---|
| `00` | Request Control (zuerst senden) |
| `01` | Reset der Gerätesteuerung (Umfang unklar, noch nicht getestet) |
| `03 <Modus>` | Lichtmodus: `00` aus, `07` schnelles Blinken, `0F` dauerhaft an, `11` SOS |
| `04 00` / `04 01` | Abblendlicht / Fernlicht |
| `06 <0–100>` | Intensität in % |
| `DF` | DFU-Modus. **Nie senden.** |

Dekodierung der Werte der UT:
- `8CE5DD01` = `0200818002000EFF6400`: Frontlicht. Modi aus (Bit 0), schnelles Blinken (7), dauerhaft an (15) und SOS (17). Abblend- und Fernlicht mit 3 Modi, Intensität bis 100 %, einfarbig. Das Functions-Feld fehlt (10 statt 20 Byte).
- `8CE5DD02` = `2F00010F00FF`: Index 0, Frontlicht, Modus 15 (dauerhaft an), Abblendlicht, Intensität `FF` (nicht gemeldet).

**Die UT meldet „Licht aus“ als unterstützten Modus.** Das evo1300-Projekt nutzt an der HORI 1300 bereits `00` und `04 01`/`04 00` auf diesem Kanal.
