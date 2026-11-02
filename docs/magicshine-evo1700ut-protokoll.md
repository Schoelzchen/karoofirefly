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

## Phase 5: Steuerung über den Light Control Point (2026-11-02, erfolgreich)

Automatischer Test vom Mac mit Auswertung über die Webcam. Es wurden keine A2-Frames gesendet. **Taster und Modi waren danach unverändert.**

| Befehl auf `8CE5DD03` | Indication | Wirkung | Status `8CE5DD02` danach |
|---|---|---|---|
| `00` | `80 00 01` (ca. 60–120 ms) | Control erhalten | – |
| `03 00` | `80 03 01` | **Licht aus** (Wand auf Dunkel-Niveau) | `2F 00 01 00 00 xx` |
| `03 0F` | `80 03 01` | **Licht an** | `2F 00 01 0F 00 xx` |
| `04 01` | `80 04 01` | Fernlicht (Status `…0F 01…`, Wand heller) | `2F 00 01 0F 01 19` |
| `04 00` | `80 04 01` | Abblendlicht | `2F 00 01 0F 00 19` |
| `06 0A` / `06 19` / `06 32` | `80 06 01` | Intensität 10 / 25 / 50 % | Byte 5 = Intensität |
| `03 11` | `80 03 01` | SOS | Modus `11` |
| `03 07` | **keine** | keine Wirkung, obwohl in den Features gemeldet | – |

Weitere Ergebnisse:
- **Reaktionszeit:** Die Indication kommt nach 40–130 ms.
- **Verhalten nach dem Trennen:** Die Lampe bleibt im gesetzten Zustand (nach `03 00` aus), ist sofort wieder sichtbar und verbindbar. Nach erneutem `00` schaltet `03 0F` sie wieder ein, mit der zuletzt gesetzten Intensität.
- **Max. Temperatur:** 35 °C.
- **Gleichzeitige Nutzung mit der Magicshine-App:** Solange die App die Einstellungsseite der Lampe zeigt, ist die Lampe für andere Clients nicht erreichbar. In der Übersicht (Akku und Temperatur) klappt die gleichzeitige Verbindung.
- **Noch offen:** Befehle ohne vorheriges `00`, Intensität über 50 % und mit Fernlicht, Tastendruck während laufender BLE-Steuerung, Bedeutung von `01` (Reset).

### Folgerung für KarooFireFly
Die UT sollte über den Light Control Point gesteuert werden, nicht über `A2`:
- Verbinden, Indications auf `8CE5DD03` und Notify auf `8CE5DD02` einschalten, `00` senden.
- Danach `03 00`/`03 0F` für Aus/An, `04 00`/`04 01` für Abblend/Fernlicht, `06 <%>` für die Helligkeit, `03 11` für SOS.
- Den Status aus `8CE5DD02` als Rückmeldung verwenden.
- Akku aus `2A19`, Temperatur aus `A1` (Frame-Byte 8).

## Phase 6: Bedeutung des Platz-Bytes in A2 (2026-11-02)

Ergebnis: Das Byte nach `A2 01` ist ein **Platz für eigene Modi**, nicht die Wahl zwischen Abblend- und Fernlicht. Welche Lichtgruppe betroffen ist, ergibt sich aus dem Inhalt des Frames (welche Gruppe „an“ ist). Am Ende hatte die Lampe wieder je drei Modi, ohne App-Reparatur. Die Temperatur lag bei höchstens 38 °C.

Acks: `DE07B60001B0ED` (Ergebnis `01`) und `DE07B60000B1ED` (Ergebnis `00`).

| Nr | Frame | Ack | Beobachtung am Taster | Befund |
|---|---|---|---|---|
| 1 | Slot 01, Abblend 10 %, BB | 01 | – | springt auf 10 % |
| 2 | Slot 02, Abblend 50 %, BB | 01 | Abblend 2 Modi, Fernlicht 3 | zwei eigene Modi in derselben Gruppe |
| 4 | Slot 02, Abblend 50 %, AA | 00 | Abblend nur noch 10 % | Slot 2 trotz Ack `00` gelöscht |
| 6 | Slot 01, Abblend 10 %, AA | 01 | je drei | Standardmodi zurück |
| 8 | Slot 01, Fernlicht 25 %, BB | 01 | – | |
| 9 | Slot 02, Abblend 25 %, BB | 01 | Abblend 1, Fernlicht 1 | Gruppe folgt dem Inhalt, nicht dem Slot |
| 11 | Slot 01, Fernlicht 25 %, AA | 01 | – | Slot 1 gelöscht |
| 12 | Slot 02, Abblend 25 %, AA | 00 | Abblend 1, Fernlicht 3 | keine Wirkung |
| 14 | Slot 03, Abblend 10 %, BB | 00 | – | kein Sprung, vermutlich abgelehnt |
| 15 | Slot 03, Abblend 50 %, AA | 00 | – | keine Wirkung |
| 17 | Slot 01, Abblend 10 %, BB | 01 | – | springt auf 10 % |
| 18 | `8CE5DD03`: `00`, `03 00` | `800001`, `800301` | – | **aus**, auch mit aktivem eigenem Modus |
| 19 | `03 0F` | `800301` | – | an, mit 10 % (Intensität des eigenen Modus) |
| 20 | Slot 01, Abblend 10 %, AA | 01 | je drei | Endzustand normal |

Deutung:
- Max' Hypothese (`01` = Abblend, `02` = Fernlicht) trifft nicht zu.
- Die Allty-Hypothese (Slot-Nummer, `BB` anlegen, `AA` löschen) passt für die Slots 1 und 2.
- **Wahrscheinlich rücken die Plätze nach dem Löschen nach.** Nach dem Löschen von Slot 1 lag der verbliebene Abblend-Modus vermutlich auf Slot 1. Deshalb blieb `AA` auf Slot 2 (Nr. 12) wirkungslos, und `BB` auf Slot 3 (Nr. 14) hätte eine Lücke gelassen. Unbestätigt.
- Ob `AA` nach Slot oder nach Inhalt löscht, ist offen. Slot 3 wurde nie angelegt.
- Die Phase-4-Hypothese „`AA` stellt die Gruppe wieder her, die im Frame an ist“ ist überholt. `AA` wirkt auf den Slot. Das „Fernlicht nur noch ein Modus“ aus Phase 4 stammt vermutlich von Nr. 14 dort (Slot 02, Fernlicht, BB), der nie gelöscht wurde.
- Fehlt noch: eine Beschreibung der Taster-Gesten der Lampe.

Die englische Referenz mit allen Ergebnissen steht in [`magicshine-evo1700ut-ble-reference.md`](magicshine-evo1700ut-ble-reference.md).

## Phase 7: Kapazität und Nachrücken der Slots (2026-11-02)

Die Modi am Taster wurden per Webcam gezählt. Max hat dafür den Taster gedrückt, der Mac hat nach jedem Druck die Wandhelligkeit gemessen. Die Temperatur lag bei höchstens 46 °C (helle Fernlicht-Stufen am Taster). Am Ende war die Lampe im Werkszustand, ohne App-Reparatur.

| Frage | Ergebnis |
|---|---|
| Kapazität | Slots 1–6 angenommen (Ack `01`, Sprung). Am Taster 6 Modi in Slot-Reihenfolge (5/15/25/35/45/50 %). Mehr nicht getestet |
| Nachrücken | **bestätigt.** Nach `AA` auf Slot 1 war Slot 6 leer: `AA` auf Slot 6 brachte Ack `00` und keine Wirkung, die 50 % blieben |
| Löschen nach Slot oder Inhalt | **nach Slot.** `AA` auf Slot 1 mit falschem Inhalt löschte trotzdem den Modus auf Slot 1 |
| Gemeinsame Slots | **ja.** Fernlicht 25 % auf Slot 1 ersetzte den Abblend-Modus dort. Danach je ein Modus pro Licht |
| Aufräumen | wiederholt `AA` auf Slot 1, bis nichts mehr übrig ist |

Standardmodi (Wandhelligkeit, nur grob vergleichbar): Abblend ca. 100 / 127 / 140, Fernlicht ca. 92 / 130 / 156.

Hinweis: Ein BLE-Befehl, der einem kurzen Tastendruck entspricht („nächste Stufe“), ist auf den Verbindungskanälen nicht bekannt. Die Fernbedienung MJ-6558 sendet ihre Tasten per Advertising. Das kann macOS nicht nachahmen (kein eigenes Service-Data-Advertising), ein Android-Gerät wie der Karoo grundsätzlich schon. Dafür fehlt aber ein Mitschnitt der echten Fernbedienung.

## Phase 8: Speichert die App die eigenen Modi selbst? (2026-11-02)

Ergebnis: **Die App speichert eigene Modi lokal und liest sie nicht aus der Lampe. Beim Verbinden schreibt sie ihren gespeicherten Modus von selbst zurück.** Mitschnitt per PacketLogger, fünf Verbindungen.

| Test | Ergebnis |
|---|---|
| Mac legt Abblend 15 % und Fernlicht 35 % an, dann öffnet Max die App | Die App zeigt keinen dieser Modi. Danach findet der Mac genau 2 Modi, die App hat nichts gelöscht |
| Max legt in der App 45 % an und speichert | Vorschau `BB` Slot 1 beim Verschieben, **„Speichern“ sendet nichts** |
| Mac löscht den Modus aus der Lampe, Max öffnet die App erneut | Die App zeigt 45 % weiterhin an und **sendet beim Verbinden von selbst** `DE14A2010101012C000000000000000000BB21ED` (Slot 1, 44 %). Die Lampe hat den Modus wieder |
| Was die App liest | nur `FFE0` (letzter Frame, z. B. `A6`/`BD`/`B1`/`B6`) und `2A19`. Kein `A5`, kein Light Service |
| Was die App beim Verbinden sendet | `A6 01`, `A1`, `A4`, `AB`, `AC`, `AD`, dazu bei gespeicherten Modi `BB` |

Folgerungen:
- Ein eigener Modus ist erst dann dauerhaft weg, wenn er in der App gelöscht wird. Ein `AA` von einem anderen Gerät hält nur bis zum nächsten Verbinden der App.
- Das erklärt sehr wahrscheinlich die „plötzlich fehlenden“ Modi vor Phase 2 und vor Phase 5: Die App war zwischendurch verbunden und hat ihren gespeicherten Modus zurückgeschrieben.
- Weitere Optionen in der App laut Screenshots: „Light Sensor“, „Motion Detection“, „Light Group1/2“, „Add New Mode“ (Eintrag „Always On“).
- Offen: Welche Slots nutzt die App bei mehreren gespeicherten Modi? Sendet „Zurück ohne Speichern“ ein `AA`? Das wurde in diesem Lauf nicht ausgelöst.

## Phase 9: Slots der App, „Zurück ohne Speichern“, Löschen (2026-11-02)

| Frage | Ergebnis |
|---|---|
| Slots bei zwei gespeicherten Modi | Vergabe in der Reihenfolge des Anlegens: Abblend 20 % → Slot 1, Fernlicht 40 % → Slot 2. Beim Verbinden schreibt die App **beide** zurück (Slot 1, dann Slot 2, ca. 60 ms Abstand) |
| Zurück ohne Speichern, bestehender Modus (geändert oder unverändert) | **kein `AA`**, nur `BB` mit dem gespeicherten Wert |
| Zurück ohne Speichern, neuer Modus | `AA` auf den neuen Slot (hier 3), danach `BB` auf den vorherigen Modus |
| Löschen in der App (nach links wischen, Mülleimer) | `AA` auf den Slot des Eintrags, danach schreibt die App die verbleibenden Modi neu auf die nachgerückten Slots (Fernlicht von Slot 2 → Slot 1) |
| Speichern | sendet nichts |

Weitere Befunde:
- **Ack `00` heißt nicht zuverlässig „fehlgeschlagen“.** `AA` auf Slot 3 bekam `00` und wirkte trotzdem, genau wie in Phase 6, Schritt 4.
- Der Regler sendet etwa 1 % unter dem angezeigten Wert (20 → `13`, 30 → `1D`, 40 → `27`, 50 → `32`).
- Beim Bearbeiten sendet die App die Werte der anderen Gruppe als „aus“ mit (z. B. `00 01 27`). Beim Öffnen eines Modus sendet sie ihn zweimal.
- Taster: Mit beiden App-Modi hatte jedes Licht genau einen Modus. Nach dem Löschen in der App wieder je drei.
- Die Lampe war am Ende im Werkszustand. App und Mac waren nie gleichzeitig verbunden.

## Phase 10: Xingzhe gegen App, Status bei Tastendruck, Zählen eigener Modi (2026-11-02)

**Teil A, Recherche:** Die vollständige Opcode-Liste der Spezifikation steht in der englischen Referenz (Abschnitt 3.3). Über die App hinaus bestätigt sind:
- Aus und Ein
- Abblend/Fernlicht und Helligkeit live, ohne eigene Modi
- SOS live
- Status-Meldungen

Laut Spezifikation gibt es außerdem Auto-Sleep, Sleep-Zeit, Aufwecken durch Erschütterung, Bremslicht, Lichtsensor, Animationen, Navigation, Fernbedienungs-Abfrage sowie Datum und Uhrzeit (Opcodes `10`–`1A`, `24`, `DA`). Auf der UT ist davon noch nichts getestet. Die meisten haben eine Abfrage-Variante mit `FF`.

**Teil B, Ergebnisse:**
- **Jeder Tastendruck erzeugt eine Status-Meldung** auf `8CE5DD02` mit Modus, Licht und Helligkeit, auch bei eigenen Modi.
- **Werksmodi laut Status:** Abblend und Fernlicht je 25 / 50 / 100 %. Beim Doppelklick kommen vor dem Wechsel Zwischenwerte.
- **Eigene Modi, wie gemeldet:** Abblend 10 % (`…0F000A`), Abblend-SOS 20 % (`…110014`), Fernlicht 30 % (`…0F011E`). Am Taster ergab das Abblend 2 Modi, Fernlicht 1.
- **Auslesen:** Nur der aktive Modus ist direkt lesbar. Die ganze Liste erhält man nur durch Durchschalten am Taster mit Mitlesen des Status.
- **Zählen über abgelehnte `BB`:** Slot 8–5 wurden abgelehnt (Ack `00`, keine Meldung), Slot 4 angenommen. Damit sind 3 Modi gezählt, das Aufräumen hat es bestätigt.
  - **Nebenwirkung:** Nach `AA` auf Slot 4 reagierte der Taster nicht mehr. Erst nach dem vollständigen Aufräumen und `03 0F` ging er wieder. Die Ursache ist offen; in Phase 6 lief eine ähnliche Lage ohne Blockade.
- Der erste Status nach dem Verbinden war veraltet (`2F00010000FF` bei leuchtender Lampe).
- Endzustand: je drei Modi, Lampe aus, höchstens 39 °C, App die ganze Zeit geschlossen.

Nachtrag (Max): In der Magicshine-App sind die Schalter „Light Sensor“ und „Motion Detection“ bei der UT sichtbar, aber ausgegraut und nicht wählbar. Die UT hat vermutlich keinen Lichtsensor. Die Magicshine-Befehle `AF`/`AE` werden deshalb nicht weiter untersucht.

## Phase 11: Abfrage der Functions im Xingzhe-Protokoll (2026-11-02)

Gesendet wurden nur Abfragen mit `FF` auf `8CE5DD03`: `10`, `11`, `12`, `13`, `14`, `15`, `16`, `17`, `18`, `19`, `1A`, `24`. Es gab zwei Runden, einmal mit ausgeschalteter und einmal mit eingeschalteter Lampe.

Ergebnis: **Keine einzige Antwort**, keine Status-Meldung, keine sichtbare Änderung. `00` wurde normal beantwortet. Taster und Modi waren danach normal.

Folgerungen:
- Die UT unterstützt Auto-Sleep, Aufwecken, Sleep-Zeit, Bremslicht, Lichtsensor, Animation, Navigation und die Fernbedienungs-Abfrage über dieses Protokoll nicht. Unbekannte Befehle ignoriert sie still, statt wie in der Spezifikation vorgesehen mit `80 <Op> 02` zu antworten. Genauso verhält sie sich bei `03 07`.
- Die Fernbedienungs-Kennung lässt sich so nicht auslesen.
- Korrektur zu Phase 10: Der Status `2F00010000FF` direkt nach dem Verbinden war nicht veraltet, sondern bedeutet „aus“. Die Lampe war in Phase 10 zu Beginn vermutlich aus.

## Phase 12: Funkprotokoll der Fernbedienung MJ-6558 (2026-11-02)

Rein passiver Mitschnitt mit Duplikaten (pyobjc/CoreBluetooth). Gleichzeitig war der Mac mit der Lampe verbunden, nur lesend (`00`, Status-Notify).

- **Die MJ-6558 (`M0-C0 MJ_6558`) ist ein BLE-Advertiser.** „FTR LightSync“ ist also Bluetooth-Rundfunk.
- Format der Service Data auf `FFE1`, alle 442 Pakete 7 Byte lang: `18 01 06 <Zähler> 02 02 <Code>`.
  - Der Zähler steigt pro Ereignis um 1.
  - Codes ○: `21` kurz, `22` doppelt, `23` lang. Codes □: `11` kurz, `12` doppelt, `13` lang. `00` kurz vor dem Code (Taste gedrückt).
  - Pro Ereignis eine Salve von 3–17 Paketen, zwischen den Drücken kommt nichts.
- Wirkung laut Lampenstatus:
  - ○ kurz: nächste Stufe (25 → 50 → 100 %)
  - ○ lang: an/aus
  - ○ doppelt: Fernlicht
  - □ kurz: Fernlicht umschalten
  - □ doppelt und □ lang: keine Statusänderung
- **Keine Geräte-ID im Inhalt.** Die Lampe erkennt ihre Fernbedienung vermutlich an der Funkadresse. Ob sie jedes passende Paket annimmt, ist offen.
- **Mit gehaltener Mac-Verbindung ließ sich die ausgeschaltete Lampe per Fernbedienung nicht einschalten** (mehrere lange Drücke). Ohne Verbindung ging es sofort, getestet einmal. Direkt nach dem Ausschalten klappte es auch mit Verbindung.
- **Die Lampe advertist nicht, solange ein Client verbunden ist.**
- Bei abgeschirmter Fernbedienung kam das Paket am Mac an, die Lampe reagierte aber nicht (Reichweite).
- Die GATT-Datenbank der Fernbedienung (nur gelesen) enthält `FFE1` (`FFE0`–`FFE7`) und den Alibaba-Update-Dienst `FEB3` (`FED4`–`FED8`). Die Werte waren leer. Die Kopplung blieb danach intakt.
- Ein Nachbau vom Karoo (Android) wäre inhaltlich einfach. Unklar ist, ob die Lampe die Adresse prüft. Nichts gesendet.

## Phase 13: Emulation der Fernbedienung vom Mac (2026-11-02)

Ohne Installation, nur mit pyobjc/CoreBluetooth. Der Mac versuchte, den Rundruf der MJ-6558 (○ kurz, `21`) über den undokumentierten Schlüssel `"kCBAdvDataServiceData"` zu senden: V1 Key `FFE1`, V2 volle UUID, V3 zusätzlich mit Name, V4 CBUUID-Key. Je Variante zuerst `00` mit Zähler N, dann der Code mit N+1. Runde 1 lief mit der echten Fernbedienung im Nebenraum, Runde 2 mit ihr neben der Lampe.

| Ergebnis | Beleg |
|---|---|
| **macOS sendet die Service Data nicht** | `didStartAdvertising` meldet jedes Mal „OK“. Ein iPhone in der Nähe (Max' Idee, iOS-Trace) empfing aber keine `FFE1`-Daten vom Mac. Mit Name kamen nur Flags, TX Power und Name an. Die echte MJ-6558 wurde in derselben Messung zuverlässig empfangen |
| V4 (CBUUID-Key) | Absturz `-[CBUUID UTF8String]` |
| Lampe | keine Reaktion in beiden Runden. Logisch, das Paket kam nie an |
| **Adresse der Fernbedienung** | `60:02:00:00:3B:77` (iPhone-Trace). **Die Lampe meldet in `42820004` genau `600200003B77`.** Sie speichert also die Adresse der gekoppelten Fernbedienung und filtert vermutlich danach |

Folgerungen:
- Vom Mac lässt sich die Fernbedienung nicht nachahmen.
- Wegen des gespeicherten Adress-Filters ist auch ein Nachbau vom Karoo sehr unwahrscheinlich, weil Android keine fremde, feste Adresse annehmen kann.
- Passives Mitlesen der Fernbedienung, z. B. als Eingabe für eine Karoo-Extension, funktioniert dagegen.
- Die echte MJ-6558 funktionierte danach normal.
- Offen: die Beschreibungen (`2901`) der Characteristics im Service `42820001`, die Bedeutung von `42820002` = `33653` und ob `42820003`/`42820005` freie Kopplungsplätze sind.
