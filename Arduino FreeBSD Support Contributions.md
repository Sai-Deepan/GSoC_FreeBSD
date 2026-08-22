## Overview

As part of the effort to make the Arduino development ecosystem fully usable on FreeBSD, I contributed FreeBSD-specific implementations and fixes across multiple Arduino projects and supporting libraries.

The work focused on three important layers of the Arduino toolchain:

1. **Serial device discovery** — enabling Arduino's serial-discovery component to operate correctly on FreeBSD.
    
2. **Firmware/tool downloading** — making Arduino's firmware uploader test suite behave correctly on platforms where a compatible prebuilt tool binary is unavailable.
    
3. **Low-level USB serial enumeration** — implementing native FreeBSD USB serial-port discovery in `go-serial`, including device metadata extraction and USB descriptor probing.

Together, these changes address a major gap in the Arduino ecosystem on FreeBSD: the operating system can now participate in the same serial-device discovery and metadata workflow used by other supported platforms, while platform-specific download-tool tests no longer incorrectly fail simply because a FreeBSD binary has not been published.

The most important contribution was the implementation of **native FreeBSD USB serial enumeration in `go-serial`**. Prior to this work, the FreeBSD implementation consisted of a TODO stub that unconditionally returned an enumeration error. The new implementation replaces that stub with a complete FreeBSD-native discovery mechanism using `sysctl`, `/dev` device enumeration, `usbconfig`, and the FreeBSD USB device hierarchy.

---

## 1. FreeBSD START_SYNC Support in `arduino/serial-discovery`

### Pull Request

**Added FreeBSD START_SYNC implementation — PR #122**
[https://github.com/arduino/serial-discovery/pull/122](https://github.com/arduino/serial-discovery/pull/122)

**Status: Merged and accepted**

The first contribution addressed synchronization support in Arduino's `serial-discovery` component.

FreeBSD provides the **kqueue** event notification facility, which is closely related to the mechanism already used by the Darwin implementation.

Instead of introducing a completely independent FreeBSD implementation, the existing Darwin implementation was generalized so that the same BSD-specific implementation could serve both operating systems.

### Generalizing the Darwin implementation

The existing:

```text
sync_darwin.go
```

implementation was generalized into:

```text
sync_bsd.go
```

and given a build constraint covering both Darwin and FreeBSD:

```go
//go:build darwin || freebsd
```

The fallback implementation was also updated so that FreeBSD would no longer incorrectly select the generic implementation:

```go
//go:build !linux && !windows && !darwin && !freebsd
```

Rather than maintaining two nearly identical implementations of the same BSD-style mechanism, the code now recognizes the common functionality shared by the platforms.

### START_SYNC functionality

The resulting implementation enables `START_SYNC` functionality on FreeBSD by reusing the existing BSD `kqueue` mechanism.

This means FreeBSD can participate in the same synchronization workflow used for serial-device state changes without introducing a separate polling-based mechanism.

The change was validated by:

- Building the project successfully.
    
- Running the project's test suite.
    
- Verifying START_SYNC behavior on a FreeBSD environment.

The PR was subsequently reviewed by the Arduino maintainers, requested changes were addressed, and the pull request was ultimately merged into `arduino/serial-discovery`.

### Why this contribution matters

This change establishes the operating-system foundation required for reliable serial-device discovery on FreeBSD.

Without synchronization support, an application can potentially discover ports initially but lack an efficient mechanism for reacting to devices being connected or disconnected. By using FreeBSD's native `kqueue` facilities, the implementation integrates FreeBSD into the existing event-driven architecture instead of introducing platform-specific polling logic.

---

# 2. FreeBSD Compatibility Fixes in `arduino/arduino-fwuploader`

### Pull Request

**Fix nil pointer in TestDownloadTool and skip gracefully on unsupported OS/arch — PR #333**

[https://github.com/arduino/arduino-fwuploader/pull/333](https://github.com/arduino/arduino-fwuploader/pull/333)

**Status: Open / under review**

The second contribution addressed an issue encountered while building and testing Arduino's firmware uploader components on FreeBSD.

The problem was not that the firmware uploader itself was fundamentally incapable of running on FreeBSD. Instead, the test suite assumed that a compatible prebuilt tool flavor would always exist for the host operating system and architecture.

That assumption is not valid for FreeBSD because Arduino's published tool packages do not necessarily contain FreeBSD-specific binaries for every tool.

Consequently, a test that attempted to download a tool could fail on FreeBSD even though the failure represented an **absence of a published platform binary rather than a software defect**.

### Fixing the test fixture

The test previously constructed an Arduino tool without its associated package information.

The patch adds the missing:

```go
Package: &cores.Package{
    Name: "arduino",
},
```

field.

This is important because `DownloadTool` uses the package information when resolving the tool's compatible flavor. Without the package information, the resolution process could result in a nil pointer dereference.

The PR therefore fixes an actual correctness issue in the test setup while making the platform behavior explicit.

### Graceful handling of unsupported tool flavors

The second part of the change detects the specific condition where no compatible tool binary exists for the current operating system and architecture.

The test now checks for the corresponding error:

```go
if err != nil && strings.Contains(err.Error(), "not available for this OS") {
    t.Skipf(
        "skipping: no tool flavor available for this platform (%s/%s)",
        runtime.GOOS,
        runtime.GOARCH,
    )
}
```

This changes the interpretation of the situation from:

```text
Test failure
```

to:

```text
Test skipped because the platform has no published tool flavor
```

That distinction is important for cross-platform development.

A test suite should fail when the implementation is broken. It should not fail merely because an external release artifact has not been published for the current platform.

### Result

The PR adds nine lines without changing the fundamental firmware uploader behavior. It specifically makes the test suite robust when executed on platforms such as FreeBSD where compatible tool binaries may not exist.

The resulting behavior makes FreeBSD development and continuous testing less dependent on assumptions derived from Linux, macOS, or Windows packaging.

---

# 3. Native FreeBSD USB Serial Enumeration in `bugst/go-serial`

### Pull Request

**Implement FreeBSD USB serial port enumeration — PR #228**

[https://github.com/bugst/go-serial/pull/228](https://github.com/bugst/go-serial/pull/228)

**Status: Open / under review**

This was the largest and most technically significant contribution.

Prior to this work, FreeBSD's detailed serial-port enumeration implementation was effectively unfinished.

The native FreeBSD function was only:

```go
func nativeGetDetailedPortsList(_ func(vid, pid string) bool) ([]*PortDetails, error) {
    // TODO
    return nil, &PortEnumerationError{}
}
```

In practice, this meant that FreeBSD could not provide the detailed USB metadata that the library already provided on platforms such as Linux and Darwin.

The contribution replaces that stub with a complete native implementation.

The PR changes two files and adds approximately **264 lines of implementation

---

## 3.1 Enumerating FreeBSD serial devices

The implementation begins by obtaining the available serial ports using the existing serial-port enumeration infrastructure:

```go
ports, err := serial.GetPortsList()
```

The implementation then examines the returned device names and identifies FreeBSD USB serial devices.

FreeBSD exposes USB serial ports using names such as:

```text
/dev/cuaU0
/dev/ttyU0
```

The implementation therefore specifically recognizes:

```text
cuaU<number>
ttyU<number>
```

rather than relying on the generic Unix naming patterns used by Linux or other operating systems.

---

## 3.2 Correcting the FreeBSD port-name filter

The existing FreeBSD port filter was also incorrect.

The old pattern was:

```go
^(cu|tty)\..*
```

which does not match real FreeBSD serial devices such as:

```text
cuaU0
ttyU0
```

The implementation replaces it with:

```go
^(cua[uU]|tty[uU])[0-9]+$
```

This allows the enumeration system to identify both USB-attached serial devices and the relevant FreeBSD naming variations while avoiding unrelated `.init` and `.lock` device entries.

The PR description explicitly identifies this as a correction to a pattern that previously failed to match real FreeBSD device names.

---

# 3.3 Extracting USB metadata through `sysctl`

One of the key technical challenges on FreeBSD is that the information required to construct a complete `PortDetails` object is distributed across different parts of the device hierarchy.

The implementation invokes:

```text
sysctl -a
```

and parses entries under:

```text
dev.*.%pnpinfo
dev.*.%location
```

The distinction is significant.

The `%pnpinfo` data provides information such as:

```text
vendor
product
ttyname
sernum
```

while `%location` provides the USB device location/ugen information.

The implementation correlates these two records by their common device-node path.

Conceptually, the process is:

```text
sysctl
  │
  ├── dev.X.%pnpinfo
  │      ├── ttyname
  │      ├── vendor
  │      ├── product
  │      └── sernum
  │
  └── dev.X.%location
         └── ugen
```

The implementation first builds an intermediate map indexed by the device node, then combines the information into a complete FreeBSD USB-device representation.

This is necessary because the USB device identifier (`ugen`) is not necessarily exposed in the same sysctl property that contains the VID/PID and serial-port information.

---

# 3.4 Constructing FreeBSD USB device records

The implementation introduces an internal representation:

```go
type freeBSDUSBDevice struct {
    VID           string
    PID           string
    SerialNumber  string
    ugen          string
    Manufacturer  string
    Product       string
    Configuration string
}
```

This creates a clear intermediate layer between the raw FreeBSD device tree and Arduino's generic `PortDetails` abstraction.

For a USB serial device, the resulting data can include:

```text
VID
PID
SerialNumber
Manufacturer
Product
Configuration
ugen
```

That metadata can then be returned through the same abstraction already used by other operating systems.

---

# 3.5 Normalizing VID and PID values

FreeBSD's device information does not necessarily present vendor and product IDs in precisely the same textual representation expected by the cross-platform API.

The implementation therefore introduces:

```go
func normalizeHex(value string) string
```

This function:

1. Removes an optional `0x` prefix.
    
2. Converts the hexadecimal identifier into a numeric value.
    
3. Formats it as a four-character uppercase hexadecimal identifier.
    
4. Falls back to uppercase text if parsing is unsuccessful.
    

This ensures that identifiers returned by FreeBSD follow a stable representation compatible with the rest of the serial-port enumeration system.

---

# 3.6 Active probing through `usbconfig`

The most important part of the detailed enumeration implementation is the active USB probing mechanism.

FreeBSD's kernel/device-tree metadata does not necessarily expose all descriptor strings needed by Arduino.

To fill those gaps, the implementation invokes the native FreeBSD utility:

```text
usbconfig
```

specifically:

```text
usbconfig -d <ugen> dump_device_desc
```

and:

```text
usbconfig -d <ugen> dump_curr_config_desc
```

These commands expose the USB descriptor information containing fields such as:

```text
iManufacturer
iProduct
iSerialNumber
iConfiguration
```

The implementation parses these fields and injects them into the generic `PortDetails` structure.

This allows FreeBSD to expose USB metadata in a manner consistent with the behavior already implemented for other supported operating systems.

The implementation also handles devices that explicitly report:

```text
no string
```

by treating those values as absent rather than returning misleading descriptor strings.

---

# 3.7 Conditional USB probing

USB descriptor probing is performed through the existing:

```go
shouldProbeUSB(vid, pid)
```

callback.

That allows the implementation to preserve the existing architecture of the library rather than unconditionally executing external commands for every device.

The general sequence becomes:

```text
Enumerate serial ports
        ↓
Identify FreeBSD USB serial ports
        ↓
Match device information
        ↓
Extract VID/PID/serial information
        ↓
Check whether USB probing is required
        ↓
Run usbconfig
        ↓
Extract descriptor strings
        ↓
Construct PortDetails
```

This keeps the implementation compatible with the existing API and probing model.

---

# 3.8 Handling multiple FreeBSD serial-port names

The implementation correctly handles both forms of USB serial device commonly exposed by FreeBSD:

```text
/dev/cuaU0
/dev/ttyU0
```

Both device names are mapped back to the same underlying USB device information.

This allows applications built on top of `go-serial` to obtain consistent metadata regardless of which FreeBSD serial device node they use.

The PR description specifically states that the implementation supports USB-attached `cuaU`/`ttyU` devices as well as the appropriate serial-port naming behavior on FreeBSD.

---

# 3.9 Real hardware validation

The implementation was validated against a **real CH340 USB-to-serial adapter on FreeBSD/amd64**.

This is particularly important because the implementation depends on FreeBSD's real device hierarchy and command-line utilities rather than purely synthetic unit-test data.

Testing confirmed that:

- The USB VID was resolved correctly.
    
- The USB PID was resolved correctly.
    
- The product string was resolved correctly.
    
- Manufacturer information correctly remained empty when the device descriptor reported no string.
    
- Serial-number information correctly remained empty when not available.
    
- Configuration information correctly remained empty when not available.
    
- The implementation worked for both:

```text
/dev/cuaU0
/dev/ttyU0
```

The reported values were cross-checked against the raw descriptor information produced by `usbconfig`, providing a direct validation of the parsing logic.

---

# 4. Why the `go-serial` Contribution Was the Major Milestone

The `go-serial` contribution represents a significantly deeper level of FreeBSD support than simply adding a build tag or modifying a test.

Before this contribution, the native FreeBSD detailed-port enumeration path was explicitly marked as:

```text
TODO
```

and returned an error unconditionally.

That meant the functionality required to discover and describe USB serial devices in detail had never been completed.

The contribution therefore implemented the entire FreeBSD-specific path from the operating-system device hierarchy to the cross-platform serial API:

```text
FreeBSD kernel/device information
            ↓
          sysctl
            ↓
  device-node correlation
            ↓
  VID / PID / tty / serial / ugen
            ↓
        usbconfig
            ↓
USB descriptor metadata
            ↓
      PortDetails API
            ↓
   Arduino serial tooling
```

This is fundamentally different from simply adapting existing code.

It required understanding how FreeBSD represents USB devices, identifying where each required field is exposed, correlating information from separate sysctl properties, invoking native FreeBSD tooling, parsing descriptor output, normalizing identifiers, and integrating all of that with the library's existing platform-independent abstraction.

The implementation consequently provides the missing native FreeBSD foundation required by higher-level Arduino components that depend on detailed serial-port discovery.

---

# 5. Combined Impact on the Arduino FreeBSD Ecosystem

These three contributions operate at different layers but collectively address the same larger objective: enabling FreeBSD to function as a first-class platform in the Arduino development workflow.

### `arduino/serial-discovery`

Added:

- Native FreeBSD START_SYNC support.
    
- BSD `kqueue` synchronization through a shared Darwin/FreeBSD implementation.
    
- Correct FreeBSD build selection.
    
- Event-driven serial-device synchronization.

**Result:** FreeBSD can participate in Arduino's serial-device synchronization system.

PR:  
[https://github.com/arduino/serial-discovery/pull/122](https://github.com/arduino/serial-discovery/pull/122)

The PR is merged into the Arduino project.

---

### `arduino/arduino-fwuploader`

Added:

- Missing package information required for proper tool resolution.
    
- Protection against a nil pointer during tool flavor resolution.
    
- Platform-aware handling of unavailable tool flavors.
    
- Graceful test skipping when no binary exists for the current OS/architecture.

**Result:** FreeBSD builds and tests no longer incorrectly report an unavailable published tool binary as a software failure.

PR:  
[https://github.com/arduino/arduino-fwuploader/pull/333](https://github.com/arduino/arduino-fwuploader/pull/333)

The PR currently remains open for review.

---

### `bugst/go-serial`

Added:

- Complete native FreeBSD detailed-port enumeration.
    
- FreeBSD USB serial-device detection.
    
- Correct `cuaU`/`ttyU` handling.
    
- `sysctl` parsing.
    
- Correlation of `%pnpinfo` and `%location`.
    
- VID/PID extraction.
    
- USB `ugen` resolution.
    
- VID/PID normalization.
    
- `usbconfig` descriptor probing.
    
- Manufacturer/Product/SerialNumber/Configuration extraction.
    
- Correct handling of absent USB descriptor strings.
    
- Hardware validation with a CH340 adapter on FreeBSD/amd64.

**Result:** FreeBSD gains a complete implementation of a functionality that was previously only a TODO stub.

PR:  
[https://github.com/bugst/go-serial/pull/228](https://github.com/bugst/go-serial/pull/228)

The current PR contains approximately 264 added lines across two files and replaces the previous unconditional error-returning implementation.

---

# 6. Overall Contribution

The combined work moves Arduino's FreeBSD support beyond simply making individual programs compile.

The goal was to make the underlying platform behave correctly at the operating-system integration layer.

The `go-serial` work is the largest contribution because it completed an entire FreeBSD subsystem that had previously remained a TODO, providing the low-level functionality necessary for higher-level Arduino tooling to discover and describe USB serial devices correctly on FreeBSD.

---

## Contribution Links

**Arduino Serial Discovery — FreeBSD START_SYNC**

[https://github.com/arduino/serial-discovery/pull/122](https://github.com/arduino/serial-discovery/pull/122)

**Arduino Firmware Uploader — FreeBSD-compatible download test handling**

[https://github.com/arduino/arduino-fwuploader/pull/333](https://github.com/arduino/arduino-fwuploader/pull/333)

**go-serial — Complete FreeBSD USB serial enumeration**

[https://github.com/bugst/go-serial/pull/228](https://github.com/bugst/go-serial/pull/228)