# usb-descriptors

[![PyPI](https://img.shields.io/pypi/v/usb-descriptors)](https://pypi.org/project/usb-descriptors/)
[![Python](https://img.shields.io/pypi/pyversions/usb-descriptors)](https://pypi.org/project/usb-descriptors/)
[![License](https://img.shields.io/pypi/l/usb-descriptors)](LICENSE.txt)

Python library for parsing, building, and manipulating USB descriptors.
Covers standard USB descriptors as well as class-specific descriptors
for **Audio (UAC 1.0/2.0/3.0)**, **MIDI (1.0/2.0)**, **CDC**, and
**Microsoft OS 1.0** descriptors.

Fork of [python-usb-protocol](https://github.com/greatscottgadgets/python-usb-protocol)
with fixes for USB Audio/MIDI class descriptor implementations.

## Installation

```bash
pip install usb-descriptors
```

Requires Python ≥ 3.9. The only runtime dependency is
[construct](https://construct.readthedocs.io/).

## Quick Start

### Parse a binary descriptor

```python
from usb_protocol.types.descriptors import StringDescriptor

raw = bytes([40, 3, ord('G'), 0, ord('S'), 0, ord('G'), 0])  # "GSG"
parsed = StringDescriptor.parse(raw)
print(parsed.bString)  # "GSG"
```

### Build descriptors from scratch

```python
from usb_protocol.emitters.descriptors import DeviceDescriptorEmitter

d = DeviceDescriptorEmitter()
d.idVendor  = 0x1234
d.idProduct = 0xabcd
d.bNumConfigurations = 1
print(d.emit())  # raw bytes
```

### Assemble a complete device descriptor collection

```python
from usb_protocol.emitters.descriptors import DeviceDescriptorCollection

collection = DeviceDescriptorCollection()

with collection.DeviceDescriptor() as d:
    d.idVendor  = 0xcafe
    d.idProduct = 0x0001
    d.bNumConfigurations = 1
    d.iManufacturer = "Acme Inc."
    d.iProduct      = "USB Widget"

with collection.ConfigurationDescriptor() as c:
    with c.InterfaceDescriptor() as i:
        i.bInterfaceNumber = 0
        with i.EndpointDescriptor() as e:
            e.bEndpointAddress = 0x81

# Iterate over all generated descriptors
for type_num, index, raw in collection:
    print(f"type={type_num} index={index}: {raw.hex()}")
```

## Supported Descriptors

### Standard USB

| Descriptor | Parse | Build |
|---|---|---|
| Device | ✓ | ✓ |
| Configuration | ✓ | ✓ |
| Interface | ✓ | ✓ |
| Endpoint | ✓ | ✓ |
| String / String Language | ✓ | ✓ |
| Device Qualifier | ✓ | ✓ |
| Interface Association | ✓ | ✓ |
| BOS / Device Capability | ✓ | ✓ |
| SuperSpeed Endpoint Companion | ✓ | ✓ |

### Class-Specific

| Class | Parse | Build |
|---|---|---|
| **CDC** (Communications) | ✓ | ✓ |
| **MIDI 1.0** | ✓ | ✓ |
| **MIDI 2.0** | ✓ | ✓ |
| **UAC 1.0** (USB Audio) | ✓ | ✓ |
| **UAC 2.0** (USB Audio) | ✓ | ✓ |
| **UAC 3.0** (USB Audio) | ✓ | ✓ |
| **Microsoft OS 1.0** | ✓ | ✓ |

### Additional Types

The `usb_protocol.types` module provides comprehensive enumerations:

- `USBDirection` — IN/OUT directions
- `USBRequestType` — Standard, Class, Vendor request types
- `USBRequestRecipient` — Device, Interface, Endpoint recipients
- `LanguageIDs` — USB language identifier codes
- `StandardRequestCodes` — GET_DESCRIPTOR, SET_ADDRESS, etc.
- `StandardFeatureSelectors` — ENDPOINT_HALT, DEVICE_REMOTE_WAKEUP

## Usage Patterns

### Parsing Mode

All descriptor types support `.parse(bytes)` for reading raw USB data:

```python
from usb_protocol.types.descriptors.standard import DeviceDescriptor

raw = bytes.fromhex("1201000200000040cafebeef000001020001")
parsed = DeviceDescriptor.parse(raw)
print(f"VID={parsed.idVendor:04x} PID={parsed.idProduct:04x}")
```

Partial descriptors (incomplete binary data) are supported via `.Partial`:

```python
from usb_protocol.types.descriptors import standard

# Works even with truncated data
partial = standard.DeviceDescriptor.Partial.parse(raw[:10])
```

### Builder Mode

Emitter classes provide a fluent API with sensible defaults:

```python
from usb_protocol.emitters.descriptors.standard import ConfigurationDescriptorEmitter

c = ConfigurationDescriptorEmitter()
c.bConfigurationValue = 1
c.bMaxPower = 100  # 200 mA

with c.InterfaceDescriptor() as i:
    i.bInterfaceNumber = 0
    i.bInterfaceClass  = 0xFF  # Vendor-specific

    with i.EndpointDescriptor() as e:
        e.bEndpointAddress = 0x81  # IN, endpoint 1
        e.bmAttributes     = 0x02  # Bulk
        e.wMaxPacketSize   = 512
```

Derived fields (`bNumEndpoints`, `wTotalLength`, `bNumInterfaces`) are
computed automatically — you don't need to set them.

### Interface Association Descriptors (IAD)

```python
with collection.ConfigurationDescriptor() as c:
    with c.InterfaceAssociationDescriptor() as ia:
        ia.bFirstInterface    = 0
        ia.bInterfaceCount    = 2
        ia.bFunctionClass     = 0x01  # Audio
        ia.bFunctionSubClass  = 0x01
        ia.bFunctionProtocol  = 0x00
        ia.iFunction          = "USB Audio Function"

    with c.InterfaceDescriptor() as i0:
        i0.bInterfaceNumber = 0
        # ...

    with c.InterfaceDescriptor() as i1:
        i1.bInterfaceNumber = 1
        # ...
```

## Differences from `python-usb-protocol`

This fork fixes and extends the upstream library:

- **Fixed** USB Audio Class 1.0/2.0 descriptor implementations
  (broken constructs and missing imports in upstream)
- **Fixed** MIDI 1.0 descriptor types and emitters
- **Added** `iJack` field support for MIDI Out Jack descriptors
- **Added** UAC 2.0 `ClockSelector` descriptor
- Added UAC 1.0/2.0/3.0 emitter classes

## Development

```bash
git clone https://github.com/hansfbaier/python-usb-descriptors.git
cd python-usb-descriptors
pip install -e ".[dev]"

# Run tests
python -m unittest discover -t . -v
```

## License

BSD — see [LICENSE.txt](LICENSE.txt).
