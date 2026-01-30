# LCD Display Documentation

This document describes LCD and E-ink display support in ATC_MiThermometer firmware.

## Overview

The firmware supports multiple LCD and E-ink display types across different device models.

---

## Display Types

### Segment LCD Displays

Used in most devices for low power operation.

| Device | Controller | I2C Address | Buffer Size |
|--------|------------|-------------|-------------|
| LYWSD03MMC | Multiple | 0x3C/0x3E | 6 bytes |
| CGDK2 | BU9792AFUV | 0x3E | 18 bytes |
| MHO-C122 | - | - | 6 bytes |
| ZTH03 | - | - | 6 bytes |
| LKTMZL02 | - | - | 7 bytes |
| ZTH05Z | - | - | 6 bytes |
| ZYZTH01 | - | - | 6 bytes |
| MJWSD06MMC | - | - | 6 bytes |

### E-ink Displays

Used in MHO-C401 and CGG1 devices.

| Device | Buffer Size | Update Time |
|--------|-------------|-------------|
| MHO-C401 | 18 bytes | ~600ms |
| MHO-C401N | 16 bytes | ~600ms |
| CGG1 | 16-18 bytes | ~600ms |

### Large LCD Display

MJWSD05MMC has a larger segmented LCD.

| Device | Buffer Size | Features |
|--------|-------------|----------|
| MJWSD05MMC | 18 bytes | Time display, multiple screens |

---

## LCD Controller Addresses

```c
#define B14_I2C_ADDR        0x3C    // LYWSD03MMC B1.4
#define B19_I2C_ADDR        0x3E    // LYWSD03MMC B1.9
#define BU9792AFUV_I2C_ADDR 0x3E    // BU9792AFUV
#define BL55028_I2C_ADDR    0x3E    // BL55028
#define AIP31620E_I2C_ADDR  0x3E    // AIP31620E
```

---

## Display Functions

### Core Functions

#### `void init_lcd(void)`
Initialize LCD controller and clear display.

#### `void lcd(void)`
Main LCD update function - refreshes display with current data.

#### `void send_to_lcd(void)`
Transfer display buffer to LCD controller via I2C/SPI.

#### `void update_lcd(void)`
Mark LCD for update on next refresh cycle.

### Display Content Functions

#### `void show_big_number_x10(s16 number)`
Display main number with automatic decimal placement.

**Parameters:**
- `number` - Value × 0.1 (range: -995 to 19995)

**Auto-formatting:**
- -99 to -9.9 → negative with decimal
- 0.0 to 199.9 → with decimal point
- 200 to 1999 → no decimal point

#### `void show_small_number(s16 number, bool percent)`
Display secondary number.

**Parameters:**
- `number` - Value (range: -9 to 99)
- `percent` - Show percent symbol

#### `void show_temp_symbol(u8 symbol)`
Display temperature unit symbol.

**Symbol Codes:**
| Code | Display |
|------|---------|
| 0x00 | (blank) |
| 0x20 | °Г |
| 0x40 | - |
| 0x60 | °F |
| 0x80 | _ |
| 0xA0 | °C |
| 0xC0 | = |
| 0xE0 | °E |

```c
#define TMP_SYM_C   0xA0    // Celsius
#define TMP_SYM_F   0x60    // Fahrenheit
```

#### `void show_smiley(u8 state)`
Display comfort indicator smiley face.

**LYWSD03MMC Smileys:**
| Value | Display | Description |
|-------|---------|-------------|
| 0 | (off) | No smiley |
| 1 | ` ^_^ ` | Eyes only |
| 2 | ` -^- ` | Flat eyes |
| 3 | ` ooo ` | Circles |
| 4 | `(   )` | Parentheses only |
| 5 | `(^_^)` | Happy (full) |
| 6 | `(-^-)` | Sad (full) |
| 7 | `(ooo)` | Circles (full) |

**MHO-C401 Smileys:**
| Value | Display |
|-------|---------|
| 0 | (off) |
| 1 | ` o ` |
| 2 | `o^o` |
| 3 | `o-o` |
| 4 | `oVo` |
| 5 | `vVv` happy |
| 6 | `^-^` sad |
| 7 | `oOo` |

```c
#define SMILE_HAPPY 5
#define SMILE_SAD   6
```

#### `void show_battery_symbol(bool state)`
Show or hide battery indicator.

#### `void show_ble_symbol(bool state)`
Show or hide Bluetooth connection indicator.

#### `void show_clock(void)` (if USE_DISPLAY_CLOCK)
Display current time on LCD.

### Special Screens

#### `void show_ota_screen(void)`
Display OTA update progress indicator.

#### `void show_reboot_screen(void)`
Display reboot message before reset.

---

## Display Buffer

Each device has a display buffer that maps to LCD segments:

```c
extern u8 display_buff[LCD_BUF_SIZE];
extern u8 display_cmp_buff[LCD_BUF_SIZE];  // Comparison buffer
```

The comparison buffer is used to detect changes and minimize I2C traffic.

---

## Screen Types (MJWSD05MMC)

```c
enum SCR_TYPE_ENUM {
    SCR_TYPE_TIME = 0,  // Time display (default)
    SCR_TYPE_TEMP,      // Temperature
    SCR_TYPE_HUMI,      // Humidity
    SCR_TYPE_BAT_P,     // Battery percentage
    SCR_TYPE_BAT_V,     // Battery voltage
    SCR_TYPE_EXT        // External data display
};
```

### Screen Rotation

The display can cycle through different screens:
1. Time (if clock enabled)
2. Temperature
3. Humidity
4. Battery (if enabled)

Configure via `cfg.flg2.screen_type` (MJWSD05MMC) or display rotation timing.

---

## External Data Display

Display custom data from external source:

```c
typedef struct {
    s16 big_number;     // Main number × 0.1 (-995..19995)
    s16 small_number;   // Small number × 1 (-9..99)
    u16 vtime_sec;      // Validity time in seconds
    struct {
        u8 smiley       : 3;    // Smiley type
        u8 percent_on   : 1;    // Show % symbol
        u8 battery      : 1;    // Show battery
        u8 temp_symbol  : 3;    // Temperature symbol
    } flg;
} external_data_t;
```

### Setting External Data

Use BLE command CMD_ID_EXTDATA (0x22) to set external display data:

```python
# Example: Display "23.5" with °C symbol
external_data = {
    'big_number': 235,      # 23.5
    'small_number': 50,     # 50
    'vtime_sec': 300,       # 5 minutes validity
    'flg': {
        'smiley': 5,        # Happy
        'percent_on': 1,    # Show %
        'battery': 0,       # No battery
        'temp_symbol': 5    # °C
    }
}
```

---

## LCD Control Flags

```c
typedef struct {
    u32 chow_ext_ut;                // External data validity timer
    u32 min_step_time_update_lcd;   // Minimum update interval
    u32 tim_last_chow;              // Last update timer
    u8  show_stage;                 // Display update stage
    u8  update_next_measure;        // Update on next measurement
    u8  update;                     // Update required flag
    union {
        struct {
            u8 ext_data_buf : 1;    // Show external buffer
            u8 notify_on    : 1;    // Send LCD notifications
            u8 res          : 5;
            u8 send_notify  : 1;    // Send notify flag
        } b;
        u8 all_flg;
    };
} lcd_flg_t;
```

---

## E-ink Display Handling

E-ink displays require special handling due to slow refresh:

### Task-based Updates

```c
int task_lcd(void);
```

E-ink updates are performed in stages to avoid blocking:
1. Stage 0: Prepare data
2. Stage 1: Clear display
3. Stage 2: Write new data
4. Stage 3: Refresh display

### Update Timing

```c
#define USE_EPD  (600/50 - 1)  // ~600ms minimum update time
```

E-ink displays should not be updated too frequently to preserve display life.

---

## Device-Specific Macros

### LYWSD03MMC

```c
#define LCD_BUF_SIZE        6
#define SHOW_SMILEY         1
#define SHOW_OTA_SCREEN()   show_ota_screen()
#define SHOW_REBOOT_SCREEN() show_reboot_screen()
```

### MHO-C401 (E-ink)

```c
#define LCD_BUF_SIZE        18
#define SHOW_SMILEY         1
#define USE_EPD             (600/50 - 1)
#define SHOW_OTA_SCREEN()   // Empty - E-ink too slow
#define SHOW_REBOOT_SCREEN() // Empty
```

### CGDK2

```c
#define LCD_BUF_SIZE        18
#define SHOW_SMILEY         0   // No smiley support
void show_batt_cgdk2(void);     // Special battery display
```

### MJWSD05MMC

```c
#define LCD_BUF_SIZE        18
#define POWERUP_SCREEN      1   // Show startup screen
void show_symbol_s1(u8 symbol); // Additional symbol control
void show_low_bat(void);        // Low battery warning
```

---

## Power Management

### Screen Off Mode

Enable screen off to save power:
```c
cfg.flg2.screen_off = 1;
```

### Update Rate Control

Configure LCD update interval:
```c
cfg.min_step_time_update_lcd = 50;  // × 50ms = 2.5 seconds
```

### E-ink Power

E-ink displays retain image without power, making them ideal for battery-powered devices.

---

## LCD Dump Command

Read current LCD buffer via BLE:

**Command:** CMD_ID_LCD_DUMP (0x60)

**Response:** Current display buffer bytes

**Set Display:** Write bytes after command ID to set display content directly.

---

## Troubleshooting

### Display Not Working

1. Check I2C address detection
2. Verify I2C pull-up resistors
3. Check power supply voltage
4. Confirm correct device type in firmware

### Incorrect Segments

1. Verify display buffer mapping
2. Check hardware version detection
3. Update to latest firmware

### E-ink Ghosting

1. Increase update interval
2. Ensure full refresh cycles
3. Check temperature (E-ink performance varies with temp)
