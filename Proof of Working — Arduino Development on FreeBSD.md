## Overview

This document provides practical evidence that the Arduino development workflow is functional on **FreeBSD 15**, covering the complete process from installing the Arduino AVR platform to compiling an Arduino sketch and programming a physical Arduino Uno board.

The validation was performed using:

- **Operating System:** FreeBSD 15
- **Board:** Arduino Uno
- **Microcontroller:** ATmega328P
- **Arduino AVR Core:** 1.8.7
- **Arduino CLI:** `arduino2-cli`
- **AVR Programmer:** `avrdude` 7.3
- **Test Sketch:** Blink

| Screenshot | What it proves                                                              | Category                              |
| ---------- | --------------------------------------------------------------------------- | ------------------------------------- |
| `035114`   | Blink sketch created and `arduino2-cli compile` invoked                     | **Sketch / Compilation**              |
| `035121`   | `arduino-core-avr-1.8.7` installed as a FreeBSD package                     | **Package / Installation**            |
| `035305`   | Arduino CLI detects `arduino:avr 1.8.7`                                     | **Board Core Detection**              |
| `035322`   | Arduino CLI compiles Blink successfully; **970 bytes** generated            | **Compilation ✅**                     |
| `035850`   | `avrdude` detects ATmega328P via `/dev/cuaU0` and gets signature `0x1e950f` | **Hardware / Serial Communication ✅** |
| `040052`   | `avrdude` writes **970 bytes** to Arduino flash                             | **Flashing / Uploading ✅**            |
| `040418`   | Flash is read back and **970 bytes verified**                               | **Flash Verification ✅**              |
| `040454`   | Arduino CLI performs the upload and identifies `/dev/cuaU0`                 | **Arduino CLI Upload ✅**              |
| `040748`   | Arduino CLI invokes `/usr/local/bin/avrdude` with the Uno configuration     | **CLI → avrdude Integration ✅**       |
| `044421`   | Same upload process showing `avrdude` processing the generated HEX          | **Upload / Flashing**                 |
| `044431`   | Final successful flash: **970 bytes written**, 100%                         | **Flashing Proof ✅**                  |

---

# 1. Arduino AVR Core Installed Successfully

The Arduino AVR platform was packaged and installed on FreeBSD as:

```text
arduino-core-avr-1.8.7
```

The package metadata confirms that the package is installed natively for FreeBSD 15 amd64:

```text
Name            : arduino-core-avr
Version         : 1.8.7
Origin          : devel/arduino-core-avr
Architecture    : FreeBSD:15:amd64
Prefix          : /usr/local
Licenses        : LGPL21
Comment         : Arduino AVR boards core
```

The package provides the Arduino AVR platform, including:

- Arduino core sources
    
- Board definitions
    
- AVR variants
    
- Libraries
    
- Bootloaders
    
- Platform configuration
    
- Support required for AVR-based Arduino boards


_Figure 1 — `arduino-core-avr` 1.8.7 installed successfully on FreeBSD 15._

This establishes that the AVR board platform required for Arduino Uno development is available through the FreeBSD environment.

---

# 2. Arduino Blink Sketch Created

A standard Arduino Blink sketch was created on the FreeBSD system:

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

The sketch was stored at:

```text
/root/Arduino/Blink/Blink.ino
```

The sketch was then passed directly to Arduino CLI for compilation using the Arduino Uno board definition:

```text
arduino2-cli compile \
    --fqbn arduino:avr:uno \
    /root/Arduino/Blink
```

_Figure 2 — Blink sketch created on FreeBSD and submitted to `arduino2-cli compile` for the Arduino Uno._

This validates that a normal Arduino project can be created and processed using the FreeBSD-hosted Arduino CLI environment.

---

# 3. Arduino CLI Configuration on FreeBSD

The Arduino CLI configuration was explicitly configured to use the FreeBSD-installed Arduino ESP32/Arduino tooling directories.

The configuration was inspected using:

```text
arduino2-cli \
    --config-file /root/.arduino15/arduino-cli.yaml \
    config dump
```

The resulting configuration included:

```text
directories:
    data: /usr/local/share/arduino-esp32
    downloads: /usr/local/share/arduino-esp32/staging
    user: /root/Arduino
```

_Figure 3 — Arduino CLI configuration showing the FreeBSD installation and user sketch directories._

---

# 4. Arduino AVR Board Detection

The installed AVR platform was successfully recognized by Arduino CLI.

Running:

```text
arduino2-cli \
    --config-file /root/.arduino15/arduino-cli.yaml \
    core list
```

returned the Arduino AVR platform:

```text
ID           Installed   Latest   Name
arduino:avr  1.8.7       n/a      Arduino AVR Boards
```

_Figure 4 — Arduino CLI recognizes `arduino:avr` version 1.8.7._

This confirms that the Arduino AVR board platform is visible to the CLI and that the Arduino Uno board definition can be resolved.

---

# 5. Compilation of the Arduino Uno Sketch

The Blink sketch was compiled using the Arduino Uno:

```text
arduino2-cli \
    --config-file /root/.arduino15/arduino-cli.yaml \
    compile \
    --fqbn arduino:avr:uno \
    --output-dir /root/Arduino/Blink/build \
    /root/Arduino/Blink
```

The compilation completed successfully and produced the expected memory usage information:

```text
Sketch uses 970 bytes (3%) of program storage space.
Maximum is 32256 bytes.

Global variables use 9 bytes (0%) of dynamic memory,
leaving 2039 bytes for local variables.
Maximum is 2048 bytes.
```

_Figure 5 — Successful compilation of the Blink sketch for the Arduino Uno._

The generated firmware occupied only 970 bytes of the Uno's available 32,256-byte program storage.

This demonstrates that the FreeBSD environment can execute the Arduino build pipeline and generate firmware suitable for the AVR target.

---

# 6. Direct `avrdude` Communication with Arduino Uno

Before validating the Arduino CLI upload workflow, communication with the physical Arduino Uno was tested directly using `avrdude`.

The command used was:

```text
avrdude \
    -p atmega328p \
    -c arduino \
    -P /dev/cuaU0 \
    -b 115200 \
    -v
```

The output confirmed:

```text
Using port          : /dev/cuaU0
Using programmer    : arduino
Setting baud rate   : 115200
AVR Part            : ATmega328P

AVR device initialized and ready to accept instructions

Device signature = 0x1e950f (probably m328p)

avrdude done. Thank you.
```

_Figure 6 — `avrdude` successfully communicates with the ATmega328P through `/dev/cuaU0`._

The device signature:

```text
0x1e950f
```

matches the ATmega328P used by the Arduino Uno.

This verifies the complete lower-level communication path:

```text
FreeBSD
   ↓
/dev/cuaU0
   ↓
USB-to-serial interface
   ↓
Arduino Uno bootloader
   ↓
ATmega328P
```

---

# 7. Firmware Flashing with `avrdude`

The successfully compiled HEX file was then flashed directly to the Arduino Uno.

The command was:

```text
avrdude \
    -p atmega328p \
    -c arduino \
    -P /dev/cuaU0 \
    -b 115200 \
    -U flash:w:/root/Arduino/Blink/build/Blink.ino.hex:i
```

`avrdude` successfully initialized the AVR device and erased the existing flash contents:

```text
AVR device initialized and ready to accept instructions

device signature = 0x1e950f (probably m328p)

erasing chip
```

The firmware was then written:

```text
reading input file /root/Arduino/Blink/build/Blink.ino.hex
with 970 bytes in 1 section
using 8 pages and 54 pad bytes

writing 970 bytes flash ...
Writing | ######################## | 100% 0.52 s

970 bytes of flash written
```

_Figure 7 — 970 bytes of compiled Arduino firmware successfully written to the Arduino Uno._

---

# 8. Flash Verification

The programming process did not stop after writing the firmware.

`avrdude` was instructed to verify the contents of the device against the generated HEX file:

```text
verifying flash memory against
/root/Arduino/Blink/build/Blink.ino.hex
```

The verification completed successfully:

```text
Reading | ######################## | 100% 0.45 s

970 bytes of flash verified

avrdude done. Thank you.
```

_Figure 8 — Firmware successfully verified after being written to the Arduino Uno._

The contents of the Arduino's flash memory were read back and matched the compiled firmware.

The complete low-level programming process was therefore validated:

```text
Compile
   ↓
Generate HEX
   ↓
Connect to ATmega328P
   ↓
Erase flash
   ↓
Write firmware
   ↓
Read back flash
   ↓
Verify firmware
```

---

# 9. Arduino CLI Upload Workflow

After validating the lower-level `avrdude` workflow, the same process was executed through Arduino CLI.

The upload command was:

```text
arduino2-cli \
    --config-file /root/.arduino15/arduino-cli.yaml \
    upload \
    --fqbn arduino:avr:uno \
    -p /dev/cuaU0 \
    /root/Arduino/Blink
```

Arduino CLI invoked the FreeBSD-installed AVR upload tooling and ultimately executed `avrdude`.

The verbose output showed the generated command:

```text
/usr/local/bin/avrdude \
    -C /usr/local/etc/arduino-avrdude.conf \
    -v \
    -V \
    -patmega328p \
    -carduino \
    -P/dev/cuaU0 \
    -b115200 \
    -D \
    -Uflash:w:.../Blink.ino.hex:i
```

_Figure 9 — Arduino CLI invokes `avrdude` using the FreeBSD serial device and Arduino AVR configuration._

This demonstrates that the user does not need to manually invoke `avrdude`.

The normal Arduino workflow can be used:

```text
arduino2-cli compile
        ↓
arduino2-cli upload
        ↓
avrdude
        ↓
Arduino Uno
```

---

# 10. Serial Port Discovery

The Arduino CLI upload process also identified the newly available serial port:

```text
New upload port: /dev/cuaU0 (serial)
```

_Figure 10 — Arduino CLI identifies `/dev/cuaU0` as the Arduino upload serial port._

The `/dev/cuaU0` device demonstrates that the Arduino USB serial interface is exposed correctly by FreeBSD and can be consumed by the Arduino development tooling.

This validates an important part of the project's objective: enabling Arduino hardware communication through FreeBSD's native serial device infrastructure.

---

# 11. Handling Unsupported Arduino Built-in Tools

During validation, Arduino CLI attempted to initialize several built-in discovery and monitoring tools:

```text
builtin:serial-discovery
builtin:serial-monitor
builtin:mdns-discovery
builtin:dfu-discovery
builtin:ctags
```

The FreeBSD package index did not provide native versions of several of these tools, resulting in messages such as:

```text
no versions available for the current OS
```

For example:

```text
Error downloading tool builtin:serial-discovery@1.5.2:
no versions available for the current OS
```

Similar messages appeared for:

- `serial-monitor`
    
- `mdns-discovery`
    
- `dfu-discovery`


_Figure 11 — Arduino CLI reports unsupported optional discovery/monitoring tools for FreeBSD._

These messages are important to document because they distinguish **optional Arduino CLI tooling limitations** from the core AVR development workflow.

Despite these unsupported tools, the essential AVR workflow remained operational:

```text
Arduino AVR platform
        ↓
Sketch compilation
        ↓
HEX generation
        ↓
Serial device access
        ↓
avrdude
        ↓
Arduino Uno programming
```

The successful compilation and upload demonstrate that these missing tools do not prevent core Arduino AVR development on FreeBSD.

---

# 12. End-to-End Validation

The complete workflow was successfully demonstrated on a physical Arduino Uno.

The final validated workflow is:

```text
                    FreeBSD 15
                        │
                        ▼
                Arduino CLI
                        │
              ┌─────────┴─────────┐
              │                   │
              ▼                   ▼
       Arduino AVR Core      Board Definition
          1.8.7               arduino:avr:uno
              │                   │
              └─────────┬─────────┘
                        ▼
                  Blink.ino
                        │
                        ▼
                    Compile
                        │
                        ▼
                 Blink.ino.hex
                        │
                        ▼
                    avrdude
                        │
                 /dev/cuaU0
                        │
                        ▼
                 Arduino Uno
                        │
                        ▼
                  ATmega328P
                        │
                        ▼
                 Flash + Verify
```

The test established all of the following:

|Component|Result|
|---|---|
|FreeBSD 15 environment|**Validated**|
|Arduino AVR Core 1.8.7|**Installed**|
|Arduino Uno board definition|**Detected**|
|Blink sketch creation|**Validated**|
|Arduino CLI compilation|**Successful**|
|HEX firmware generation|**Successful**|
|`/dev/cuaU0` serial device|**Detected**|
|ATmega328P communication|**Successful**|
|AVR device signature|**0x1e950f**|
|Firmware flashing|**Successful**|
|Flash verification|**Successful**|
|Arduino CLI upload workflow|**Successful**|
|`avrdude` integration|**Successful**|

---
# Conclusion

The Arduino AVR core was successfully packaged and installed, Arduino CLI was able to recognize the AVR platform and compile a standard Blink sketch, and the resulting firmware was successfully transferred to a physical Arduino Uno through FreeBSD's serial device interface.

At the hardware level, `avrdude` detected the ATmega328P using the expected device signature:

```text
0x1e950f
```

The generated 970-byte firmware was successfully written to the Arduino's flash memory and subsequently verified:

```text
970 bytes of flash written
970 bytes of flash verified
```

The remaining messages concerning unavailable `serial-monitor`, `mdns-discovery`, and related tools identify areas where additional FreeBSD-specific packaging or upstream support could be developed. They do not prevent the core AVR compilation and upload workflow from functioning.

This evidence therefore demonstrates a functional end-to-end Arduino development path on FreeBSD:

**Sketch → Arduino CLI → AVR compilation → HEX firmware → FreeBSD serial device → avrdude → Arduino Uno → ATmega328P flash → verification.**