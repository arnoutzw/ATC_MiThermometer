# BLE Advertising Formats

This document describes the BLE advertising formats supported by ATC_MiThermometer firmware.

## Overview

The firmware supports four advertising formats, selectable via configuration:

| Format | ID | UUID | Description |
|--------|-----|------|-------------|
| ATC1441 | 0 | 0x181A | Legacy format, mixed endianness |
| PVVX Custom | 1 | 0x181A | Enhanced format with more data |
| Xiaomi Mi | 2 | 0xFE95 | Xiaomi Mijia protocol |
| BTHome v2 | 3 | 0xFCD2 | Home Assistant native (default) |

All formats support optional encryption using a 16-byte bindkey.

---

## BTHome v2 Format (Recommended)

**Service UUID:** 0xFCD2

BTHome v2 is the default and recommended format for Home Assistant integration.

### Packet Structure

```
+--------+------+-------+------+------+--------+
| Size   | Type | UUID  | Info | PID  | Data   |
| 1 byte | 0x16 | FCD2  | 1    | 2    | N bytes|
+--------+------+-------+------+------+--------+
```

### Info Byte

| Bit | Description |
|-----|-------------|
| 0-4 | Version (2 for BTHome v2) |
| 5 | Trigger based device |
| 6 | Encryption (0=clear, 1=encrypted) |
| 7 | Reserved |

- **0x40**: Unencrypted (version 2)
- **0x41**: Encrypted (version 2)

### Object IDs (Sensor Types)

| ID | Type | Name | Unit | Size |
|----|------|------|------|------|
| 0x00 | uint8 | Packet ID | - | 1 |
| 0x01 | uint8 | Battery | % | 1 |
| 0x02 | int16 | Temperature | 0.01°C | 2 |
| 0x03 | uint16 | Humidity | 0.01% | 2 |
| 0x04 | uint24 | Pressure | 0.01 hPa | 3 |
| 0x05 | uint24 | Illuminance | 0.01 lux | 3 |
| 0x06 | uint16 | Weight | 0.01 kg | 2 |
| 0x08 | int16 | Dew Point | 0.01°C | 2 |
| 0x09 | uint8 | Count (8-bit) | - | 1 |
| 0x0A | uint24 | Energy | 0.001 kWh | 3 |
| 0x0B | uint24 | Power | 0.01 W | 3 |
| 0x0C | uint16 | Voltage | 0.001 V | 2 |
| 0x10 | uint8 | Switch | 0/1 | 1 |
| 0x11 | uint8 | Opened | 0/1 | 1 |
| 0x12 | uint16 | CO2 | ppm | 2 |
| 0x13 | uint16 | TVOC | - | 2 |
| 0x3A | uint8 | Button | event | 1 |
| 0x3D | uint16 | Count (16-bit) | - | 2 |
| 0x3E | uint32 | Count (32-bit) | - | 4 |
| 0x43 | uint16 | Current | 0.001 A | 2 |
| 0x45 | int16 | Temperature | 0.1°C | 2 |
| 0x4A | uint16 | Voltage | 0.1 V | 2 |
| 0x5D | int16 | Current (signed) | 0.001 A | 2 |

### Unencrypted Data Packet Example

```
Byte:  0    1    2    3    4    5    6    7    8    9   10   11   12   13
Data: [sz] [16] [D2] [FC] [40] [00] [xx] [01] [bb] [02] [tt] [tt] [03] [hh] [hh]
       |    |    UUID     |info |pid |    |bat%|    | temp x0.01  |  humi x0.01
       |    Service Data type
       Size
```

### Encrypted Packet Structure

```
+--------+------+-------+------+-----------+--------+------+
| Size   | Type | UUID  | Info | Encrypted | Count  | MIC  |
| 1 byte | 0x16 | FCD2  | 0x41 | N bytes   | 4 bytes| 4 bytes|
+--------+------+-------+------+-----------+--------+------+
```

### Encryption Details

- **Algorithm:** AES-128-CCM
- **Key:** 16-byte bindkey
- **Nonce:** MAC (6) + UUID16 (2) + Info (1) + Counter (4) = 13 bytes
- **MIC Length:** 4 bytes

---

## PVVX Custom Format

**Service UUID:** 0x181A (Environmental Sensing)

Enhanced format with full precision sensor data.

### Unencrypted Structure (18 bytes)

```c
typedef struct {
    u8  size;           // 0: = 18
    u8  uid;            // 1: = 0x16 (Service Data)
    u16 UUID;           // 2-3: = 0x181A
    u8  MAC[6];         // 4-9: MAC address (LSB first)
    s16 temperature;    // 10-11: x 0.01°C
    u16 humidity;       // 12-13: x 0.01%
    u16 battery_mv;     // 14-15: Battery voltage mV
    u8  battery_level;  // 16: 0-100%
    u8  counter;        // 17: Measurement counter
    u8  flags;          // 18: Trigger flags
} adv_custom_t;
```

### Flags Byte

| Bit | Description |
|-----|-------------|
| 0 | Reed switch 1 state |
| 1 | Trigger output state |
| 2 | Trigger active |
| 3 | Temperature trigger fired |
| 4 | Humidity trigger fired |
| 5 | Key pressed |
| 6 | Reed switch 2 state |
| 7 | Reserved |

### Encrypted Structure (15 bytes)

```c
typedef struct {
    u8  size;           // 0: = 11
    u8  uid;            // 1: = 0x16
    u16 UUID;           // 2-3: = 0x181A
    u8  counter;        // 4: Frame counter
    // --- Encrypted data (6 bytes) ---
    s16 temp;           // 5-6: Temperature x0.01°C
    u16 humi;           // 7-8: Humidity x0.01%
    u8  bat;            // 9: Battery level %
    u8  trg;            // 10: Trigger flags
    // --- End encrypted ---
    u8  mic[4];         // 11-14: Authentication tag
} adv_cust_enc_t;
```

---

## ATC1441 Format (Legacy)

**Service UUID:** 0x181A (Environmental Sensing)

Original format by ATC1441, uses mixed endianness for compatibility.

### Unencrypted Structure (16 bytes)

```c
typedef struct {
    u8  size;           // 0: = 16
    u8  uid;            // 1: = 0x16
    u16 UUID;           // 2-3: = 0x181A (little-endian)
    u8  MAC[6];         // 4-9: MAC address (BIG-ENDIAN!)
    u8  temperature[2]; // 10-11: x 0.1°C (BIG-ENDIAN!)
    u8  humidity;       // 12: x 1%
    u8  battery_level;  // 13: 0-100%
    u8  battery_mv[2];  // 14-15: mV (BIG-ENDIAN!)
    u8  counter;        // 16: Frame counter
} adv_atc1441_t;
```

**Note:** MAC, temperature, and battery voltage are big-endian, while UUID is little-endian.

### Encrypted Structure (11 bytes)

```c
typedef struct {
    u8  size;           // 0: = 8
    u8  uid;            // 1: = 0x16
    u16 UUID;           // 2-3: = 0x181A
    u8  counter;        // 4: Frame counter
    // --- Encrypted data (3 bytes) ---
    u8  temp;           // 5: Temperature (compressed)
    u8  humi;           // 6: Humidity (compressed)
    u8  bat;            // 7: Battery level
    // --- End encrypted ---
    u8  mic[4];         // 8-11: Authentication tag
} adv_atc_enc_t;
```

**Temperature compression:** Range -40°C to +87°C with 0.5°C resolution.

---

## Xiaomi Mi Format

**Service UUID:** 0xFE95 (Xiaomi Inc.)

Compatible with Xiaomi Mi Home app and devices.

### Frame Control Word

```c
typedef union {
    struct {
        u16 Factory         : 1;  // Reserved
        u16 Connected       : 1;  // Reserved
        u16 Central         : 1;  // Keep
        u16 isEncrypted     : 1;  // 0=clear, 1=encrypted
        u16 MACInclude      : 1;  // 0=no MAC, 1=MAC included
        u16 CapabilityInclude : 1; // 0=no capability, 1=included
        u16 ObjectInclude   : 1;  // 0=no object, 1=data included
        u16 Mesh            : 1;  // 0=standard BLE, 1=mesh
        u16 registered      : 1;  // 0=unbound, 1=bound
        u16 solicited       : 1;  // 0=no, 1=request binding
        u16 AuthMode        : 2;  // 0=old, 1=safety, 2=standard, 3=reserved
        u16 version         : 4;  // Protocol version (5)
    } bit;
    u16 word;
} adv_mi_fctrl_t;
```

### Header Structure

```c
typedef struct {
    u8  size;           // Packet size
    u8  uid;            // 0x16 (Service Data)
    u16 UUID;           // 0xFE95 (Xiaomi)
    adv_mi_fctrl_t fctrl; // Frame control
    u16 dev_id;         // Device type ID
    u8  counter;        // Frame counter (0-255)
} adv_mi_head_t;
```

### Device Type IDs

| ID | Device |
|----|--------|
| 0x055B | LYWSD03MMC |
| 0x0387 | MHO-C401 |
| 0x0347 | CGG1 |
| 0x0B48 | CGG1 (encrypted) |
| 0x066F | CGDK2 |
| 0x2832 | MJWSD05MMC |
| 0x55B5 | MJWSD06MMC |

### Data Object Types

| ID | Name | Size | Description |
|----|------|------|-------------|
| 0x1004 | Temperature | 2 | x0.1°C (int16) |
| 0x1006 | Humidity | 2 | x0.1% (uint16) |
| 0x100A | Battery | 1 | 0-100% (uint8) |
| 0x100D | Temp+Humi | 4 | Combined (int16+uint16) |
| 0x1019 | Door Sensor | 1 | Open/closed (uint8) |

### Encrypted Packet

Includes 3-byte counter and 4-byte MIC after encrypted payload.

---

## Encryption (Bindkey)

All formats support optional AES-128-CCM encryption.

### Bindkey Setup

The bindkey is a 16-byte (128-bit) encryption key stored in flash memory.

**Generation:**
- Can be set via BLE command (CMD_ID_BKEY = 0x18)
- Auto-generated if not set
- Displayed in TelinkFlasher web interface

### CCM Parameters

| Parameter | Value |
|-----------|-------|
| Algorithm | AES-128-CCM |
| Key Length | 16 bytes |
| Nonce Length | 12-13 bytes (format dependent) |
| Tag Length | 4 bytes |
| AAD | None or header bytes |

### Nonce Construction by Format

**BTHome v2:**
```
MAC[6] + UUID16[2] + Info[1] + Counter[4] = 13 bytes
```

**PVVX Custom:**
```
MAC_reversed[6] + Header[4] + Counter[1] = 11 bytes
```

**Mi Format:**
```
MAC_reversed[6] + DevID[2] + FrameCnt[1] + CountID[3] = 12 bytes
```

---

## Advertising Intervals

| Setting | Range | Default |
|---------|-------|---------|
| Base Interval | 62.5ms - 10s | 2.5s |
| Random Delay | 0-15 * 0.625ms | 10ms |
| Measure Interval | 2-25 × base | 4 × base |

### Formula

```
advertising_interval_ms = cfg.advertising_interval * 62.5
measure_interval_ms = advertising_interval_ms * cfg.measure_interval
```

---

## Extension Advertising (BT5.0)

The firmware supports BLE 5.0 extension advertising:

| Mode | Description |
|------|-------------|
| Legacy | Standard BLE 4.x advertising |
| LE Coded (Long Range) | PHY Coded for extended range |
| LE 1M | Extension advertising at 1 Mbps |

Enable via `cfg.flg2.longrange` and `cfg.flg2.bt5phy` configuration flags.

---

## Implementation Notes

### Selecting Format

```c
// In configuration
cfg.flg.advertising_type = ADV_TYPE_BTHOME; // 0-3
```

### Enabling Encryption

```c
cfg.flg2.adv_crypto = 1;  // Enable encrypted advertising
```

### Frame Counter

- 32-bit counter incremented with each advertisement
- Used as nonce component for encryption
- Prevents replay attacks
- Reset on device reboot

---

## References

- [BTHome v2 Specification](https://bthome.io/format/)
- [Xiaomi BLE Protocol](https://github.com/pvvx/ATC_MiThermometer/tree/master/InfoMijiaBLE)
- [Bluetooth Assigned Numbers](https://www.bluetooth.com/specifications/assigned-numbers/)
