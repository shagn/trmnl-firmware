# TRMNL Firmware Development Guide

## Architecture Overview

This is ESP32-based firmware for TRMNL e-ink displays that fetches images from a web API and displays them. The system follows a sleep-wake cycle architecture with deep sleep power management.

### Core Components

- **Business Logic (`src/bl.cpp`)**: Main state machine handling device lifecycle - WiFi setup, API communication, image display, OTA updates, and sleep management
- **Display Module (`src/display.cpp`)**: E-ink display abstraction with multi-format image support and optimized refresh strategies
- **API Client (`src/api-client/`)**: HTTP client for `/api/setup` (device registration) and `/api/display` (image fetching) endpoints
- **Custom Libraries (`lib/trmnl/`)**: Core utilities for image processing (BMP/PNG), API parsing, logging, and data persistence

### Display Abstraction System

The display module provides hardware abstraction for 800x480 e-ink panels across multiple board configurations:

**Hardware Abstraction**: Uses `bb_epaper` library with board-specific pin mappings in `DEV_Config.h`. SPI interface pins (SCK, MOSI, CS, RST, DC, BUSY) are defined per board type (TRMNL, Waveshare, Seeed variants).

**Image Format Support**:
- **BMP Priority**: 1-bit BMP format (48KB) is preferred for optimal performance and partial updates
- **PNG Fallback**: PNG decoder with smart bit-depth detection (1-bit or 2-bit grayscale)
- **G5 Compression**: Custom compressed bitmap format for built-in graphics

**Refresh Strategy Intelligence**:
- **Partial Updates**: For 1-bit images or 2-bit images with only 2 unique colors to minimize flicker
- **Full Updates**: Required for true 4-grayscale content or first-time display
- **Update Counter**: Tracks partial updates in RTC memory to force periodic full refreshes

**Memory Management**: Direct line-by-line decoding to framebuffer due to RAM constraints - images are never fully loaded into memory.

**Text Rendering**: Multi-line text with word wrapping, center alignment, and font scaling for system messages and error states.

### Key Data Flow

1. Wake from deep sleep → Button handling → WiFi connection
2. Device registration via MAC address exchange for API key/friendly ID
3. Fetch display content with device status headers (battery, RSSI, firmware version)
4. Process images (BMP preferred, PNG supported) and display
5. Handle OTA updates and credential resets based on API response
6. Return to deep sleep for configured refresh interval

## Development Patterns

### Board Configuration
Use `platformio.ini` environments for different hardware targets:
- `env:trmnl` - Production TRMNL hardware (ESP32-C3)
- `env:local` - Development with USB serial output
- `env:native` - Unit testing on host machine

### Testing Strategy
- Unit tests in `test/` use Unity framework with ArduinoFake for mocking
- Test specific modules: `test_parse_api_display`, `test_bmp`, `test_stored_logs`
- Run native tests with `pio test -e native`

### Error Handling & Logging
- Custom logging via `trmnl_log.h` with different severity levels
- Persistent error logging to preferences with automatic server submission
- Network failures trigger progressive retry delays (15s → 30s → 60s)

### Power Management
- Deep sleep between refresh cycles (default 15 minutes)
- GPIO wake-up for button interactions (long press = WiFi reset, double-click = special functions)
- Battery voltage monitoring with critical level detection

### Memory Management
- Static buffers for image data (`buffer`, `buffer_old` globals)
- RTC memory for state persistence across deep sleep cycles (`RTC_DATA_ATTR`)
- SPIFFS filesystem for temporary image storage and file operations
- Direct framebuffer streaming for large images to avoid RAM overflow

### Display Development Patterns
- **Board-Specific Configuration**: Pin mappings defined in `DEV_Config.h` using preprocessor directives
- **Image Processing Pipeline**: BMP preferred → PNG fallback → format validation → refresh strategy selection
- **Partial Update Logic**: Compare new vs old image, detect color count, choose optimal refresh mode
- **Error Display**: System messages rendered with multi-line text wrapping and logos from embedded graphics

## Critical Workflows

### Building & Flashing
```bash
# Build for production TRMNL device
pio run -e trmnl

# Build with debug output  
pio run -e local

# Flash device (requires boot button press)
pio run -e trmnl -t upload
```

### API Integration
- Server expects specific headers: `ID` (MAC), `Access-Token`, `Battery-Voltage`, `FW-Version`, `RSSI`
- Image URLs must return BMP format (800x480, 1-bit depth) for optimal performance
- Special functions triggered via double-click: identify, sleep, add WiFi

### Display Integration
- **Format Priority**: Always prefer BMP over PNG for performance - server should deliver 1-bit BMP (48KB)
- **Refresh Modes**: Use `REFRESH_PARTIAL` for 1-bit or 2-color images, `REFRESH_FULL` for 4-grayscale
- **Update Counter**: `iUpdateCount` in RTC memory tracks partial updates to force periodic full refresh
- **Pin Configuration**: Each board variant defines unique SPI pins in `DEV_Config.h` - use appropriate `BOARD_*` flags

### Configuration Management
- Device credentials stored in preferences (API key, friendly ID, WiFi)
- Firmware version defined in `include/config.h` 
- Board-specific pins and features via preprocessor directives

## Integration Points

- **usetrmnl.com API**: RESTful endpoints for device management and content delivery
- **bb_epaper Library**: Hardware abstraction for various e-ink display controllers
- **ESPAsyncWebServer**: Captive portal for WiFi configuration
- **ArduinoJson**: API response parsing and log serialization
- **PNGdec**: PNG image decoding fallback for non-BMP content
