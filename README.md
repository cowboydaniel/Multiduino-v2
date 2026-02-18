# Multiduino v2

**by Outback Electronics** — Arduino-Compatible Development Board

---

## Overview

The Multiduino v2 is an Arduino Uno-compatible development board designed for projects that need more than a standard Uno can offer. It uses the ATmega328PB processor and ATmega16U2 USB bridge — the same architecture as the official Arduino Uno R3, with a drop-in compatible (but enhanced) variant of the Uno's ATmega328P.

On top of the standard Uno feature set, the Multiduino v2 adds a substantial suite of on-board hardware: extra RAM and non-volatile storage, a precision real-time clock, a humidity and temperature sensor, a microSD card slot, and a switchable 3.3V power rail. Everything is ready to use without any external modules or wiring.

**Two ways to use this board:**

**Arduino Uno compatible mode** — Select **Arduino Uno** as your board in the Arduino IDE and the Multiduino v2 works immediately with standard Arduino libraries. This is the quickest way to get started and is fully supported. Compatible libraries for all on-board peripherals are available through the Arduino Library Manager.

**Multiduino core** — Outback Electronics provides a custom Arduino core specifically designed for this board. It includes tailored libraries for every on-board peripheral that expose hardware-specific functionality not possible with generic libraries. The Multiduino core is recommended for any project making full use of the board's hardware. See the [Multiduino Core repository](#) for installation instructions.

---

## What's On The Board

### Processor
The ATmega328PB is an enhanced version of the ATmega328P found in the official Arduino Uno R3. It is fully backward compatible with existing Arduino sketches and libraries. Compared to the 328P it adds additional timer/counters (providing more PWM channels), and four extra I/O pins (PE0–PE3) which are broken out on the extended header. The chip also includes a second USART, SPI, and I2C port internally, however these are not broken out on this board.

### Memory
Beyond the processor's built-in 2KB of RAM and 1KB of EEPROM, the Multiduino v2 includes three additional memory devices:

**2 Megabit SPI SRAM** — 256KB of fast external RAM accessible over SPI. Ideal for buffering large datasets, audio samples, or any data that is too large to fit in the processor's internal memory but does not need to survive a power cycle.

**64 Kilobit FRAM** — 8KB of non-volatile ferroelectric RAM. Unlike flash or EEPROM, FRAM can be written an effectively unlimited number of times (100 trillion write cycles) with no wear. Use it for variables, counters, or settings that change frequently and must be preserved across power cycles.

**32 Kilobit EEPROM with Unique Serial Number** — 4KB of standard I2C EEPROM. This chip also contains a factory-programmed 64-bit unique serial number, which can be read by your sketch and used to uniquely identify each individual board — useful for licensing, fleet management, or asset tracking.

### MicroSD Card Slot
A full-size microSD card slot (supports up to 32GB, FAT32 formatted) provides essentially unlimited storage for data logging, configuration files, audio, images, or any other file-based data. The SD card interface can be completely isolated from the SPI bus using the on-board slide switch, which is useful when no SD card is installed or when SPI bus conflicts need to be avoided.

### Real-Time Clock
The DS3231 is a high-precision, temperature-compensated real-time clock accurate to ±5 parts per million — less than half a second of drift per day — and requires no external crystal. It maintains the current date and time even when the board is powered off, provided a backup battery is connected to the on-board JST connector. A standard CR2032 coin cell in an appropriate holder, or a small LiPo battery, can be used for backup power.

### Humidity and Temperature Sensor
The SHT4x sensor measures ambient relative humidity (±1.8% accuracy) and temperature (±0.2°C accuracy) directly on the board. It communicates over I2C and is ready to use with no additional components.

### USB-C Connector
The board connects to your computer via USB-C for programming and serial communication, matching the same workflow as the Arduino Uno. The USB-C port is USB 2.0 and works with any standard USB-C cable.

---

## Pinout and Headers

The Multiduino v2 uses standard Arduino Uno-compatible headers, so Arduino shields and accessories are physically and electrically compatible.

**Digital I/O (IO0–IO7)** — 8 digital pins compatible with standard Arduino digital pins 0–7. IO0 and IO1 are the hardware serial port (RX/TX), also used during USB programming.

**Analog Inputs (A0–A5)** — 6 analog input pins compatible with standard Arduino analog pins. A4 and A5 are also the hardware I2C bus (SDA/SCL).

**SPI and Utility Header** — a 10-pin header exposing the SPI bus (SCK, MOSI, MISO), the dedicated SRAM chip select, the SD card chip select, I2C (SDA/SCL), the SD card detect signal, and AREF.

**Extended Pins (PE0–PE3)** — four additional I/O pins from the ATmega328PB's extended port, not present on a standard Uno. These can be used as general-purpose I/O or connected to on-board peripherals via solder jumpers (see the Solder Jumpers section).

**Power Header** — exposes 5V, raw input voltage, reset, and GND.

**ICSP Headers** — two standard 6-pin ICSP headers for programming the main processor and the USB bridge chip directly, without going through the bootloader.

---

## On-Board I2C Devices

All on-board I2C devices have unique addresses and none conflict with each other, leaving the bus free for additional external devices.

| Device | Address |
|--------|---------|
| SHT4x humidity/temperature sensor | 0x44 |
| EEPROM data | 0x50 |
| EEPROM serial number (read-only) | 0x58 |
| FRAM | 0x57 |
| DS3231 real-time clock | 0x68 |

The board uses a dual-voltage I2C architecture. The processor communicates at 5V, while the FRAM and temperature sensor operate at 3.3V. An on-board level translator bridges the two voltage domains transparently — your sketch communicates with all devices the same way regardless of their operating voltage.

---

## Slide Switch — SD Card Enable

The slide switch on the board enables or disables the microSD card interface.

- **ON** — the SD card is connected to the SPI bus and ready to use.
- **OFF** — the SD card is completely isolated from the SPI bus. Use this position when no SD card is installed, or if you need to ensure the SPI bus is unaffected by the SD card slot.

The 256KB SRAM is unaffected by the slide switch and is always accessible regardless of its position.

---

## Solder Jumpers

The board includes solder jumpers for configuring on-board hardware. These are small pads on the PCB. A jumper is **closed** by bridging the pads with a blob of solder, and **opened** by carefully cutting the trace between them with a sharp knife.

---

### JP2 — Auto-Reset

Controls whether the Arduino IDE can automatically reset the board to enter the bootloader when uploading a sketch.

| State | Behaviour |
|-------|-----------|
| **Closed** *(default)* | Auto-reset enabled. The IDE resets the board automatically at the start of each upload — no manual button press needed. |
| **Open** | Auto-reset disabled. You must manually press the reset button at the correct moment during upload, or use ICSP programming instead. |

Cut this jumper if you have an external device connected to the serial port that interferes with the reset pulse, or if you simply want to prevent accidental resets during upload.

---

### JP3 — PE0 / 32KHz RTC Output

Routes the real-time clock's 32KHz square wave output to the extended pin PE0.

| State | Behaviour |
|-------|-----------|
| **Open** *(default)* | PE0 is a free general-purpose I/O pin on the extended header. |
| **Closed** | The RTC's 32KHz output is connected to PE0. Configure PE0 as an **input** in your sketch. |

The 32KHz output can be used as an external clock source or timing reference. Do not configure PE0 as an output when this jumper is closed.

---

### JP4 — PE1 / RTC Alarm Interrupt

Routes the real-time clock's alarm interrupt and configurable square wave output to the extended pin PE1.

| State | Behaviour |
|-------|-----------|
| **Open** *(default)* | PE1 is a free general-purpose I/O pin on the extended header. |
| **Closed** | The RTC's SQW/INT output is connected to PE1. Configure PE1 as an **input** in your sketch. |

Via software this output can be set to generate a square wave (1Hz, 1.024kHz, 4.096kHz, or 8.192kHz) or to fire an interrupt when an RTC alarm triggers, enabling accurate wake-from-sleep functionality. Do not configure PE1 as an output when this jumper is closed.

---

### JP5 — PE2 / SD Card Switch State

Connects the extended pin PE2 to the same signal that controls the SD card slide switch, giving your sketch visibility into — and optionally control over — the SD card enable state.

| Position | Behaviour |
|----------|-----------|
| **Open** *(default)* | PE2 is a free general-purpose I/O pin. |
| **Closed** | PE2 is connected to the SD card enable line. |

**With JP5 closed, PE2 can be used in two ways:**

**As an input (read the switch state):** PE2 reads LOW when the slide switch is ON and the SD card is active, and HIGH when the switch is OFF and the SD card is disabled. This lets your sketch detect whether the SD card is enabled before attempting to use it.

**As an output (software control of the SD card):** With the slide switch in the OFF position, driving PE2 LOW in your sketch will enable the SD card — exactly as if the switch were turned on. Driving PE2 HIGH will keep the SD card disabled. This allows full software control of the SD card without touching the physical switch.

> **Warning:** If the slide switch is in the ON position, do not configure PE2 as an output. The switch holds the enable line LOW directly, and driving PE2 HIGH against it will cause a conflict between the MCU output and the switch. Always ensure the slide switch is OFF before using PE2 as an output to control the SD card.

---

### JP6 — PE3 Routing

JP6 is a two-position jumper that determines where the extended pin PE3 is connected. It works in conjunction with JP7 to configure 3.3V rail control.

| Position | Behaviour |
|----------|-----------|
| **A–C** *(default)* | PE3 is connected to the 3.3V rail enable line. Combined with JP7 in its default position, your sketch can drive PE3 HIGH to turn the 3.3V rail on, or LOW to turn it off. |
| **B–C** | PE3 is routed out to the extended header (pin 4) as a free general-purpose I/O pin, and has no connection to the 3.3V enable circuit. |

---

### JP7 — 3.3V Rail Enable Source

JP7 is a two-position jumper that determines what controls the 3.3V power rail enable line. It should be configured in conjunction with JP6.

| Position | Behaviour |
|----------|-----------|
| **A–C** *(default)* | The 3.3V rail enable line is connected to PE3 (via JP6). The rail defaults to **off** at power-up and is controlled by your sketch via PE3. |
| **B–C** | The 3.3V rail enable line is connected directly to +5V. The rail is **always on** as soon as the board is powered, regardless of JP6 or PE3. |

**Combined JP6 + JP7 configuration options:**

| JP7 | JP6 | Result |
|-----|-----|--------|
| **A–C** + **A–C** | MCU PE3 → EN line | 3.3V rail controlled by sketch. Drive PE3 HIGH to enable, LOW to disable. Rail defaults off at power-up. |
| **A–C** + **B–C** | PE3 routed to header, EN line floating | 3.3V rail stays **off** permanently. Use only if 3.3V devices are not needed. |
| **B–C** + any | +5V → EN line | 3.3V rail **always on** at power-up. PE3 is free for other use. |

> **Important:** In the default configuration (both jumpers A–C), the FRAM, temperature sensor, and SD card are all off until your sketch drives PE3 HIGH. If your project needs these devices available immediately at power-up — for example if you access the SD card before your sketch has had a chance to run — move JP7 to the B–C position instead.

Turning off the 3.3V rail via software powers down the FRAM, temperature sensor, and SD card simultaneously, which can significantly reduce power consumption during sleep or idle periods.

---

## LEDs

| LED | Colour | Function |
|-----|--------|----------|
| Power | Green | Lit whenever the board is powered |
| Pin 13 | Yellow | Connected to digital pin 13. Blinks during SPI communication and can be controlled with `digitalWrite(13, ...)` |
| RX | Yellow | Flashes when the board receives data over USB serial |
| TX | Yellow | Flashes when the board transmits data over USB serial |

---

## Power

The board is powered via the USB-C connector. The 5V rail supplies the processor and all 5V peripherals. The 3.3V rail powers the FRAM, temperature sensor, and SD card, and is controlled by the JP7 jumper setting.

A self-resetting polyfuse protects the USB power input against overcurrent faults and resets automatically once the fault clears.

**Approximate current consumption:**

| Condition | Current |
|-----------|---------|
| Idle, no SD card activity | ~95 mA |
| Active with SD card writing | ~345 mA peak |

The board draws well within the 500mA USB specification under normal operation.

---

## Getting Started

### Option 1 — Arduino Uno Compatible Mode

The quickest way to get up and running. No custom core installation required.

1. Install the [Arduino IDE](https://www.arduino.cc/en/software) if not already installed.
2. Connect the Multiduino v2 to your computer with a USB-C cable.
3. In the Arduino IDE, select **Tools → Board → Arduino Uno**.
4. Select the correct port under **Tools → Port**.
5. Open any sketch and click **Upload**.

For the on-board peripherals, use the standard Arduino Wire and SPI libraries. Compatible libraries for the DS3231, SHT4x, FRAM, and EEPROM are available through the Arduino Library Manager.

### Option 2 — Multiduino Core (Recommended)

The Multiduino core unlocks the full capability of the board's hardware with libraries written specifically for it. Features like software SD card control, 3.3V rail switching, and board-aware peripheral management are only available through the core.

1. Install the [Arduino IDE](https://www.arduino.cc/en/software) if not already installed.
2. Follow the installation instructions in the [Multiduino Core repository](#) to add the board to the IDE.
3. Select **Tools → Board → Multiduino v2**.
4. Connect via USB-C, select the correct port, and upload as normal.

All standard Arduino sketches remain compatible when using the Multiduino core.

---

## RTC Battery Backup

To retain the time and date when the board is powered off, connect a backup battery to the JST 1.00mm 2-pin connector. A CR2032 coin cell in a compatible JST-terminated holder (3V) is recommended. The RTC draws only a few microamps from the backup supply, so a single CR2032 will last several years.

---

*Multiduino v2 — Outback Electronics*
