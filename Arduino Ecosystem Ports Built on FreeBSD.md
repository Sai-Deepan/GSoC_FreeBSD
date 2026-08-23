# Arduino CLI

Link: [https://reviews.freebsd.org/D57589](https://reviews.freebsd.org/D57589)

Arduino CLI provides the command-line infrastructure used to manage Arduino boards, platforms, libraries, compilation, uploading and related functionality.

The first major part of the contribution was getting Arduino CLI itself working as a FreeBSD package.

The port was ultimately built as:

```text
arduino2-cli-1.3.1
```

and installed as:

```text
/usr/local/bin/arduino2-cli
```

---
## Version and Build Information

The successfully built executable reported:

```text
arduino-cli
Version: 1.3.1
Commit: 08ff7e2
Date: 2026-05-26T18:53:36Z
```

The package was available through the FreeBSD ports system under:

```text
devel/arduino2-cli
```

---
## Build Challenges

One of the challenges encountered during the Arduino CLI porting work involved upstream dependency/version compatibility.

The port originally encountered issues around Go modules and dependency versions.

The build environment was adjusted to use an available Arduino CLI release and its corresponding dependencies.

The successful build established the basic FreeBSD-native foundation required for the rest of the project.

___
### Arduino Language Server

Link: https://reviews.freebsd.org/D57969

Arduino Language Server provides language-server functionality for Arduino development environments, enabling features such as code intelligence, symbol information, diagnostics and editor integration.

The contribution focused on bringing the Arduino Language Server into the FreeBSD environment and establishing the dependencies and build infrastructure required for native operation.

The work involved adapting the build process to the FreeBSD environment, resolving dependency and tooling compatibility issues, and validating that the resulting components could be built and installed through the FreeBSD ports framework.

---

## Build and Integration

The Arduino Language Server was integrated into the FreeBSD development environment as a native port.

The port provides the foundation required for Arduino-aware development tools and editors on FreeBSD, allowing developers to work with Arduino projects without depending on a Linux-specific environment.

The work also complements the Arduino CLI port, since the language-server functionality can use the same Arduino toolchain and board/platform information provided by Arduino CLI.

---

## Build Challenges

The primary challenges involved adapting the upstream build process and its dependencies to FreeBSD.

- Go/toolchain compatibility
    
- Upstream dependency versions
    
- FreeBSD filesystem and installation paths
    
- Port build and staging requirements
    
- Ensuring that the generated binaries were installed correctly
    
- Verifying compatibility with the Arduino CLI-based workflow

Successfully building the language-server component extends the Arduino ecosystem beyond basic command-line compilation and upload functionality, providing the foundation for a complete Arduino development environment on FreeBSD.

---
### Arduino Serial Discovery

Link: https://reviews.freebsd.org/D58919

Arduino Serial Discovery is responsible for identifying and providing information about serial devices connected to the system.

This component is particularly important for Arduino development because board detection and serial-port identification are required before operations such as uploading sketches, monitoring serial output, and communicating with development boards can take place.

The contribution focused on making Arduino Serial Discovery usable within the FreeBSD environment and integrating it into the broader Arduino toolchain.

---
## FreeBSD Integration

The work involved adapting the upstream build process to FreeBSD, resolving platform-specific compatibility issues, and validating the resulting executable in the FreeBSD environment.

The component provides the infrastructure required to discover connected Arduino-compatible devices and obtain information about their serial interfaces.

This is an important dependency for the overall Arduino CLI workflow because Arduino CLI needs reliable access to serial devices when performing board discovery and upload operations.

---
## Build Challenges


- FreeBSD-specific serial-device handling
    
- Go module and dependency compatibility
    
- Correct installation paths
    
- Port staging and packaging
    
- Integration with the existing Arduino tooling
    
- Validation against FreeBSD device nodes  

The work establishes Serial Discovery as another native component of the Arduino ecosystem on FreeBSD, reducing the dependency on platform-specific implementations designed primarily around Linux and macOS.

---
### Arduino ESP32 Platform

Arduino ESP32 support provides the platform definitions, tools and board support required to develop and upload Arduino sketches to ESP32-based development boards.

Porting this component is an important part of the project because ESP32 is one of the most widely used Arduino-compatible platforms for IoT, embedded systems and wireless development.

The work focused on making the Arduino ESP32 platform available within the FreeBSD-based Arduino development workflow.

---
## Platform Integration

The ESP32 platform depends on a collection of board definitions, compiler/toolchain components, libraries and supporting utilities.

The porting work therefore required more than simply compiling a single executable. The platform had to be integrated into the Arduino ecosystem so that it could be discovered, installed and used through the Arduino CLI workflow.

The intended workflow is:

```text
FreeBSD
   │
   ├── Arduino CLI
   │
   ├── Arduino ESP32 Platform
   │       ├── Board definitions
   │       ├── ESP32 tools
   │       ├── Libraries
   │       └── Compiler/toolchain
   │
   └── Serial Discovery
           │
           └── Connected ESP32 board
```

This allows FreeBSD to function as a complete development environment rather than only providing the Arduino CLI executable.

---
## Build Challenges

The ESP32 platform introduced additional complexity because of its dependency on multiple platform-specific tools and packages.

The major areas requiring attention included:

- Platform package dependencies
    
- Toolchain compatibility
    
- FreeBSD host compatibility
    
- Installation and staging paths
    
- Board package layout
    
- Integration with Arduino CLI
    
- Serial-port access
    
- Compatibility between the ESP32 platform tools and the FreeBSD environment
    

The ESP32 work therefore represents an important step toward supporting real-world Arduino-compatible hardware on FreeBSD rather than limiting the contribution to CLI functionality alone.

---

# Arduino AVR Core

Link: [https://reviews.freebsd.org/D59114](https://reviews.freebsd.org/D59114)

The Arduino AVR Core provides the core software required to compile and develop sketches for classic Arduino AVR-based boards, including boards such as the Arduino Uno and other boards based on AVR microcontrollers.

As part of the GSoC work, the Arduino AVR Core was packaged for FreeBSD so that AVR-based Arduino development could be integrated into the native FreeBSD Ports ecosystem.

The contribution adds the AVR core as a FreeBSD port, providing the platform-specific source files, build configuration and metadata required by Arduino CLI to use AVR-based boards on FreeBSD.

The port is particularly important for the Arduino Uno workflow because the AVR core contains the board-specific compilation infrastructure required to transform an Arduino sketch into firmware that can be uploaded to an AVR-based Arduino board.

---

## FreeBSD Port

The Arduino AVR Core was added to the FreeBSD Ports Collection through:

```text
devel/arduino-avr-core
```

The port packages the Arduino AVR core and makes its files available through the standard FreeBSD ports/package infrastructure.

This allows the AVR platform support to be installed independently from Arduino CLI while still integrating naturally with the Arduino CLI board-management workflow.

---

## AVR Platform Support

The AVR core provides the platform-specific components required when compiling Arduino sketches for AVR-based boards.

For an Arduino Uno workflow, the core is used alongside the AVR compiler toolchain to perform the following process:

```text
Arduino Sketch
      ↓
Arduino CLI
      ↓
Arduino AVR Core
      ↓
AVR Toolchain
      ↓
AVR Firmware
      ↓
Arduino Uno
```

This makes the AVR core an essential component of the complete Arduino development environment.

---

## FreeBSD Integration

Packaging the AVR core was necessary to make the Arduino ecosystem usable through FreeBSD's native packaging infrastructure.

Instead of requiring users to manually download Arduino's AVR platform files, the required core can be provided through the FreeBSD Ports Collection.

This also allows Arduino CLI and the AVR core to be installed and managed using the same package-management mechanisms as other FreeBSD software.

The contribution therefore complements the Arduino CLI port by providing one of the primary hardware platforms that Arduino CLI needs to support.

---

## Build and Packaging

The port required the Arduino AVR Core sources and associated metadata to be adapted to the structure expected by the FreeBSD Ports Collection.

The work included defining the port metadata, source distribution information, dependencies and installation paths required for the AVR core to be correctly staged and packaged.

The resulting package can then be consumed by the Arduino development environment when an AVR-based board is selected.

This establishes the FreeBSD-native packaging layer required for compiling Arduino sketches for AVR hardware.

---

## Arduino Uno Workflow

One of the main practical targets of the AVR core integration was the Arduino Uno.

With the Arduino CLI, AVR core and required compiler toolchain available on FreeBSD, the development workflow becomes:

```text
Sketch
  ↓
arduino-cli compile
  ↓
Arduino AVR Core
  ↓
avr-gcc / AVR toolchain
  ↓
Compiled firmware
  ↓
arduino-cli upload
  ↓
Arduino Uno
```

This was an important milestone for the overall GSoC project because it demonstrated that FreeBSD could support the complete development pipeline for a commonly used Arduino board.

---

## Role in the Overall Project

The Arduino CLI port provides the command-line interface, while the AVR Core provides the hardware-platform implementation required to compile programs for AVR boards.

Together they form the foundation for native Arduino development on FreeBSD:

```text
FreeBSD
   │
   ├── Arduino CLI
   │
   ├── Arduino AVR Core
   │
   ├── AVR Toolchain
   │
   └── Serial/USB Support
           │
           ↓
       Arduino Uno
```

This contribution therefore extends the Arduino CLI port from simply having the CLI executable available to actually supporting a complete AVR-based Arduino development workflow on FreeBSD.

---

## Result

Together, the Arduino CLI, Arduino Language Server, Arduino Serial Discovery and Arduino ESP32 work form the foundation of a broader Arduino development ecosystem on FreeBSD:

```text
                    Arduino Development on FreeBSD
                               │
                ┌──────────────┴──────────────┐
                │                             │
          Arduino CLI                Arduino Language Server
                │                             │
        ┌───────┴────────┐              Editor / IDE Support
        │                │
 Serial Discovery    Platform Support
        │                │
        │             Arduino ESP32
        │                │
        └─────────┬──────┘
                  │
          Physical Arduino /
             ESP32 Hardware
```

The overall contribution therefore moves the project toward a **complete, native Arduino development workflow on FreeBSD**, covering tooling, language support, device discovery, board platforms and hardware communication rather than treating Arduino CLI as an isolated package.
