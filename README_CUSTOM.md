# Custom Lora-Shuttle Firmware Build Instructions

## Features Added

Custom functionality added to `ExternalNotificationModule` for PROKYBER LORA-Shuttle board (HT-CT62):

### Functionality

1. **LED Notifications (GPIO0)**
   - Blinks when message received if BLE is OFF and WiFi is OFF
   - Blinks every 500ms

2. **Buzzer Notifications (GPIO1)**
   - Beeps when message received if BLE is OFF and WiFi is OFF
   - Repeats beep every minute

3. **Button Control (GPIO2)**
   - **Single click**: stops notifications (LED + buzzer)
   - **Double click**: performs custom action 1 (configurable)
   - **Triple click**: performs custom action 2 (configurable)

4. **Telegram Integration (Async)**
   - If WiFi connected, sends copy of received message to Telegram bot
   - Includes all metadata: sender, RSSI, SNR, hop limit
   - **Non-blocking send**: uses FreeRTOS task, doesn't freeze main loop

### Alert Stop Conditions

- Phone connects via Bluetooth
- Button single-click

## Telegram Configuration

Before building, configure Telegram:

1. Create bot via [@BotFather](https://t.me/BotFather)
2. Get bot token
3. Get your Chat ID (via [@userinfobot](https://t.me/userinfobot))
4. Open file `src/modules/ExternalNotificationModule.cpp`
5. Find lines (around 77-78):
   ```cpp
   String botToken = "YOUR_TELEGRAM_BOT_TOKEN";
   String chatID = "YOUR_TELEGRAM_CHAT_ID";
   ```
6. Replace with your real values:
   ```cpp
   String botToken = "123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11";
   String chatID = "123456789";
   ```

## VS Code and PlatformIO Setup

1. Install [Visual Studio Code](https://code.visualstudio.com/)
2. Install PlatformIO IDE extension
3. Open `meshtastic-firmware` folder in VS Code

## Building Firmware

### For Heltec ESP32-C3 (Lora-Shuttle)

```bash
# In VS Code terminal
pio run -e heltec-ht62-esp32c3-sx1262
```

### Flashing Device

1. Connect Lora-Shuttle to computer via USB-C
2. Enter bootloader mode:
   - Hold BOOT button (right)
   - Press RST button (left)
   - Release both buttons
3. Flash:
```bash
pio run -e heltec-ht62-esp32c3-sx1262 -t upload
```

Or use VS Code Task:
- Press `Ctrl+Shift+P`
- Select `PlatformIO: Upload`
- Choose environment `heltec-ht62-esp32c3-sx1262`

## Testing

1. After flashing, device will reboot
2. Connect via Meshtastic app
3. Configure WiFi (if Telegram integration needed)
4. Ask someone to send you a message
5. If BLE disconnected and WiFi OFF - LED should blink and buzzer beep
6. If WiFi ON - message should arrive in Telegram

## Testing Bluetooth Disconnect

To test notifications:

1. Connect to device via Meshtastic app
2. Configure External Notification module:
   ```bash
   meshtastic --set external_notification.enabled true
   ```
3. Disconnect from device in app
4. Ask someone to send a message

## Pin Configuration

| Function | GPIO | Notes |
|----------|------|-------|
| LED | 0 | Free pin on board |
| Buzzer | 1 | I2C SDA (if I2C not used) |
| Button | 2 | I2C SCL (if I2C not used) |

**WARNING**: If you connect I2C devices (display, sensors) to white connector,
GPIO1 and GPIO2 will be occupied! Change pins in code to other free GPIOs.

## Customization

### Changing Pins

Open `src/modules/ExternalNotificationModule.cpp` and modify (around line 72):

```cpp
#define CUSTOM_LED_PIN 0          // Your pin
#define CUSTOM_BUZZER_PIN 1       // Your pin
#define CUSTOM_BUTTON_PIN 2       // Your pin
```

### Adding Double/Triple Click Actions

Find in `runOnce()` function (around line 220):

```cpp
if (click == DOUBLE_CLICK) {
    LOG_INFO("Double click detected - performing action 1");
    // Add your code here
    // Example: send broadcast message
    // Or: toggle operation mode
}
if (click == TRIPLE_CLICK) {
    LOG_INFO("Triple click detected - performing action 2");
    // Add your code here
}
```

### Changing Intervals

At file beginning (around line 74):

```cpp
#define BLINK_INTERVAL 500        // LED blink frequency (ms)
#define BEEP_INTERVAL 60000       // Beep frequency (ms) - 60000 = 1 minute
#define CLICK_TIMEOUT 500         // Timeout between clicks (ms)
```

## Git Branch

All changes made in branch: `custom-lora-shuttle-v2.6.11`

Base version: `v2.6.11.60ec05e`

## Troubleshooting

### ArduinoJson not found

ArduinoJson is included in ESP32 dependencies by default. If error occurs, add to `platformio.ini`:

```ini
lib_deps = 
  bblanchon/ArduinoJson@^6.21.4
```

### WiFi not connecting

Make sure WiFi is configured in Meshtastic config:

```bash
meshtastic --set network.wifi_enabled true
meshtastic --set network.wifi_ssid "YOUR_SSID"
meshtastic --set network.wifi_psk "YOUR_PASSWORD"
```

### Button not responding

Check that GPIO2 is not used for other purposes. Button should connect GPIO2 to GND when pressed.

## Repository

GitHub: https://github.com/lexejs/meshtastic_firmware/tree/custom-lora-shuttle-v2.6.11

## Contact

For questions and suggestions, create an Issue in the repository.
