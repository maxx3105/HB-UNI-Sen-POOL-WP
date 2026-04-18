# HB-UNI-Sen-POOL-WP

[![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc-sa/4.0/)

AskSin++-basierter Pool-Wärmepumpen-Controller für Homematic/OpenCCU.

Das zugehörige CCU-Addon befindet sich unter [HB-AddOn](https://github.com/maxx3105/HB-AddOn).

Basiert auf [AskSin++](https://github.com/pa-pa/AskSinPP) von [pa-pa](https://github.com/pa-pa) und [jp112sdl](https://github.com/jp112sdl).

---

## Features

- 3× NTC-Temperatursensor (Pool, Solar/Vorlauf, Luft/Frostschutz)
- pH-Messung mit Zwei-Punkt-Kalibrierung (pH 7.0 / pH 4.0)
- ORP/Redox-Messung mit konfigurierbarem Offset
- Druckmessung (0–0.5 MPa Sensor)
- Durchflussmessung via PCF8583 Impulszähler
- 4× Relaisausgänge (Pumpe, Wärmepumpe, Ventile etc.)
- 3× Shutter-Contact-Eingänge (Strömungswächter, Druckschalter etc.)
- 20×4 I²C LCD-Display mit konfigurierbarer Backlight-Zeit
- Temperatursensor-Offsets per CCU konfigurierbar (±1.4K in 0.2K-Schritten)

---

## Hardware

### Mikrocontroller

| Komponente | Beschreibung |
|------------|--------------|
| MCU | ATmega1284P |
| Funkmodul | CC1101 868 MHz |
| Display | 20×4 I²C LCD (Adresse 0x27) |
| Impulszähler | PCF8583 (Adresse 0xA0) |

### Pin-Belegung

| Pin | Arduino | Funktion |
|-----|---------|----------|
| PA0 | A0 | NTC Sensor 1 (Pool) |
| PA1 | A1 | NTC Sensor 2 (Solar/Vorlauf) |
| PA2 | A2 | NTC Sensor 3 (Luft/Frostschutz) |
| PA5 | A5 | Drucksensor |
| PA6 | A6 | Config-Taste |
| PA7 | A7 | pH/ORP-Signal |
| PB0 | 0 | Status-LED |
| PB1 | 1 | Shutter-Contact 1 |
| PB2 | 2 | Shutter-Contact 2 |
| PB3 | 3 | Shutter-Contact 3 |
| PB4 | 4 | CC1101 CS |
| PC2 | 18 | Taster SW3 |
| PC3 | 19 | Taster SW4 |
| PC4 | 20 | Relais 3 |
| PC5 | 21 | Relais 4 |
| PC6 | 22 | Relais 1 |
| PC7 | 23 | Relais 2 |
| PD2 | 10 | CC1101 GDO0 |
| PD3 | 11 | Taster SW1 |
| PD4 | 12 | Taster SW2 |
| PD7 | 15 | pH/ORP Sensor-Umschalter |
| PC0/PC1 | SDA/SCL | I²C (LCD + PCF8583) |

### NTC-Sensor Parameter

| Parameter | Wert |
|-----------|------|
| R0 | 10 kΩ |
| T0 | 25 °C |
| B-Wert | 3950 |
| Überabtastung | 4 Bit |
| Referenzspannung | 3300 mV |

### Drucksensor

| Parameter | Wert |
|-----------|------|
| Typ | 0–0.5 MPa (anpassbar auf 0–1.2 MPa) |
| Korrekturwert | `ANALOG_SOCKET_VALUE 150` |
| Sensor-Faktor | 1.6 (0.5 MPa) / 0.75 (1.2 MPa) |

---

## Software

### Abhängigkeiten

| Bibliothek | Quelle |
|------------|--------|
| AskSin++ | [pa-pa/AskSinPP](https://github.com/pa-pa/AskSinPP) |
| EnableInterrupt | Arduino Library Manager |
| LowPower | Arduino Library Manager |
| LiquidCrystal_I2C | Arduino Library Manager |
| PCF8583 | Enthalten in AskSin++ |

### Arduino IDE Einstellungen

| Einstellung | Wert |
|-------------|------|
| Board | MightyCore ATmega1284 |
| Takt | 8 MHz intern |
| Bootloader | No bootloader (ISP-Programmierung) |
| Compiler | Standard |

### Installation

1. [Arduino IDE](https://www.arduino.cc/en/software) installieren
2. MightyCore Board-Support hinzufügen:
   - Arduino IDE → Einstellungen → Zusätzliche Boardverwalter-URLs:
   ```
   https://mcudude.github.io/MightyCore/package_MCUdude_MightyCore_index.json
   ```
3. Werkzeuge → Board → MightyCore → ATmega1284
4. Alle Abhängigkeiten über den Bibliotheksverwalter installieren
5. `HB-UNI-Sen-POOL-WP.ino` öffnen
6. Device ID, Serial und Modell-ID bei Bedarf anpassen (Zeile 86–93)
7. Sketch kompilieren und via ISP flashen

### Konfiguration im Sketch

```cpp
// Geräteidentifikation – bei mehreren Geräten anpassen!
const struct DeviceInfo PROGMEM devinfo = {
  {0xF3, 0x18, 0x01},    // Device ID  – muss eindeutig sein
  "JPPOOL0001",           // Serial     – muss eindeutig sein (10 Zeichen)
  {0xFC, 0x60},           // Model ID   – passend zum HB-AddOn
  0x10,                   // Firmware Version
  0x53,                   // Device Type
  {0x01, 0x01}            // Info Bytes
};

// Relais-Logik
#define INVERT_RELAY false   // true = LOW-aktiv (z.B. Optokoppler-Relais)

// Drucksensor
#define ANALOG_SOCKET_VALUE 150   // Nullpunkt-Korrektur, hardwarespezifisch anpassen
```

---

## Homematic-Kanalstruktur

| Kanal | Typ | Inhalt |
|-------|-----|--------|
| 0 | MAINTENANCE | Systemkanal (UNREACH, RSSI etc.) |
| 1 | HB_GENERIC | Temp1/2/3, pH, ORP, Druck, Durchfluss, Kalibrierstatus |
| 2 | WEATHER | Temperatur Sensor 2 |
| 3 | WEATHER | Temperatur Sensor 3 |
| 4–6 | SHUTTER_CONTACT | Digitaleingänge |
| 7–10 | SWITCH | Relaisausgänge |

### Nachrichtenformat (0x53-Frame)

```
Byte  9   : 0x41 (Kanal 1 Marker) + Temp1 High
Byte 10   : Temp1 Low
Byte 11-12: pH (×100, uint16)
Byte 13-14: ORP (int16, mV)
Byte 15-16: Druck (uint16)
Byte 17   : Durchfluss (uint8, l/min)
Byte 18   : 0x00
Byte 19   : 0x42 (Kanal 2 Marker) + Temp2 High
Byte 20   : Temp2 Low
Byte 21   : 0x00
Byte 22   : 0x43 (Kanal 3 Marker) + Temp3 High
Byte 23   : Temp3 Low
Byte 24   : 0x00 (Länge 0x19 = 25 Byte)
```

---

## pH-Kalibrierung

Die Kalibrierung erfolgt direkt am Gerät über den Config-Button:

1. **Langen Tastendruck** (>3s) → Kalibriermodus startet (LCD zeigt "CALIBRATION MODE")
2. **pH 7.0 Pufferlösung** einlegen → kurzen Tastendruck → Spannung wird gemessen
3. **pH 4.0 Pufferlösung** einlegen → kurzen Tastendruck → Spannung wird gemessen
4. Kalibrierung wird im EEPROM gespeichert, Kalibriermodus beendet sich
5. **Automatischer Timeout** nach 15 Minuten

Die Kalibrierungsdaten (neutralVoltage, acidVoltage, Temperatur) werden im UserStorage des ATmega1284P gespeichert und überleben einen Stromausfall.

> ⚠️ Bei ungültigen Kalibrierwerten (außerhalb der Toleranz) erscheint "CAL FAILED" und der Vorgang muss wiederholt werden.

---

## CCU-Addon

Für die Integration in OpenCCU/RaspberryMatic/CCU3 wird das **HB-AddOn** benötigt:

👉 [https://github.com/maxx3105/HB-AddOn](https://github.com/maxx3105/HB-AddOn)

---

## Lizenz

[Creative Commons BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)

Ursprüngliche Autoren: [pa-pa](https://github.com/pa-pa) / [jp112sdl](https://github.com/jp112sdl)
