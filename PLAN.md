# Detailed Implementation Plan for FireBeetle ESP32 + Waveshare 7.5" e-Paper HAT Support

## Current Architecture Analysis

The TRMNL firmware uses a well-structured hardware abstraction system:

- **Display Support**: Uses `bb_epaper` library with support for `EP75_800x480` (GDEY075T7) displays
- **Board Configuration**: Hardware-specific pin mappings in `DEV_Config.h` with preprocessor directives
- **PlatformIO Environments**: Separate build configurations for different board types
- **Pin Management**: Board-specific GPIO definitions in `config.h`

## Implementation Plan

### Phase 1: Board Configuration Setup ✅ COMPLETED

#### 1.1 Add PlatformIO Environment ✅

**File:** `platformio.ini`
**Action:** Added new environment `dfrobot_firebeetle2_esp32e` for FireBeetle ESP32

#### 1.2 Define Hardware Configuration ✅

**File:** `DEV_Config.h`
**Action:** Added FireBeetle ESP32 pin definitions

Based on provided pin configuration:

```cpp
#elif defined(BOARD_FIREBEETLE_ESP32)
   // Pin definition for FireBeetle ESP32 + Waveshare 7.5" e-Paper HAT
   #define EPD_SCK_PIN  18
   #define EPD_MOSI_PIN 23
   #define EPD_CS_PIN   2
   #define EPD_RST_PIN  21
   #define EPD_DC_PIN   22
   #define EPD_BUSY_PIN 13
```

#### 1.3 Device Configuration ✅

**File:** `config.h`
**Action:** Added FireBeetle-specific definitions

```cpp
#elif defined(BOARD_FIREBEETLE_ESP32)
#define DEVICE_MODEL "dfrobot_firebeetle2_esp32e"
#define PIN_INTERRUPT 27  // Built-in button on FireBeetle ESP32
#define FAKE_BATTERY_VOLTAGE
```

### Phase 2: Display Configuration

#### 2.1 Verify Display Compatibility ✅ COMPLETED

- **Current**: Firmware uses `EP75_800x480` (GDEY075T7) for TRMNL boards
- **Target**: Waveshare 7.5" HAT uses compatible controller but optimized for Waveshare panels  
- **Action**: Updated display.cpp to use board-specific display definitions
  - FireBeetle ESP32: Uses `EP75R_800x480` (Waveshare-optimized) for better compatibility
  - Other boards: Continue using `EP75_800x480` (GDEY075T7) as before
- **Implementation**: Added conditional compilation in display.cpp based on `BOARD_FIREBEETLE_ESP32` flag
- **Result**: FireBeetle ESP32 now uses Waveshare-specific display driver for optimal compatibility with 7.5" HAT

#### 2.2 Test Display Initialization

**File:** `display.cpp`
**Action:** Verify `bb_epaper` library compatibility with Waveshare 7.5" HAT

> **Note**: May need to test `EP75_800x480`, `EP75R_800x480`, or create new display profile

### Phase 3: Pin and Button Configuration

#### 3.1 Button Handling

- **Current**: Uses `PIN_INTERRUPT` for wake-up and configuration
- **FireBeetle**: GPIO27 has built-in button (D4)
- **Action**: Configure GPIO27 as interrupt pin for button functionality

#### 3.2 Power Management

- **Current**: Battery voltage monitoring on GPIO3
- **FireBeetle**: No built-in battery management
- **Action**: Use `FAKE_BATTERY_VOLTAGE` to report fixed 4.2V

### Phase 4: Hardware Integration Testing

#### 4.1 SPI Communication Verification

- **Test**: Verify SPI pins (18=SCK, 23=MOSI) work with e-Paper HAT
- **Validate**: No conflicts with FireBeetle's built-in peripherals

#### 4.2 Pin Conflict Analysis

- **GPIO5**: FireBeetle RGB LED - no conflict with e-Paper
- **GPIO2**: FireBeetle onboard LED - no conflict
- **GPIO27**: User button - conflicts with DC pin, may need reassignment

**Potential Pin Reassignment** (if GPIO27 conflict):
- DC: GPIO14 (recommended)
- Button: Keep GPIO27 (built-in hardware)

### Phase 5: Testing and Validation

#### 5.1 Basic Functionality Tests

- **SPI Communication**: Verify e-Paper display initialization
- **Image Display**: Test BMP and PNG rendering
- **Refresh Modes**: Validate partial/full refresh strategies
- **Button Interaction**: Test wake-up and configuration modes
- **WiFi Connectivity**: Verify no interference with display SPI

#### 5.2 Power Management Testing

- **Deep Sleep**: Verify proper sleep/wake cycles
- **Display Power**: Test display sleep functionality
- **Current Consumption**: Measure power usage (should be similar to other boards)

### Phase 6: Documentation and Integration

#### 6.1 Update Documentation

- Add FireBeetle ESP32 to supported hardware list
- Document pin connections for Waveshare 7.5" HAT
- Update build instructions

#### 6.2 Create Connection Guide

**FireBeetle ESP32 to Waveshare 7.5" e-Paper HAT connections:**

| e-Paper HAT | FireBeetle ESP32 | Function |
|-------------|------------------|----------|
| VCC         | 3V3             | Power    |
| GND         | GND             | Ground   |
| DIN         | GPIO23 (MOSI)   | SPI Data |
| CLK         | GPIO18 (SCK)    | SPI Clock|
| CS          | GPIO15          | Chip Select |
| DC          | GPIO14          | Data/Command |
| RST         | GPIO26          | Reset    |
| BUSY        | GPIO25          | Busy     |

## Implementation Files to Modify

- `platformio.ini` - Add `dfrobot_firebeetle2_esp32e` environment
- `DEV_Config.h` - Add pin definitions for `BOARD_FIREBEETLE_ESP32`
- `config.h` - Add device model and interrupt pin
- `README.md` - Update supported hardware documentation

## Potential Challenges and Solutions

### Challenge 1: Pin Conflicts
- **Issue**: GPIO27 used for both button and potentially DC pin
- **Solution**: Reassign DC pin to GPIO14, keep GPIO27 for button

### Challenge 2: Display Compatibility
- **Issue**: Waveshare 7.5" HAT may use different controller than GDEY075T7
- **Solution**: Test with existing `EP75_800x480` definition, create new profile if needed

### Challenge 3: Power Management
- **Issue**: FireBeetle lacks battery monitoring
- **Solution**: Use `FAKE_BATTERY_VOLTAGE` flag (already implemented pattern)

### Challenge 4: SPI Interface
- **Issue**: Ensure no conflicts with other FireBeetle peripherals
- **Solution**: Verify SPI pins are available and not used by RGB LED or other functions

## Testing Strategy

1. **Start Simple**: Begin with basic display initialization and static image
2. **Incremental Testing**: Add features one by one (WiFi, buttons, sleep)
3. **Comparative Testing**: Compare behavior with existing supported boards
4. **Edge Case Testing**: Test various image formats, network conditions, sleep cycles

This plan provides a comprehensive approach to adding FireBeetle ESP32 support while maintaining the existing architecture and ensuring compatibility with the current firmware design patterns.