
---

# LEGO Kamera Dolly Projekt

Der Kameradolly besteht aus einer Hoizontalen und Vertikalen Drehachse, sowie einem Chassis zum fahren.

> Hier Foto einfügen

Das 'Gimble' (bestehend aus der horizontalen und vertikalen Rotationsachse) wird modular auf das Chassis gesetzt und kann um jeweils 90° rotiert angebaut werden.

> Hier Foto einfügen

## Vertikale und horizontale Rotationsachsen

Die Untersetzung für sowohl die **vertikale** als auch die **horizontale Rotationsachse** beträgt 1:960. Dabei sind an der horizontalen Achse zwei Motoren verbaut, während die vertikale Achse von nur einem Motor angetrieben wird.

> Hier Fotos einfügen

## Chassis mit Panzersteuerung (Skid-Steering)

Das Chassis besteht aus vier Gearboxen mit einer Untersetzung von 1:256. Jede Gearbox verfügt über einen eigenen Motor. Dadurch kann das Fahrzeug ähnlich wie ein Panzer manövrieren, agiert jedoch effizienter: Kurven werden intelligent durch Geschwindigkeitsunterschiede zwischen dem Innen- und dem Außenrad gefahren. Dieses Verhalten simuliert Funktionen einer Differenzial-Lenkung oder der Ackermann-Lenkung.

> Hier Fotos einfügen

Alle sieben Antriebe, sowohl für das Chassis als auch für die Drehachsen der Kamera, verwenden einheitlich den Lego 8883 Motor. Bei der Motorenwahl überwog der Kosten-Nutzen-Faktor: Ein Motor mit Encoder kostet aktuell rund 21 €, während der Lego 8883 als Import bereits für 2,60 € erhältlich ist. Da alle sieben benötigten Motoren bereits vorhanden waren, wurden die ersten Prototypen damit realisiert. Dabei stellte sich heraus, dass die erreichte Präzision – selbst für Lego-Verhältnisse – absolut ausreichend ist.

## Spezifikationen des Lego Motors 8883

* **Betriebsspannung:** 9V
* **Leerlaufdrehzahl:** ca. 405 U/min
* **Max. Drehmoment:** ca. 11 Ncm (Blockierdrehmoment)
* **Stromaufnahme (Leerlauf):** ca. 40 mA
* **Stromaufnahme (Blockiert):** ca. 850 mA

| Komponente | Geschwindigkeit |
| :--- | :--- |
| **Chassis** | ca. 20,8 cm/min |
| **Kamera Horizontal** | ca. 151,9° / min |
| **Kamera Vertikal** | ca. 151,9° / min |

> Detaillierte Informationen sind in [gearing.md](gearing.md) dokumentiert.


## Elektronik und Steuerung

Das System wird von einem **ESP32-Mikrocontroller** gesteuert, der über den I2C-Bus mit zwei PCA9685-PWM-Treibern kommuniziert. Diese steuern vier TB6612FNG-Motortreiber an, die wiederum die sieben Lego-Motoren versorgen.

| Komponente | Anzahl | Funktion |
| :--- | :---: | :--- |
| ESP32 | 1 | Mikrocontroller / Hauptsteuerung |
| PCA9685 PWM-Treiber | 2 | I2C-gesteuerter 16-Kanal-PWM (0x40: Richtung, 0x41: Geschwindigkeit) |
| TB6612FNG Motortreiber | 4 | Dual H-Brücke für je 2 Motoren (4 × 2 = 7 Motoren + 1 Reserve) |
| Buck Converter | 1 | Spannungswandler 9V → 3,3V für die Logikversorgung |
| Lego 8883 Motor | 7 | Antrieb: 4× Chassis, 2× Kamera horizontal, 1× Kamera vertikal |
| Netzteil 9V | 1 | Motorstromversorgung für alle TB6612FNG |

>Einen detaillierter Anschlussplan, Pinbelegungen und Inbetriebnahme-Schritte sind in [anschlussanleitung.md](anschlussanleitung.md) dokumentiert.

---
