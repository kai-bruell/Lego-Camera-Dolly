# Anschlussanleitung v4
**ESP32 · 2x PCA9685 · 4x TB6612FNG · 7x LEGO 8884 Motor**  
12-Bit PWM für Richtung und Geschwindigkeit – alles über einen I2C-Bus

---

## Konzept

| PCA9685 #1 – Adresse 0x40 | PCA9685 #2 – Adresse 0x41 |
|---|---|
| CH 0–13 → AIN1/2 + BIN1/2 | CH 0–6 → PWMA + PWMB |
| Richtungssteuerung aller 7 Motoren | 12-Bit Geschwindigkeit aller 7 Motoren |

> **Info:** Beide PCA9685 teilen denselben I2C-Bus (SDA/SCL). Die Adresse von Chip #2 wird durch Brücken des A0-Pads auf HIGH von 0x40 auf 0x41 geändert.

---

## Schritt 1 – Buck Converter auf 3,3V einstellen
*Vor allem anderen – nur Buck Converter + Netzteil, keine weiteren Komponenten*

1. Buck Converter allein an 9V Netzteil anschließen.
2. Multimeter auf DC Volt, Messspitzen an OUT+ und OUT−.
3. Poti langsam drehen bis exakt 3,30V angezeigt werden.
4. Netzteil wieder trennen.

> ⚠️ **Achtung:** Poti sehr langsam drehen – eine Viertelumdrehung kann bereits 1V Unterschied machen.

---

## Schritt 2 – PCA9685 #2 auf Adresse 0x41 konfigurieren
*A0-Pad auf HIGH brücken – einmalig, vor dem Einbau*

PCA9685 #2 hat ab Werk dieselbe Adresse (0x40) wie #1. Das muss geändert werden:

1. PCA9685 #2 (noch nicht verbaut) in die Hand nehmen.
2. Auf der Unterseite das Lötpad **A0** suchen.
3. A0-Pad mit einem Löttropfen brücken (kurzschließen) → A0 = HIGH.
4. Neue Adresse: 0x40 + 1 = **0x41**.

> 💡 **Tipp:** Mit dem Multimeter prüfen – zwischen A0-Pin und VCC sollte ~0 Ohm messbar sein. Im Code: `pca1 = PCA9685(i2c, address=0x40)` und `pca2 = PCA9685(i2c, address=0x41)`.

---

## Schritt 3 – Gemeinsame Masse (GND-Bus) aufbauen

> **Info:** Alle GND-Leitungen auf eine gemeinsame Schiene auf dem Breadboard. Gilt für beide PCA9685.

| Von | Nach | Hinweis |
|---|---|---|
| Netzteil GND | GND-Bus | Eingangs- und Motorstrom-Masse |
| Buck Conv. OUT− | GND-Bus | 3,3V Logik-Masse |
| GND-Bus | ESP32 GND | Controller |
| GND-Bus | PCA9685 #1 GND | Chip 1 (Richtung, 0x40) |
| GND-Bus | PCA9685 #2 GND | Chip 2 (Geschwindigkeit, 0x41) |
| GND-Bus | TB6612 #1 GND | Treiber 1 |
| GND-Bus | TB6612 #2 GND | Treiber 2 |
| GND-Bus | TB6612 #3 GND | Treiber 3 |
| GND-Bus | TB6612 #4 GND | Treiber 4 |

---

## Schritt 4 – 9V Motorstrom: Netzteil → VM-Pins der Treiber

| Von | Nach | Hinweis |
|---|---|---|
| Netzteil +9V | Buck Conv. IN+ | Eingang für 3,3V-Erzeugung |
| Netzteil +9V | TB6612 #1 VM | Motorstrom Treiber 1 |
| Netzteil +9V | TB6612 #2 VM | Motorstrom Treiber 2 |
| Netzteil +9V | TB6612 #3 VM | Motorstrom Treiber 3 |
| Netzteil +9V | TB6612 #4 VM | Motorstrom Treiber 4 |

> ⚠️ **Achtung:** VM (9V Motorstrom) und VCC (3,3V Logik) sind verschiedene Pins! Verwechslung zerstört den Treiber sofort.

---

## Schritt 5 – 3,3V Logikversorgung: Buck Converter → alle Logikpins
*Beide PCA9685 + ESP32 + VCC und STBY aller Treiber*

| Von | Nach | Hinweis |
|---|---|---|
| Buck Conv. OUT+ | ESP32 3V3 | ESP32 Versorgung |
| Buck Conv. OUT+ | PCA9685 #1 VCC | Chip 1 Versorgung |
| Buck Conv. OUT+ | PCA9685 #2 VCC | Chip 2 Versorgung |
| Buck Conv. OUT+ | TB6612 #1 VCC | Logik Treiber 1 |
| Buck Conv. OUT+ | TB6612 #2 VCC | Logik Treiber 2 |
| Buck Conv. OUT+ | TB6612 #3 VCC | Logik Treiber 3 |
| Buck Conv. OUT+ | TB6612 #4 VCC | Logik Treiber 4 |
| Buck Conv. OUT+ | TB6612 #1 STBY | Treiber 1 freischalten |
| Buck Conv. OUT+ | TB6612 #2 STBY | Treiber 2 freischalten |
| Buck Conv. OUT+ | TB6612 #3 STBY | Treiber 3 freischalten |
| Buck Conv. OUT+ | TB6612 #4 STBY | Treiber 4 freischalten |

> 💡 **Tipp:** STBY optional an einen ESP32-GPIO für Software-Notaus aller Motoren gleichzeitig.

---

## Schritt 6 – I2C-Bus: ESP32 → beide PCA9685
*Ein Bus, zwei Chips – nur SDA und SCL benötigt*

> **Info:** Beide PCA9685 werden parallel an denselben I2C-Bus gehängt. Der ESP32 unterscheidet sie ausschließlich über die Adresse (0x40 vs. 0x41).

| Von | Nach | Hinweis |
|---|---|---|
| ESP32 GPIO 21 (SDA) | PCA9685 #1 SDA | Chip 1 Datenleitung |
| ESP32 GPIO 21 (SDA) | PCA9685 #2 SDA | Chip 2 Datenleitung (parallel!) |
| ESP32 GPIO 22 (SCL) | PCA9685 #1 SCL | Chip 1 Taktleitung |
| ESP32 GPIO 22 (SCL) | PCA9685 #2 SCL | Chip 2 Taktleitung (parallel!) |

> 💡 **Tipp:** Auf dem Breadboard eine SDA-Schiene und eine SCL-Schiene aufbauen. Beide PCA9685 und der ESP32 werden jeweils auf diese Schienen gesteckt.

---

## Schritt 7 – PCA9685 #1 (0x40) → TB6612FNG: Richtungssignale
*CH 0–13 für AIN1/2 und BIN1/2 aller 4 Treiber*

### Treiber 1 – Motor 1 & 2

| Von | Nach | Hinweis |
|---|---|---|
| PCA9685 #1  CH 0 | TB6612 #1 AIN1 | Richtung Motor 1 |
| PCA9685 #1  CH 1 | TB6612 #1 AIN2 | Richtung Motor 1 |
| PCA9685 #1  CH 2 | TB6612 #1 BIN1 | Richtung Motor 2 |
| PCA9685 #1  CH 3 | TB6612 #1 BIN2 | Richtung Motor 2 |

### Treiber 2 – Motor 3 & 4

| Von | Nach | Hinweis |
|---|---|---|
| PCA9685 #1  CH 4 | TB6612 #2 AIN1 | Richtung Motor 3 |
| PCA9685 #1  CH 5 | TB6612 #2 AIN2 | Richtung Motor 3 |
| PCA9685 #1  CH 6 | TB6612 #2 BIN1 | Richtung Motor 4 |
| PCA9685 #1  CH 7 | TB6612 #2 BIN2 | Richtung Motor 4 |

### Treiber 3 – Motor 5 & 6

| Von | Nach | Hinweis |
|---|---|---|
| PCA9685 #1  CH 8  | TB6612 #3 AIN1 | Richtung Motor 5 |
| PCA9685 #1  CH 9  | TB6612 #3 AIN2 | Richtung Motor 5 |
| PCA9685 #1  CH 10 | TB6612 #3 BIN1 | Richtung Motor 6 |
| PCA9685 #1  CH 11 | TB6612 #3 BIN2 | Richtung Motor 6 |

### Treiber 4 – Motor 7

| Von | Nach | Hinweis |
|---|---|---|
| PCA9685 #1  CH 12 | TB6612 #4 AIN1 | Richtung Motor 7 |
| PCA9685 #1  CH 13 | TB6612 #4 AIN2 | Richtung Motor 7 |
| PCA9685 #1  CH 14/15 | — frei — | Reserve |

> **Info:** AIN1=4095 + AIN2=0 = vorwärts. AIN1=0 + AIN2=4095 = rückwärts. AIN1=0 + AIN2=0 = Motor aus (trudelt nach). AIN1=4095 + AIN2=4095 = aktive Bremse (Motor stoppt sofort).

---

## Schritt 8 – PCA9685 #2 (0x41) → TB6612FNG: Geschwindigkeit
*CH 0–6 für PWMA und PWMB aller 4 Treiber – 12-Bit, 0–4095*

| Motor | PCA9685 #2 Kanal | TB6612 Pin | Hinweis |
|---|---|---|---|
| Motor 1 | CH 0 | TB6612 #1 PWMA | Geschwindigkeit Motor 1 |
| Motor 2 | CH 1 | TB6612 #1 PWMB | Geschwindigkeit Motor 2 |
| Motor 3 | CH 2 | TB6612 #2 PWMA | Geschwindigkeit Motor 3 |
| Motor 4 | CH 3 | TB6612 #2 PWMB | Geschwindigkeit Motor 4 |
| Motor 5 | CH 4 | TB6612 #3 PWMA | Geschwindigkeit Motor 5 |
| Motor 6 | CH 5 | TB6612 #3 PWMB | Geschwindigkeit Motor 6 |
| Motor 7 | CH 6 | TB6612 #4 PWMA | Geschwindigkeit Motor 7 |
| (Reserve) | CH 7 | TB6612 #4 PWMB | Kein 8. Motor – offen lassen |
| (Reserve) | CH 8–15 | — | Frei für zukünftige Erweiterungen |

> 💡 **Tipp:** PCA9685 #2 PWM-Frequenz auf 1000 Hz einstellen: `pca2.freq(1000)`. Duty-Cycle 0 = Motor steht, 4095 = volle Geschwindigkeit.

---

## Schritt 9 – LEGO 8884 Kabel aufschneiden & anschließen
*4 Adern – nur 3 davon werden benutzt*

| Ader | Verbunden mit | Hinweis |
|---|---|---|
| Schwarz | GND-Bus | Gemeinsame Masse |
| Rot (C1) | TB6612 AO1 oder BO1 | Motor+ |
| Blau (C2) | TB6612 AO2 oder BO2 | Motor− |
| Gelb (+9V) | **ISOLIEREN** | Nicht verwenden – sofort isolieren! |

> ⚠️ **Achtung:** Die gelbe Ader führt +9V vom LEGO-System. Sofort mit Schrumpfschlauch isolieren und nie anschließen!

1. Kabel ca. 5 cm vom Stecker durchtrennen.
2. Alle 4 Adern ca. 5 mm abisolieren.
3. Gelbe Ader sofort mit Schrumpfschlauch isolieren.
4. Rote Ader (C1) an AO1 bzw. BO1 des Treibers löten/klemmen.
5. Blaue Ader (C2) an AO2 bzw. BO2 des Treibers löten/klemmen.
6. Schwarze Ader an GND-Bus.

---

## Schritt 10 – Prüfung & Erstinbetriebnahme

### Checkliste vor dem Einschalten

- [ ] GND aller Komponenten auf gemeinsamem Bus?
- [ ] PCA9685 #2 A0-Pad gebrückt (0x41)?
- [ ] VM der TB6612FNG an 9V (nicht 3,3V!)?
- [ ] VCC der TB6612FNG an 3,3V?
- [ ] STBY aller Treiber an 3,3V?
- [ ] SDA beider PCA9685 an GPIO 21?
- [ ] SCL beider PCA9685 an GPIO 22?
- [ ] LEGO 8884 gelbe Ader isoliert?
- [ ] Buck Converter Output auf 3,3V eingestellt?

### Einschaltreihenfolge

1. Netzteil einschalten – noch kein ESP32-Code.
2. Multimeter: Buck Conv. Ausgang – muss 3,30V zeigen.
3. Multimeter: ESP32 3V3 – muss 3,3V zeigen.
4. Multimeter: VCC der TB6612FNG – muss 3,3V zeigen.
5. Multimeter: VM der TB6612FNG – muss 9V zeigen.
6. ESP32 per USB verbinden, I2C-Scanner flashen und beide Adressen (0x40 + 0x41) prüfen.
7. Erst dann den eigentlichen Motorsteuerungs-Code flashen.

> 💡 **Tipp (I2C-Scanner MicroPython):**
> ```python
> from machine import SoftI2C, Pin
> i2c = SoftI2C(scl=Pin(22), sda=Pin(21))
> print(i2c.scan())  # Ausgabe muss [64, 65] zeigen (0x40, 0x41)
> ```

---

## Schritt 11 – Beispiel-Code (MicroPython)
*Beide PCA9685 – saubere Trennung von Richtung und Geschwindigkeit*

```python
from machine import SoftI2C, Pin
from pca9685 import PCA9685

# I2C Bus initialisieren
i2c = SoftI2C(scl=Pin(22), sda=Pin(21), freq=400000)

# Chip 1: Richtung (0x40)
dir_pca = PCA9685(i2c, address=0x40)
dir_pca.freq(1000)

# Chip 2: Geschwindigkeit (0x41)
spd_pca = PCA9685(i2c, address=0x41)
spd_pca.freq(1000)  # 1000 Hz für DC-Motoren


def motor(ain1_ch, ain2_ch, pwm_ch, speed, forward=True):
    """
    ain1_ch / ain2_ch : Kanal auf PCA9685 #1 (Richtung)
    pwm_ch            : Kanal auf PCA9685 #2 (Geschwindigkeit)
    speed             : 0–4095
    forward           : True = vorwärts, False = rückwärts
    """
    if forward:
        dir_pca[ain1_ch] = 4095  # AIN1 = HIGH
        dir_pca[ain2_ch] = 0     # AIN2 = LOW
    else:
        dir_pca[ain1_ch] = 0     # AIN1 = LOW
        dir_pca[ain2_ch] = 4095  # AIN2 = HIGH
    spd_pca[pwm_ch] = speed      # Geschwindigkeit 0–4095


def motor_stop(ain1_ch, ain2_ch, pwm_ch, brake=True):
    """
    brake=True  : aktive Bremse (Motor stoppt sofort)
    brake=False : Motor aus (trudelt nach)
    """
    val = 4095 if brake else 0
    dir_pca[ain1_ch] = val
    dir_pca[ain2_ch] = val
    spd_pca[pwm_ch] = 0


# Beispiele:
motor(0, 1, 0, 2867, forward=True)   # Motor 1: 70% vorwärts
motor(4, 5, 2, 1638, forward=False)  # Motor 3: 40% rückwärts
motor_stop(0, 1, 0, brake=True)      # Motor 1: sofort bremsen
motor_stop(0, 1, 0, brake=False)     # Motor 1: trudeln lassen
```

---

## Schnellreferenz – alle Verbindungen

| Komponente | Pin | Verbunden mit |
|---|---|---|
| ESP32 | GPIO 21 (SDA) | PCA9685 #1 + #2 SDA (beide parallel) |
| ESP32 | GPIO 22 (SCL) | PCA9685 #1 + #2 SCL (beide parallel) |
| ESP32 | 3V3 | Buck Conv. OUT+ |
| ESP32 | GND | GND-Bus |
| PCA9685 #1 (0x40) | VCC + GND | Buck Conv. OUT+ / GND-Bus |
| PCA9685 #1 (0x40) | CH 0–13 | AIN1/2 + BIN1/2 aller 4 Treiber |
| PCA9685 #2 (0x41) | A0-Pad | Gebrückt auf HIGH → Adresse 0x41 |
| PCA9685 #2 (0x41) | VCC + GND | Buck Conv. OUT+ / GND-Bus |
| PCA9685 #2 (0x41) | CH 0–6 | PWMA + PWMB aller 4 Treiber |
| TB6612 #1–4 | VCC | Buck Conv. OUT+ (3,3V) |
| TB6612 #1–4 | GND | GND-Bus |
| TB6612 #1–4 | VM | Netzteil +9V |
| TB6612 #1–4 | STBY | Buck Conv. OUT+ (3,3V) |
| TB6612 #1–4 | AO1/AO2/BO1/BO2 | LEGO 8884 C1 (Rot) / C2 (Blau) |
| Buck Converter | IN+ / IN− | Netzteil +9V / GND |
| Buck Converter | OUT+ (3,3V) | ESP32, beide PCA9685, alle VCC+STBY |
| Buck Converter | OUT− | GND-Bus |

> ⚠️ **Achtung:** Immer zuerst alle Spannungen mit dem Multimeter prüfen. VM (9V) und VCC (3,3V) der TB6612FNG niemals vertauschen.
