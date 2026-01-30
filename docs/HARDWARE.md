# Hardware Compatibility Guide

This document lists all devices supported by ATC_MiThermometer firmware.

## Supported Devices

### Xiaomi Devices

#### LYWSD03MMC

| Hardware Version | ID | Sensor | LCD | I2C Addr |
|-----------------|-----|--------|-----|----------|
| B1.4 | 0 | SHTC3 | Yes | 0x3C |
| B1.5 | 10 | SHTC3 | Yes | 0x3C |
| B1.6 | 4 | SHT4x | Yes | 0x3E |
| B1.7 | 5 | SHT4x | Yes | 0x3E |
| B1.9 | 3 | SHT4x | Yes | 0x3E |
| NB16 | 14 | SHT4x | Yes | - |

**Features:**
- Segment LCD display
- CR2032 battery
- Temperature/humidity sensor
- Reset button

#### MHO-C401

| Hardware Version | ID | Sensor | Display |
|-----------------|-----|--------|---------|
| 2020 | 1 | SHTC3 | E-ink |
| 2022 (N) | 8 | SHTC3 | E-ink |

**Features:**
- E-ink display (low power)
- CR2032 battery
- Key button
- Reed switch support on key pin

#### MHO-C122

| Hardware Version | ID | Sensor | Display |
|-----------------|-----|--------|---------|
| Standard | 11 | SHTC3 | LCD |

**Features:**
- Segment LCD display
- CR2032 battery

#### MJWSD05MMC

| Hardware Version | ID | Sensor | Display | Language |
|-----------------|-----|--------|---------|----------|
| Chinese | 9 | SHT4x | Large LCD | CN |
| English | 12 | SHT4x | Large LCD | EN |

**Features:**
- Large segment LCD with time display
- 2 × AAA batteries
- Multiple screen modes

#### MJWSD06MMC

| Hardware Version | ID | Sensor | Display |
|-----------------|-----|--------|---------|
| Standard | 13 | - | LCD |

### Qingping Devices

#### CGG1 (Qingping Temp & RH Monitor)

| Hardware Version | ID | Sensor | Display |
|-----------------|-----|--------|---------|
| 2020/2021 | 2 | SHTC3 | E-ink |
| 2022 | 7 | SHTC3 | E-ink |

**Features:**
- E-ink display
- CR2430 battery
- Original Xiaomi ecosystem device

#### CGDK2 (Qingping Temp & RH Monitor Lite)

| Hardware Version | ID | Sensor | Display |
|-----------------|-----|--------|---------|
| Standard | 6 | SHTC3 | LCD |

**Features:**
- Segment LCD display
- CR2032 battery
- Special battery display function

### Tuya/Zigbee Devices

#### TH03Z / ZTH03

| Device Type | ID | Sensor | Display | Protocol |
|-------------|-----|--------|---------|----------|
| TH03Z | 22 | Various | LCD | Zigbee |
| ZTH03 | 30 | Various | LCD | Zigbee |

#### LKTMZL02

| Hardware Version | ID | Sensor | Display | Battery |
|-----------------|-----|--------|---------|---------|
| Standard | 31 | Various | LCD | 2 × AAA |

#### ZTH05Z

| Hardware Version | ID | Sensor | Display | Battery |
|-----------------|-----|--------|---------|---------|
| Standard | 33 | AHT30 | LCD | CR2032 |

#### ZY-ZTH02 / ZY-ZTH02Pro

| Device | ID | Sensor | Display | Battery |
|--------|-----|--------|---------|---------|
| ZY-ZTH02 | 37 | SHT30/CHT832x | No | 2 × AAA |
| ZY-ZTH02Pro (ZTH01) | 38 | SHT30/CHT832x | LCD | 2 × AAA |

#### ZTH01 / ZTH02

| Device | ID | Sensor | Protocol |
|--------|-----|--------|----------|
| ZTH01 | 27 | Various | Zigbee |
| ZTH02 | 28 | Various | Zigbee |

#### ZG-227Z

| Hardware Version | ID | Sensor | Battery |
|-----------------|-----|--------|---------|
| Standard | 39 | AHT20 | CR2450 |

#### ZBEACON-TH01

| Version | ID | Sensor | Battery |
|---------|-----|--------|---------|
| v1.0 | 45 | SHT4x | 2 × AAA |
| v2.0 | 47 | SHT4x/G40 | 2 × AAA |

#### ZG-303Z

| Hardware Version | ID | Sensor | Features |
|-----------------|-----|--------|----------|
| Standard | 44 | AHT20 | Plant monitor |

#### ZB-MC

| Hardware Version | ID | Sensor | Battery |
|-----------------|-----|--------|---------|
| Standard | 46 | CHT8305 | 2 × AAA |

### DIY Devices

#### TB03F-Kit

| Device Type | ID | Sensor Options | Features |
|-------------|-----|----------------|----------|
| Standard | 16 | INA226, MY18B20 | Development kit |

#### TNK01

| Device Type | ID | Features |
|-------------|-----|----------|
| PB-03F module | 18 | Water tank controller |

#### TS0201

| Device Type | ID | Sensor | Protocol |
|-------------|-----|--------|----------|
| IH-K009 analog | 17 | Various | Zigbee |

#### PLM1

| Device Type | ID | Features |
|-------------|-----|----------|
| ECF-SGS01-A | 29 | Plant monitor, BT3L Tuya module |

---

## Hardware Version Detection

The firmware automatically detects hardware version based on:

1. **LCD Controller Address**
   - 0x3C → LYWSD03MMC B1.4/B1.5
   - 0x3E → LYWSD03MMC B1.6+, CGDK2, etc.

2. **Sensor Type**
   - SHTC3 → Older hardware
   - SHT4x → Newer hardware

3. **GPIO Configuration**
   - Button/key pins
   - Reed switch pins
   - Display interface

### Hardware Version IDs

```c
enum HW_VERSION_ID {
    HW_VER_LYWSD03MMC_B14 = 0,  // SHTV3
    HW_VER_MHO_C401 = 1,        // SHTV3
    HW_VER_CGG1 = 2,            // SHTV3
    HW_VER_LYWSD03MMC_B19 = 3,  // SHT4x
    HW_VER_LYWSD03MMC_B16 = 4,  // SHT4x
    HW_VER_LYWSD03MMC_B17 = 5,  // SHT4x
    HW_VER_CGDK2 = 6,           // SHTV3
    HW_VER_CGG1_2022 = 7,       // SHTV3
    HW_VER_MHO_C401_2022 = 8,   // SHTV3
    HW_VER_MJWSD05MMC = 9,      // SHT4x
    HW_VER_LYWSD03MMC_B15 = 10, // SHTV3
    HW_VER_MHO_C122 = 11,       // SHTV3
    HW_VER_MJWSD05MMC_EN = 12,  // SHT4x
    HW_VER_MJWSD06MMC = 13,
    HW_VER_LYWSD03MMC_NB16 = 14,
    HW_VER_EXTENDED = 15        // DIY devices
};
```

---

## Device Services

Each device supports different services based on hardware capabilities:

```c
#define SERVICE_OTA         0x00000001  // OTA update
#define SERVICE_OTA_EXT     0x00000002  // Extended OTA
#define SERVICE_PINCODE     0x00000004  // PIN code support
#define SERVICE_BINDKEY     0x00000008  // Encryption support
#define SERVICE_HISTORY     0x00000010  // Flash logger
#define SERVICE_SCREEN      0x00000020  // LCD/E-ink display
#define SERVICE_LE_LR       0x00000040  // Long Range support
#define SERVICE_THS         0x00000080  // T&H sensor
#define SERVICE_RDS         0x00000100  // Reed switch
#define SERVICE_KEY         0x00000200  // Button support
#define SERVICE_OUTS        0x00000400  // GPIO outputs
#define SERVICE_INS         0x00000800  // GPIO inputs
#define SERVICE_TIME_ADJUST 0x00001000  // Time correction
#define SERVICE_HARD_CLOCK  0x00002000  // Hardware RTC
#define SERVICE_TH_TRG      0x00004000  // T&H triggers
#define SERVICE_LED         0x00008000  // LED support
#define SERVICE_MI_KEYS     0x00010000  // Mi keys support
#define SERVICE_PRESSURE    0x00020000  // Pressure sensor
#define SERVICE_18B20       0x00040000  // DS18B20 sensor
#define SERVICE_IUS         0x00080000  // Current/voltage
#define SERVICE_PLM         0x00100000  // Plant monitor
#define SERVICE_BUTTON      0x00200000  // Button device
#define SERVICE_FINDMY      0x00400000  // FindMy support
#define SERVICE_ZIGBEE      0x01000000  // Zigbee version
```

### Example: MHO-C401 Services

```c
#define DEV_SERVICES ( \
    SERVICE_OTA | \
    SERVICE_OTA_EXT | \
    SERVICE_PINCODE | \
    SERVICE_BINDKEY | \
    SERVICE_HISTORY | \
    SERVICE_SCREEN | \
    SERVICE_LE_LR | \
    SERVICE_THS | \
    SERVICE_RDS | \
    SERVICE_TIME_ADJUST | \
    SERVICE_TH_TRG | \
    SERVICE_MI_KEYS \
)
```

---

## GPIO Mapping

### LYWSD03MMC

| GPIO | Function |
|------|----------|
| PC2 | I2C SDA |
| PC3 | I2C SCL |
| PB6 | Key / Reed Switch |
| PB0 | Battery ADC |

### MHO-C401

| GPIO | Function |
|------|----------|
| PC2 | I2C SDA |
| PC3 | I2C SCL |
| PA5 | EPD Busy |
| PA6 | EPD Reset |
| PB6 | Key |
| PB7 | EPD SDA |
| PC4 | EPD SHD |
| PD2 | EPD CSB |
| PD7 | EPD SCL |
| PB0 | Battery ADC |

---

## Battery Information

| Battery Type | Voltage | Devices |
|--------------|---------|---------|
| CR2032 | 3.0V | LYWSD03MMC, MHO-C401, CGDK2, MHO-C122, ZTH05Z |
| CR2430 | 3.0V | CGG1 |
| CR2450 | 3.0V | ZG-227Z |
| 2 × AAA | 3.0V | MJWSD05MMC, LKTMZL02, ZBEACON-TH01, etc. |

### Battery Level Thresholds

```c
#define MAX_VBAT_MV     3000    // 100%
#define MIN_VBAT_MV     2200    // 0%
#define LOW_VBAT_MV     2800    // Low battery warning
#define END_VBAT_MV     2000    // Shutdown
```

---

## Chipset Information

All supported devices use the Telink TLSR825x series:

| Chip | Flash | RAM | Features |
|------|-------|-----|----------|
| TLSR8251F512 | 512KB | 32KB | BLE 5.0 |
| TLSR8253F512 | 512KB | 32KB | BLE 5.0 |
| TLSR8258F1K | 1MB | 64KB | BLE 5.0, Zigbee |

### Flash Layout

| Region | Address | Size | Purpose |
|--------|---------|------|---------|
| Firmware | 0x00000 | 256KB | Application code |
| OTA | 0x40000 | 256KB | OTA update area |
| Data | 0x76000 | 8KB | Configuration |
| Keys | 0x78000 | 4KB | Encryption keys |
| MAC | 0x76000 | 4KB | MAC address |

---

## Flashing Instructions

### Requirements

- USB to UART adapter (3.3V)
- TelinkFlasher web tool or command-line tools
- Device in OTA mode (hold button during power on)

### Connection

| Device Pin | USB Adapter |
|------------|-------------|
| VCC (3.3V) | VCC |
| GND | GND |
| SWS (PA7) | TX |
| - | RX (optional) |

### Web Flashing

1. Open TelinkFlasher web interface
2. Connect device via BLE
3. Select firmware file
4. Click "Start Flashing"

### Command Line

```bash
# Using TlsrPgm.py
python TlsrPgm.py -p /dev/ttyUSB0 wf firmware.bin
```

---

## Troubleshooting

### Device Not Detected

1. Verify battery is installed correctly
2. Check battery voltage (>2.0V)
3. Try holding reset button during battery insertion
4. Verify correct firmware for device type

### BLE Connection Issues

1. Reset device (remove/insert battery)
2. Clear BLE bonding on phone/computer
3. Move closer to device
4. Try different BLE adapter

### Display Issues

1. Verify correct hardware version detected
2. Check I2C connection (for LCD devices)
3. Update to latest firmware
4. For E-ink: wait for full refresh cycle

### Sensor Reading Errors

1. Check sensor I2C address
2. Verify sensor is detected at boot
3. Check calibration coefficients
4. Sensor may need replacement
