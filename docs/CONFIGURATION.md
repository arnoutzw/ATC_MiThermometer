# Device Configuration Guide

This document describes all configuration options for ATC_MiThermometer firmware.

## Configuration Structure

The device configuration is stored in flash memory and persists across reboots.

```c
typedef struct _cfg_t {
    struct flg { ... };     // Feature flags
    struct flg2 { ... };    // Extended flags
    struct flg3 { ... };    // Additional flags
    u8 event_adv_cnt;       // Event advertising count
    u8 advertising_interval;// Advertising interval
    u8 measure_interval;    // Measurement multiplier
    u8 rf_tx_power;         // TX power level
    u8 connect_latency;     // Connection latency
    u8 min_step_time_update_lcd; // LCD update rate
    u8 hw_ver;              // Hardware version (read-only)
    u8 averaging_measurements;   // Averaging count
} cfg_t;
```

---

## Configuration Flags (flg)

| Bit | Field | Description |
|-----|-------|-------------|
| 0-1 | `advertising_type` | 0=ATC1441, 1=PVVX, 2=Mi, 3=BTHome |
| 2 | `comfort_smiley` | Enable comfort indicator on display |
| 3 | `show_time_smile` | Show time instead of smiley (if clock enabled) |
| 4 | `temp_F_or_C` | Temperature unit: 0=Celsius, 1=Fahrenheit |
| 5 | `show_batt_enabled` | Show battery on display rotation |
| 6 | `tx_measures` | Send measurements while connected |
| 7 | `lp_measures` | Low power measurement mode |

### Advertising Type Values

| Value | Format | Description |
|-------|--------|-------------|
| 0 | ATC1441 | Legacy format (mixed endian) |
| 1 | PVVX Custom | Full precision custom format |
| 2 | Xiaomi Mi | Mi Home compatible |
| 3 | BTHome v2 | Home Assistant native (default) |

---

## Extended Flags (flg2)

| Bit | Field | Description |
|-----|-------|-------------|
| 0-2 | `smiley` | Smiley face type (0-7) |
| 3 | `adv_crypto` | Enable encrypted advertising |
| 4 | `adv_flags` | Include BLE advertising flags |
| 5 | `bt5phy` | Support BLE 5.0 All PHY |
| 6 | `longrange` | Enable LE Long Range mode |
| 7 | `screen_off` | Turn off display |

### Smiley Types (LYWSD03MMC)

| Value | Display |
|-------|---------|
| 0 | Off |
| 1 | ` ^_^ ` |
| 2 | ` -^- ` |
| 3 | ` ooo ` |
| 4 | `(   )` |
| 5 | `(^_^)` happy |
| 6 | `(-^-)` sad |
| 7 | `(ooo)` |

---

## Additional Flags (flg3)

| Bit | Field | Description |
|-----|-------|-------------|
| 0-3 | `adv_interval_delay` | Random delay 0-15 * 0.625ms |
| 4-5 | Reserved | - |
| 6 | `date_ddmm` | Date format DD/MM (MJWSD05MMC) |
| 7 | `not_day_of_week` | Hide day of week |

---

## Timing Parameters

### Advertising Interval

```
cfg.advertising_interval: 1-160
Actual interval = value × 62.5 ms
Range: 62.5 ms to 10 seconds
Default: 40 (2.5 seconds)
```

| Value | Interval |
|-------|----------|
| 1 | 62.5 ms |
| 16 | 1 second |
| 40 | 2.5 seconds |
| 80 | 5 seconds |
| 160 | 10 seconds |

### Measurement Interval

```
cfg.measure_interval: 2-25
Actual interval = advertising_interval × measure_interval
Default: 4
```

Example: With `advertising_interval=40` (2.5s) and `measure_interval=4`:
- Measurement every: 2.5 × 4 = 10 seconds
- Advertisement every: 2.5 seconds

### LCD Update Rate

```
cfg.min_step_time_update_lcd: 10-255
Actual rate = value × 50 ms
Range: 0.5 to 12.75 seconds
Default: 50 (2.5 seconds)
```

### Connection Latency

```
cfg.connect_latency: 0-255
Latency = (value + 1) × 20 ms
Range: 20 ms to 5.12 seconds
Default: 49 (1 second)
```

---

## RF TX Power

```
cfg.rf_tx_power: 130-191
```

| Value | Power | Description |
|-------|-------|-------------|
| 130 | -25.18 dBm | Minimum |
| 140 | -15.02 dBm | Very low |
| 150 | -8.78 dBm | Low |
| 160 | -4.34 dBm | Medium-low |
| 170 | -0.51 dBm | Medium |
| 180 | +1.88 dBm | Medium-high |
| 191 | +3.01 dBm | Maximum |

**Note:** Higher power = more range but shorter battery life.

---

## Averaging

```
cfg.averaging_measurements: 0-255
0 = Disabled (send each measurement)
1-255 = Number of measurements to average
```

When enabled, the firmware averages N measurements before advertising. This reduces noise but increases latency.

---

## Comfort Zone Parameters

```c
typedef struct {
    s16 t[2];   // Temperature [low, high] × 0.01°C
    u16 h[2];   // Humidity [low, high] × 0.01%
} scomfort_t;
```

### Default Values

| Parameter | Value |
|-----------|-------|
| Temp Low | 20.00°C (2000) |
| Temp High | 25.00°C (2500) |
| Humi Low | 40.00% (4000) |
| Humi High | 60.00% (6000) |

When temperature and humidity are within these ranges, the comfort smiley shows happy; otherwise sad.

---

## Trigger Configuration

```c
typedef struct {
    s16 temp_threshold;     // Temperature threshold × 0.01°C
    s16 humi_threshold;     // Humidity threshold × 0.01%
    s16 temp_hysteresis;    // Temperature hysteresis
    s16 humi_hysteresis;    // Humidity hysteresis
    u16 rds_time_report;    // Reed switch report interval (sec)
    rds_type_t rds;         // Reed switch configuration
    trigger_flg_t flg;      // Trigger output state
} trigger_t;
```

### Trigger Behavior

1. **Temperature Trigger**: Activates when temp crosses threshold
2. **Humidity Trigger**: Activates when humidity crosses threshold
3. **Hysteresis**: Prevents rapid on/off cycling

Example:
- Threshold: 25.00°C
- Hysteresis: 0.50°C
- Trigger ON when temp >= 25.00°C
- Trigger OFF when temp <= 24.50°C

---

## Reed Switch Configuration

```c
typedef struct {
    u8 type1 : 2;       // RS1 type (RDS_TYPES)
    u8 type2 : 2;       // RS2 type (RDS_TYPES)
    u8 rs1_invert : 1;  // Invert RS1 logic
    u8 rs2_invert : 1;  // Invert RS2 logic
} rds_type_t;
```

### Reed Switch Types

| Value | Type | Description |
|-------|------|-------------|
| 0 | RDS_NONE | Disabled |
| 1 | RDS_SWITCH | Binary switch (open/closed) |
| 2 | RDS_COUNTER | Pulse counter |
| 3 | RDS_CONNECT | Connect on trigger |

---

## External Display Data

For showing custom data on the LCD:

```c
typedef struct {
    s16 big_number;     // Main display × 0.1 (-995..19995)
    s16 small_number;   // Small display × 1 (-9..99)
    u16 vtime_sec;      // Validity time in seconds
    struct {
        u8 smiley     : 3;  // Smiley type (0-7)
        u8 percent_on : 1;  // Show % symbol
        u8 battery    : 1;  // Show battery
        u8 temp_symbol: 3;  // Temperature symbol
    } flg;
} external_data_t;
```

### Temperature Symbols

| Value | Display |
|-------|---------|
| 0 | (blank) |
| 1 | °Г |
| 2 | - |
| 3 | °F |
| 4 | _ |
| 5 | °C |
| 6 | = |
| 7 | °E |

---

## BLE Commands for Configuration

| Command | ID | Description |
|---------|-----|-------------|
| CMD_ID_CFG | 0x55 | Get/Set configuration |
| CMD_ID_CFG_DEF | 0x56 | Reset to defaults |
| CMD_ID_COMFORT | 0x20 | Get/Set comfort zone |
| CMD_ID_TRG | 0x44 | Get/Set trigger config |
| CMD_ID_EXTDATA | 0x22 | Set external display data |
| CMD_ID_UTC_TIME | 0x23 | Get/Set device time |
| CMD_ID_TADJUST | 0x24 | Get/Set time adjustment |

### Using Python Configuration Tool

```bash
# Read current configuration
python -m atc_mi_interface -m A4:C1:38:AA:BB:CC -c

# Open GUI editor
python -m atc_mi_interface -m A4:C1:38:AA:BB:CC -g

# Set date/time
python -m atc_mi_interface -m A4:C1:38:AA:BB:CC -D

# Reset to defaults
python -m atc_mi_interface -m A4:C1:38:AA:BB:CC -R
```

---

## Configuration Storage

- **Flash Address:** Device-specific
- **EEP Version:** 0x09 (minimum supported)
- **Persistence:** Survives power cycles
- **Reset:** CMD_ID_CFG_DEF or holding key during boot

---

## Hardware Version Detection

The firmware automatically detects hardware version based on:
- LCD controller I2C address
- Sensor type detection
- GPIO configuration

Hardware version is stored in `cfg.hw_ver` (read-only).

---

## Default Configuration

```c
const cfg_t def_cfg = {
    .flg.advertising_type = ADV_TYPE_BTHOME,
    .flg.comfort_smiley = 1,
    .flg.temp_F_or_C = 0,           // Celsius
    .flg.show_batt_enabled = 1,
    .flg.tx_measures = 0,
    .flg.lp_measures = 1,           // Low power mode
    .flg2.smiley = 5,               // Happy
    .flg2.adv_crypto = 0,           // No encryption
    .flg2.bt5phy = 0,
    .flg2.longrange = 0,
    .flg2.screen_off = 0,
    .advertising_interval = 40,      // 2.5 seconds
    .measure_interval = 4,           // 10 seconds
    .rf_tx_power = 170,              // ~0 dBm
    .connect_latency = 49,           // 1 second
    .min_step_time_update_lcd = 50,  // 2.5 seconds
    .averaging_measurements = 0,     // Disabled
};
```
