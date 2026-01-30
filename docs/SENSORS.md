# Sensor Support

This document describes all sensors supported by ATC_MiThermometer firmware.

## Overview

The firmware supports multiple sensor types for temperature, humidity, pressure, and other measurements.

---

## Temperature/Humidity Sensors

### Sensirion SHTC3

| Property | Value |
|----------|-------|
| Type ID | `TH_SENSOR_SHTC3` (1) |
| I2C Address | 0x70 |
| Temp Range | -40°C to +125°C |
| Temp Accuracy | ±0.2°C |
| Humidity Range | 0-100% RH |
| Humidity Accuracy | ±2% RH |

**Used in:** LYWSD03MMC B1.4/B1.5, MHO-C401, CGG1, CGDK2

**Features:**
- Ultra-low power consumption
- Fast response time
- Built-in heater for condensation removal

### Sensirion SHT4x (SHT40/SHT41/SHT45)

| Property | Value |
|----------|-------|
| Type ID | `TH_SENSOR_SHT4x` (2) |
| I2C Address | 0x44, 0x45, 0x46 |
| Temp Range | -40°C to +125°C |
| Temp Accuracy | ±0.2°C (typical) |
| Humidity Range | 0-100% RH |
| Humidity Accuracy | ±1.8% RH |

**Used in:** LYWSD03MMC B1.6/B1.7/B1.9, MJWSD05MMC

**Features:**
- Fourth generation Sensirion sensor
- Excellent long-term stability
- Multiple precision modes

### Sensirion SHT30

| Property | Value |
|----------|-------|
| Type ID | `TH_SENSOR_SHT30` (3) |
| I2C Address | 0x44, 0x45 |
| Temp Range | -40°C to +125°C |
| Temp Accuracy | ±0.2°C |
| Humidity Range | 0-100% RH |
| Humidity Accuracy | ±2% RH |

**Used in:** Various Tuya/Zigbee devices

### CHT8305

| Property | Value |
|----------|-------|
| Type ID | `TH_SENSOR_CHT8305` (4) |
| I2C Address | 0x40-0x43 |
| Temp Range | -40°C to +125°C |
| Humidity Range | 0-100% RH |

**Used in:** Some Tuya Zigbee devices

### AHT20/AHT30

| Property | Value |
|----------|-------|
| Type ID | `TH_SENSOR_AHT2x` (5) |
| I2C Address | 0x38 |
| Temp Range | -40°C to +85°C |
| Temp Accuracy | ±0.3°C |
| Humidity Range | 0-100% RH |
| Humidity Accuracy | ±2% RH |

**Used in:** Tuya ZTH05Z, ZG-227Z

### CHT8215

| Property | Value |
|----------|-------|
| Type ID | `TH_SENSOR_CHT8215` (6) |
| I2C Address | Varies |

---

## Pressure Sensor

### Bosch BME280

| Property | Value |
|----------|-------|
| Type ID | `IU_SENSOR_BME280` (15) |
| I2C Address | 0x76 |
| Chip ID | 0x60 |
| Pressure Range | 300-1100 hPa |
| Pressure Accuracy | ±1 hPa |
| Also measures | Temperature, Humidity |

**Features:**
- Barometric pressure sensing
- Can calculate altitude
- Low power consumption

**Register Map:**

| Register | Address | Description |
|----------|---------|-------------|
| CHIP_ID | 0xD0 | Returns 0x60 |
| SOFT_RESET | 0xE0 | Write 0xB6 to reset |
| CTRL_HUM | 0xF2 | Humidity control |
| STATUS | 0xF3 | Measurement status |
| CTRL_MEAS | 0xF4 | Temp/pressure control |
| CONFIG | 0xF5 | Rate, filter, interface |
| PRESS | 0xF7-F9 | Pressure data (20-bit) |
| TEMP | 0xFA-FC | Temperature data (20-bit) |
| HUM | 0xFD-FE | Humidity data (16-bit) |

---

## External Temperature Sensor

### Dallas DS18B20 (MY18B20)

| Property | Value |
|----------|-------|
| Type ID | `IU_SENSOR_MY18B20` (8) |
| Interface | 1-Wire |
| Temp Range | -55°C to +125°C |
| Resolution | 9-12 bit configurable |
| Accuracy | ±0.5°C (typical) |

**Features:**
- Parasitic power mode support
- Unique 64-bit serial code
- Multiple sensors on one bus

**Configuration Structure:**

```c
typedef struct {
    u32 val1_k;     // Temperature coefficient
    s16 val1_z;     // Temperature offset
    u32 id;         // ROM ID
    u8  res;        // Resolution
    u8  type;       // Sensor type
    u8  rd_ok;      // Read success flag
    u8  stage;      // State machine stage
    s16 temp[N];    // Temperature readings
} my18b20_t;
```

**Dual Sensor Support:**
- Type ID: `IU_SENSOR_MY18B20x2` (9)
- Two independent temperature readings

---

## Current/Voltage Sensors

### Texas Instruments INA226

| Property | Value |
|----------|-------|
| Type ID | `IU_SENSOR_INA226` (7) |
| I2C Address | 0x40-0x4F |
| Bus Voltage | 0-36V |
| Shunt Voltage | ±81.92mV |
| Resolution | 16-bit |

**Features:**
- Bi-directional current sensing
- Power monitoring
- Energy accumulation

**Measurements:**
- Voltage (0.001V resolution)
- Current (0.001A resolution)
- Power calculation
- Energy integration

### Texas Instruments INA3221

| Property | Value |
|----------|-------|
| Type ID | `IU_SENSOR_INA3221` (13) |
| Channels | 3 independent |
| Bus Voltage | 0-26V |

**Features:**
- Triple-channel current/voltage monitor
- Independent measurements per channel
- Warning/critical alerts

---

## Scale Sensor

### HX711/HX710 ADC

| Property | Value |
|----------|-------|
| Type ID | `IU_SENSOR_HX71X` (10) |
| Resolution | 24-bit |
| Data Rate | 10/80 SPS |
| Gain | 32, 64, 128 |

**Configuration:**

```c
typedef struct {
    u32 zero;           // Zero offset
    u32 coef;           // Scale coefficient
    u32 volume_10ml;    // Full tank volume
} hx71x_cfg_t;
```

**Gain/Channel Modes:**

| Mode | Value | Description |
|------|-------|-------------|
| HX71XMODE_A128 | 25 | Channel A, gain 128 |
| HX71XMODE_B32 | 26 | Channel B, gain 32 |
| HX71XMODE_A64 | 27 | Channel A, gain 64 |

**Applications:**
- Weight measurement
- Liquid level sensing
- Force measurement

---

## CO2 Sensor

### Sensirion SCD41

| Property | Value |
|----------|-------|
| Type ID | `IU_SENSOR_SCD41` (14) |
| CO2 Range | 400-5000 ppm |
| CO2 Accuracy | ±(40ppm + 5%) |
| Also measures | Temperature, Humidity |

**Features:**
- Photoacoustic sensing principle
- On-chip signal processing
- Automatic self-calibration

---

## Air Quality Sensor

### ScioSense ENS160

| Property | Value |
|----------|-------|
| Type ID | - |
| TVOC Range | 0-65000 ppb |
| eCO2 Range | 400-65000 ppm |
| AQI | 1-5 scale |

**Features:**
- Metal oxide gas sensing
- Multiple gas detection
- Built-in algorithms for AQI

---

## Sensor Calibration

### Calibration Coefficients

```c
typedef struct {
    u32 val1_k;     // Temperature multiplier (scale factor)
    u32 val2_k;     // Humidity multiplier
    s16 val1_z;     // Temperature offset (zero point)
    s16 val2_z;     // Humidity offset
} sensor_coef_t;
```

### Calibration Formula

```
calibrated_value = (raw_value * k) / 65536 + z
```

Where:
- `k` = multiplier coefficient (default: 65536 = 1.0)
- `z` = offset (default: 0)

### Default Coefficients

```c
sensor_coef_t default_coef = {
    .val1_k = 65536,    // 1.0 multiplier
    .val2_k = 65536,    // 1.0 multiplier
    .val1_z = 0,        // No offset
    .val2_z = 0         // No offset
};
```

### Applying Calibration

1. Measure reference values with calibrated equipment
2. Calculate offset: `z = reference - measured`
3. Send calibration via CMD_ID_CFS (0x25)

---

## Sensor Detection

The firmware auto-detects sensors during initialization:

```c
void init_sensor(void) {
    // Scan I2C bus for known addresses
    if (scan_i2c_addr(SHTC3_I2C_ADDR) == 0)
        sensor_cfg.sensor_type = TH_SENSOR_SHTC3;
    else if (scan_i2c_addr(SHT4x_I2C_ADDR) == 0)
        sensor_cfg.sensor_type = TH_SENSOR_SHT4x;
    // ... etc
}
```

### Sensor Type Enumeration

```c
enum TH_SENSOR_TYPES {
    TH_SENSOR_NONE = 0,
    TH_SENSOR_SHTC3,        // 1
    TH_SENSOR_SHT4x,        // 2
    TH_SENSOR_SHT30,        // 3
    TH_SENSOR_CHT8305,      // 4
    TH_SENSOR_AHT2x,        // 5
    TH_SENSOR_CHT8215,      // 6
    IU_SENSOR_INA226,       // 7
    IU_SENSOR_MY18B20,      // 8
    IU_SENSOR_MY18B20x2,    // 9
    IU_SENSOR_HX71X,        // 10
    IU_SENSOR_PWMRH,        // 11
    IU_SENSOR_NTC,          // 12
    IU_SENSOR_INA3221,      // 13
    IU_SENSOR_SCD41,        // 14
    IU_SENSOR_BME280,       // 15
};
```

---

## Measurement Data Structure

```c
typedef struct {
    u16 battery_mv;         // Battery voltage mV
    s16 temp;               // Temperature × 0.01°C
    s16 humi;               // Humidity × 0.01%
    u16 count;              // Measurement counter
    u32 pressure;           // Pressure (if BME280)
    u16 co2;                // CO2 ppm (if SCD41)
    s16 xtemp[N];           // External temps (if MY18B20)
    s16 temp_x01;           // Temperature × 0.1°C
    s16 humi_x01;           // Humidity × 0.1%
    u8  humi_x1;            // Humidity × 1%
    u8  battery_level;      // Battery 0-100%
    s32 energy;             // Energy (if INA226)
} measured_data_t;
```

---

## Power Modes

### Normal Mode
- Regular measurement intervals
- Standard accuracy
- Moderate power consumption

### Low Power Mode
- Extended sleep between measurements
- Slightly reduced accuracy
- Minimum power consumption

Enable low power mode:
```c
cfg.flg.lp_measures = 1;
```

---

## Adding New Sensor Support

1. Define sensor type in `TH_SENSOR_TYPES` enum
2. Add I2C address constant
3. Implement detection in `init_sensor()`
4. Implement `read_sensor_cb()` for the sensor
5. Update `start_measure_sensor_deep_sleep()` if needed
6. Add calibration structure if required
