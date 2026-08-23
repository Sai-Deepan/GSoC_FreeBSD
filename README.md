**Google Summer of Code 2026 — The FreeBSD Project**

Hello! My name is **Deepan Sai**, This repository documents my work as a **Google Summer of Code 2026 contributor with The FreeBSD Project**, focused on improving the Arduino and ESP32 development experience on FreeBSD.

The goal of this project is to make it possible to use modern Arduino development workflows natively on FreeBSD, including board platform installation, compilation, serial communication, and firmware uploading.

The project covers both **FreeBSD packaging work** and **upstream contributions to the Arduino ecosystem** required to make the complete workflow functional.

---

## Project Goals

The primary objectives of the project are:

- Port **Arduino CLI** to FreeBSD.
    
- Package the Arduino AVR platform for FreeBSD.
    
- Improve Arduino serial communication support on FreeBSD.
    
- Add or improve FreeBSD support in Arduino tooling.
    
- Validate the complete Arduino development workflow with physical hardware.
    
- Document the process so that future FreeBSD users and contributors can reproduce the setup.

The intended workflow is:

```text
Arduino Sketch
      │
      ▼
 Arduino CLI
      │
      ├── Board Core
      │
      ├── Compiler
      │
      └── Upload Tools
              │
              ▼
        FreeBSD Serial
              │
              ▼
        Arduino Board
```

---

# Major Contributions

## 1. Arduino CLI on FreeBSD

Arduino CLI provides the command-line infrastructure used to manage Arduino boards, platforms, libraries, compilation, uploading, and related development functionality.

A FreeBSD port was developed for Arduino CLI and successfully built and installed as:

```text
arduino2-cli-1.3.1
```

The port uses the FreeBSD ports framework and Go modules to build Arduino CLI natively.

The resulting binary is installed under:

```text
/usr/local/bin/arduino2-cli
```

The port was tested through the FreeBSD ports build system and package installation workflow.

Detailed documentation:

- [`Arduino CLI`](https://github.com/Sai-Deepan/GSoC_FreeBSD/blob/master/contributions/Arduino%20Ecosystem%20Ports%20Built%20on%20FreeBSD.md)

---

## 2. Arduino AVR Core

The Arduino AVR platform was packaged for FreeBSD:

```text
arduino-core-avr-1.8.7
```

The package provides the Arduino AVR platform required for boards such as the Arduino Uno.

The installed package contains:

- Arduino AVR board definitions
    
- Arduino core sources
    
- AVR variants
    
- Libraries
    
- Bootloaders
    
- Platform configuration

The package was successfully installed on:

```text
FreeBSD 15 amd64
```

Detailed documentation:

- [`Arduino AVR Core`](https://github.com/Sai-Deepan/GSoC_FreeBSD/blob/master/contributions/Arduino%20Ecosystem%20Ports%20Built%20on%20FreeBSD.md)

---

# 3. Arduino Uno End-to-End Validation

A physical Arduino Uno was used to validate the complete development workflow.

A standard Blink sketch was created:

```cpp
void setup() {
    pinMode(LED_BUILTIN, OUTPUT);
}

void loop() {
    digitalWrite(LED_BUILTIN, HIGH);
    delay(1000);
    digitalWrite(LED_BUILTIN, LOW);
    delay(1000);
}
```

The sketch was compiled using:

```bash
arduino2-cli compile \
    --fqbn arduino:avr:uno \
    /root/Arduino/Blink
```

Compilation completed successfully:

```text
Sketch uses 970 bytes (3%) of program storage space.
Maximum is 32256 bytes.

Global variables use 9 bytes (0%) of dynamic memory,
leaving 2039 bytes for local variables.
Maximum is 2048 bytes.
```

The resulting firmware was then uploaded to the physical Arduino Uno.

The board was exposed by FreeBSD as:

```text
/dev/cuaU0
```

`avrdude` successfully detected the ATmega328P:

```text
AVR device initialized and ready to accept instructions
device signature = 0x1e950f
```

The generated firmware was successfully written:

```text
970 bytes of flash written
```

and verified:

```text
970 bytes of flash verified
```

Detailed proof:

- [`Working Proof`](https://github.com/Sai-Deepan/GSoC_FreeBSD/tree/master/media/video)
    
- [`Arduino Uno Validation`](https://github.com/Sai-Deepan/GSoC_FreeBSD/blob/master/Proof%20of%20Working%20%E2%80%94%20Arduino%20Development%20on%20FreeBSD.md)

---

# 4. Arduino Serial Communication

A major part of the project involves making Arduino serial communication work correctly on FreeBSD.

The Arduino Uno was successfully exposed through the FreeBSD serial device:

```text
/dev/cuaU0
```

Communication with the board was tested using `avrdude`:

```bash
avrdude \
    -p atmega328p \
    -c arduino \
    -P /dev/cuaU0 \
    -b 115200 \
    -v
```

The Arduino bootloader responded successfully and returned the expected ATmega328P device signature:

```text
0x1e950f
```

This validates the communication path between:

```text
FreeBSD USB subsystem
        ↓
FreeBSD serial device
        ↓
/dev/cuaU0
        ↓
Arduino Uno bootloader
        ↓
ATmega328P
```

Detailed documentation:

- [`Serial Port Support`](https://github.com/Sai-Deepan/GSoC_FreeBSD/blob/master/contributions/Arduino%20Ecosystem%20Ports%20Built%20on%20FreeBSD.md)
    

---

# 5. Upstream Arduino Contributions

In addition to FreeBSD ports work, changes were contributed to upstream Arduino projects to improve FreeBSD compatibility.

## Arduino Serial Discovery

FreeBSD support was contributed to Arduino's serial discovery infrastructure.

Upstream pull request:

[Arduino serial-discovery — Pull Request #122](https://github.com/arduino/serial-discovery/pull/122?utm_source=chatgpt.com)

This work is important because serial device discovery forms part of the Arduino CLI workflow for identifying connected boards.

---

## Arduino Firmware Updater

FreeBSD-related work was also performed in Arduino's firmware updater tooling.

Upstream project:

[Arduino Firmware-Updater — Pull Request #333](https://github.com/arduino/arduino-fwuploader/pull/333)

---

# Testing and Validation

## Package-Level Testing

FreeBSD packages were built and installed through the ports framework.

Example:

```bash
pkg info arduino-core-avr
```

Result:

```text
arduino-core-avr-1.8.7
Architecture : FreeBSD:15:amd64
Origin       : devel/arduino-core-avr
```

---

## Platform-Level Testing

Arduino CLI successfully detected the AVR platform:

```text
ID           Installed   Latest   Name
arduino:avr  1.8.7       n/a      Arduino AVR Boards
```

---

## Compilation Testing

The Arduino Blink sketch successfully compiled for:

```text
arduino:avr:uno
```

with:

```text
970 bytes (3%) of program storage space
9 bytes (0%) of dynamic memory
```

---

## Hardware Testing

The physical Arduino Uno was successfully detected:

```text
/dev/cuaU0
```

and the ATmega328P returned:

```text
0x1e950f
```

---

## Flash Testing

Firmware was successfully written:

```text
970 bytes of flash written
```

and subsequently verified:

```text
970 bytes of flash verified
```

---

# Proof of Working

Screenshots and other evidence are stored under:

```text
media/
```

The evidence covers the complete workflow:

```text
01  Arduino AVR Core installation
02  Arduino CLI configuration
03  Arduino AVR platform detection
04  Blink sketch compilation
05  Arduino Uno detection
06  Firmware flashing
07  Flash verification
08  Arduino CLI upload
09  Arduino CLI → avrdude integration
10  Arduino CLI flashing
11  Successful flash completion
```

---

# Repository Structure

```text
.
├── README.md
├── Proof of Working — Arduino Development on FreeBSD.md
|
├── contributions/
│   ├── Arduino Ecosystem Ports Built on FreeBSD.md
│   └── Arduino FreeBSD Support Contributions.md
│
├── media/
|   ├── screenshots/
|	│   ├── 01-arduino-core-avr-installed.png
|	│   ├── 02-arduino-cli-configuration.png
|	│   ├── 03-arduino-avr-platform-detected.png
|	│   ├── 04-blink-sketch-compiled.png
|	│   ├── 05-arduino-uno-detected.png
|	│   ├── 06-firmware-flashed.png
|	│   ├── 07-flash-verification.png
|	│   ├── 08-arduino-cli-upload.png
|	│   ├── 09-arduino-cli-avrdude-integration.png
|	│   ├── 10-arduino-cli-flashing.png
|	│   └── 11-flash-written-successfully.png
|	|
|	├── video/
|	│   └── 01-ArduinoUno-BlinkTest.mov
│
├── references/
│   ├── FreeBSD-Code-Review-with-git-arc.pdf
│   └── porters-handbook_en.pdf
```

The exact directory structure may evolve as additional project documentation is added.

---

# Acknowledgements

This project was completed as part of **Google Summer of Code 2026 with The FreeBSD Project**.

I would like to thank my mentor Moin Rahman and the FreeBSD community for their guidance, reviews, technical discussions, and support throughout the project.
