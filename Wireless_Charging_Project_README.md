# Wireless Charging System for a Lithium-Ion Battery

A low-voltage educational prototype that transfers power wirelessly using a Qi-compatible transmitter/receiver pair, charges a **single-cell (1S) lithium-ion battery** through a dedicated charger module, and monitors battery-side voltage, current, and temperature using an ESP32.

> **Safety first:** This is an educational prototype, not a certified consumer charger. Never connect a Qi receiver directly to a lithium-ion cell. The receiver must feed a correctly configured, chemistry-compatible charger. Use a protected cell, verify the charger's current setting against the cell datasheet, and supervise all tests. Do not leave the prototype charging unattended.

## 1. Project overview

### Objective
Design and demonstrate contactless power transfer for charging a 1S lithium-ion battery while displaying basic battery telemetry.

### Main functions
- Wireless power transfer from a Qi transmitter to a Qi receiver.
- Regulated receiver output supplied to a 1S Li-ion charger.
- Battery-side voltage and current measurement using INA219.
- Battery temperature measurement using DS18B20.
- Serial monitoring through the Arduino IDE.
- Clear separation between the charging safety hardware and the monitoring microcontroller.

### Important design choice
The ESP32 is used **for monitoring only**. It does not replace the charger IC, battery protection circuit, or thermal protection. The charger module controls the CC/CV charging process and terminates charging according to its own design.

## 2. Block diagram

```text
5 V regulated USB supply
          |
          v
Qi transmitter pad  ))))) magnetic coupling (((((  Qi receiver coil
                                                     |
                                                     v
                                          Regulated 5 V receiver output
                                                     |
                                                     v
                                      1S Li-ion CC/CV charger module
                                                     |
                                             BAT+ / BAT-
                                                     |
                                      INA219 current-sense path
                                                     |
                                                     v
                                      Protected 1S Li-ion cell
                                                     |
                          +--------------------------+------------------+
                          |                                             |
                     INA219 I2C                                  DS18B20 sensor
                          |                                             |
                          +---------------- ESP32 ---------------------+
                                              |
                                      USB Serial Monitor
```

## 3. Bill of materials

| Item | Qty | Purpose / selection notes |
|---|---:|---|
| ESP32 development board | 1 | Telemetry controller |
| Qi transmitter pad/module | 1 | Wireless power transmitter; use a reputable, enclosed module |
| Qi receiver module | 1 | Must provide regulated 5 V output |
| 1S Li-ion CC/CV charger module | 1 | Must explicitly support a 4.2 V Li-ion cell |
| Protected 1S Li-ion cell | 1 | Use a reputable cell with known datasheet and protection |
| INA219 breakout | 1 | Battery-side current/voltage monitoring |
| DS18B20 temperature sensor | 1 | Temperature monitoring |
| 4.7 kΩ resistor | 1 | DS18B20 data pull-up |
| Breadboard / wires | As needed | Low-voltage sensor connections |
| Regulated 5 V USB supply | 1 | Powers Qi transmitter; must meet transmitter rating |
| Multimeter | 1 | Essential for checking polarity and voltage |

**Do not choose a charger by appearance alone.** Confirm its datasheet, supported chemistry, cell count, charge voltage, charge current, and protection features. The common TP4056 is for a single 4.2 V Li-ion cell only; its programmed current must be suitable for the exact cell. A protection-equipped board is not a substitute for a correctly selected charger.

## 4. System operation

1. The regulated 5 V supply powers the Qi transmitter.
2. The transmitter creates an alternating magnetic field.
3. The Qi receiver converts the received energy into regulated DC output.
4. The receiver's regulated output powers the input of the 1S CC/CV charger.
5. The charger applies the correct charging profile to the protected cell.
6. INA219 measures battery charging current and voltage in the battery-positive path.
7. DS18B20 measures the cell surface temperature.
8. ESP32 reads the sensors and prints the measurements to the Serial Monitor.

Wireless charging efficiency depends on coil alignment, distance, module rating, and temperature. Record the measured input/output values if you want to calculate an approximate efficiency.

## 5. Wiring

### A. Power path

```text
Qi receiver +5V  ---> Charger IN+
Qi receiver GND  ---> Charger IN-

Charger BAT+ ---> INA219 VIN+
INA219 VIN-   ---> Protected cell positive (+)

Charger BAT- ----------------> Protected cell negative (-)
ESP32 GND -------------------> Common low-voltage ground
```

The INA219 is inserted in series in the **positive battery lead**. Do not connect the battery positive directly across VIN+ and VIN-; the current path must pass through the module's shunt.

### B. ESP32 to INA219

| INA219 pin | ESP32 pin |
|---|---|
| VCC | 3V3 |
| GND | GND |
| SDA | GPIO 21 |
| SCL | GPIO 22 |

Use 3.3 V logic for the ESP32. Check your particular INA219 breakout's pinout before wiring.

### C. ESP32 to DS18B20

| DS18B20 wire/pin | Connection |
|---|---|
| VDD | ESP32 3V3 |
| GND | ESP32 GND |
| DATA | ESP32 GPIO 4 |
| 4.7 kΩ resistor | Between DATA and 3V3 |

For a bare three-wire DS18B20, verify the pin order from its datasheet. Do not rely only on wire colours.

### Important ground note
The charger/battery negative and ESP32 ground must share a reference for this monitoring arrangement. Only connect low-voltage, isolated modules as specified by their manufacturers. Do not connect mains wiring to a breadboard.

## 6. Arduino IDE setup

1. Install Arduino IDE.
2. Add ESP32 board support using Espressif's official Arduino-ESP32 installation instructions.
3. Select your ESP32 board and the correct COM port.
4. Open `firmware/wireless_battery_monitor.ino`.
5. Install these libraries from Library Manager:
   - Adafruit INA219
   - OneWire
   - DallasTemperature
6. Upload the sketch.
7. Open Serial Monitor at **115200 baud**.

## 7. Firmware

The complete sketch is in `firmware/wireless_battery_monitor.ino`.

It reports:
- INA219 bus voltage
- Shunt voltage
- Current
- Approximate power
- DS18B20 temperature
- A configurable temperature warning

The temperature warning is an **indication only**. It does not disconnect the charger. If the cell becomes unusually warm, stop the test by disconnecting the transmitter's low-voltage supply and investigate. Follow the cell and charger manufacturers' temperature limits.

## 8. Testing procedure

Perform the tests in this order:

1. **Inspect:** Check polarity, solder joints, exposed conductors, and module ratings.
2. **Receiver test without battery:** Power the Qi transmitter from its rated regulated supply. Check receiver output with a multimeter. Confirm it is within the charger's specified input range.
3. **Charger verification:** With the battery disconnected, verify the charger board documentation and programmed current. Do not guess the current from the board label.
4. **Sensor test:** Power the ESP32 from USB separately. Confirm INA219 and DS18B20 readings in Serial Monitor.
5. **Charging test:** Only after the previous checks, connect the protected cell exactly as specified by the charger manufacturer. Keep the setup supervised on a non-flammable surface.
6. **Alignment test:** Record receiver output and charging current at several coil alignments. Do not exceed module or cell ratings.
7. **Temperature observation:** Stop if temperature exceeds the limits in the cell/charger datasheets, rises unexpectedly, or the cell swells, smells, leaks, or makes unusual sounds.

Never short the cell, puncture it, heat it, reverse its polarity, or charge a damaged cell. Do not use loose salvaged cells of unknown history.

## 9. Suggested results table

Fill this table with your own measured values. Do not publish estimated values as experimental results.

| Trial | Coil alignment / gap | Receiver output (V) | Battery current (mA) | Cell temperature (°C) | Observation |
|---|---|---:|---:|---:|---|
| 1 | Best alignment | | | | |
| 2 | Slight offset | | | | |
| 3 | Larger offset | | | | |

### Optional efficiency calculation
If you have suitable meters and can measure transmitter input power and receiver output power:

`Efficiency (%) = (Receiver output power / Transmitter input power) × 100`

Use simultaneous measurements and state where each measurement was taken. This is a system-level estimate and includes losses in the transmitter and receiver.

## 10. Limitations

- This prototype is designed for one 1S cell only.
- The ESP32 does not perform safety-critical charge control.
- No battery percentage is shown because reliable state-of-charge estimation requires cell-specific characterization and a suitable fuel-gauge method.
- Wireless power transfer performance depends strongly on coil alignment and module design.
- This is not suitable for an EV pack, multi-cell pack, or unattended charging.

## 11. Future enhancements

- Add a calibrated fuel-gauge IC for better state-of-charge estimation.
- Add an OLED display for local telemetry.
- Send telemetry to an IoT dashboard using ESP32 Wi-Fi.
- Compare wireless and wired charging efficiency.
- Log temperature and current over time.
- Design a custom enclosure and a repeatable coil-alignment fixture.

## 12. Suggested GitHub repository structure

```text
wireless-lithium-battery-charger/
├── README.md
├── LICENSE
├── .gitignore
├── firmware/
│   └── wireless_battery_monitor.ino
└── docs/
    ├── block-diagram.md
    ├── wiring.md
    └── test-plan.md
```

## 13. Resume description

**Wireless Charging System for Lithium-Ion Batteries | ESP32, Qi Wireless Power, INA219, DS18B20**

Developed a low-voltage wireless charging prototype using a Qi transmitter/receiver pair and a dedicated 1S Li-ion CC/CV charger. Integrated an ESP32 with INA219 and DS18B20 sensors to monitor battery-side voltage, current, and temperature, and documented the system architecture, wiring, and test procedure.

## 14. References to consult

- Espressif Arduino-ESP32 documentation: https://docs.espressif.com/projects/arduino-esp32/
- Adafruit INA219 guide: https://learn.adafruit.com/adafruit-ina219-current-sensor-breakout
- DallasTemperature library: https://github.com/milesburton/Arduino-Temperature-Control-Library
- OneWire library: https://github.com/PaulStoffregen/OneWire
- Qi overview: https://www.wirelesspowerconsortium.com/

Check the exact datasheets for the Qi modules, charger board, battery cell, INA219 breakout, and temperature sensor used in your build.
