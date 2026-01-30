# ATC_MiThermometer Documentation

Welcome to the comprehensive documentation for the ATC_MiThermometer custom firmware project.

## Table of Contents

| Document | Description |
|----------|-------------|
| [API Reference](API_REFERENCE.md) | Complete API documentation for all functions and interfaces |
| [BLE Advertising Formats](BLE_ADVERTISING.md) | Detailed BLE advertising format specifications |
| [Configuration Guide](CONFIGURATION.md) | Device configuration options and settings |
| [Sensors](SENSORS.md) | Supported temperature, humidity, and other sensors |
| [LCD Display](LCD_DISPLAY.md) | LCD/E-ink display drivers and control |
| [Hardware Compatibility](HARDWARE.md) | Supported devices and hardware versions |

## Project Overview

ATC_MiThermometer is a custom firmware for Xiaomi Mijia BLE thermometers and compatible devices based on the Telink TLSR8251 chipset. The firmware provides enhanced functionality over the stock firmware including:

- **Multiple BLE Advertising Formats**: Support for ATC1441, PVVX Custom, Xiaomi Mi, and BTHome v2 formats
- **Encryption Support**: Optional AES-CCM encryption for secure data transmission
- **Home Automation Integration**: Native BTHome v2 support for Home Assistant
- **Extended Battery Life**: Low-power optimizations
- **Data Logging**: Flash-based measurement history
- **Sensor Calibration**: Temperature and humidity offset calibration
- **Reed Switch Support**: Door/window sensor functionality
- **OTA Updates**: Over-the-air firmware updates

## Quick Links

### For Users
- [Configuration Guide](CONFIGURATION.md) - How to configure your device
- [Hardware Compatibility](HARDWARE.md) - Check if your device is supported

### For Developers
- [API Reference](API_REFERENCE.md) - Complete function documentation
- [BLE Advertising Formats](BLE_ADVERTISING.md) - Protocol specifications
- [Sensors](SENSORS.md) - Adding sensor support

## Firmware Version

Current firmware version: **5.6** (BCD format: 0x56)

## License

This project is open source. See the main repository for license details.

## Contributing

Contributions are welcome! Please refer to the main repository for contribution guidelines.
