# Инструкции по сборке кастомной прошивки для Lora-Shuttle

## Что было добавлено

В модуль `ExternalNotificationModule` добавлена кастомная функциональность для платы PROKYBER LORA-Shuttle (HT-CT62):

### Функциональность:

1. **LED уведомления (GPIO0)**
   - Мигает при получении сообщения, если BLE выключен и WiFi выключен
   - Мигает каждые 500мс

2. **Звуковые уведомления (GPIO1)**
   - Пищит при получении сообщения, если BLE выключен и WiFi выключен
   - Повторяет писк каждую минуту

3. **Кнопка управления (GPIO2)**
   - **Одинарный клик**: останавливает уведомления (LED + звук)
   - **Двойной клик**: выполняет пользовательское действие 1 (настраивается)
   - **Тройной клик**: выполняет пользовательское действие 2 (настраивается)

4. **Интеграция с Telegram (асинхронная)**
   - Если WiFi подключен, отправляет копию полученного сообщения в Telegram бот
   - Включает все метаданные: отправитель, RSSI, SNR, hop limit
   - **Неблокирующая отправка**: использует FreeRTOS task, не замораживает основной цикл

### Условия остановки уведомлений:

- Подключение телефона по Bluetooth
- Нажатие кнопки (одинарный клик)

## Настройка Telegram

Перед сборкой необходимо настроить Telegram:

1. Создайте бота через [@BotFather](https://t.me/BotFather)
2. Получите токен бота
3. Узнайте ваш Chat ID (можно через [@userinfobot](https://t.me/userinfobot))
4. Откройте файл `src/modules/ExternalNotificationModule.cpp`
5. Найдите строки (примерно 77-78):
   ```cpp
   String botToken = "YOUR_TELEGRAM_BOT_TOKEN";
   String chatID = "YOUR_TELEGRAM_CHAT_ID";
   ```
6. Замените на ваши реальные значения:
   ```cpp
   String botToken = "123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11";
   String chatID = "123456789";
   ```

## Установка VS Code и PlatformIO

1. Установите [Visual Studio Code](https://code.visualstudio.com/)
2. Установите расширение PlatformIO IDE
3. Откройте папку `meshtastic-firmware` в VS Code

## Сборка прошивки

### Для Heltec ESP32-C3 (Lora-Shuttle):

```bash
# В терминале VS Code
pio run -e heltec-ht62-esp32c3-sx1262
```

### Прошивка устройства:

1. Подключите Lora-Shuttle к компьютеру через USB-C
2. Переведите в режим загрузчика:
   - Зажмите кнопку BOOT (правая)
   - Нажмите кнопку RST (левая)
   - Отпустите обе кнопки
3. Прошейте:
```bash
pio run -e heltec-ht62-esp32c3-sx1262 -t upload
```

Или используйте Task в VS Code:
- Нажмите `Ctrl+Shift+P`
- Выберите `PlatformIO: Upload`
- Выберите окружение `heltec-ht62-esp32c3-sx1262`

## Проверка работы

1. После прошивки устройство перезагрузится
2. Подключитесь через Meshtastic приложение
3. Настройте WiFi (если нужна интеграция с Telegram)
4. Попросите кого-то отправить вам сообщение в сети
5. Если BLE отключен и WiFi выключен - должен замигать LED и пищать динамик
6. Если WiFi включен - сообщение должно прийти в Telegram

## Отключение Bluetooth для теста

Чтобы проверить работу уведомлений:

1. Подключитесь к устройству через Meshtastic app
2. Настройте модуль External Notification:
   ```
   meshtastic --set external_notification.enabled true
   ```
3. Отключитесь от устройства в приложении
4. Попросите кого-то отправить сообщение

## Распиновка

| Функция | GPIO | Примечание |
|---------|------|------------|
| LED | 0 | Свободный пин на плате |
| Buzzer | 1 | I2C SDA (если I2C не используется) |
| Button | 2 | I2C SCL (если I2C не используется) |

**ВНИМАНИЕ**: Если вы подключаете I2C устройства (дисплей, датчики) к белому разъёму, 
пины GPIO1 и GPIO2 будут заняты! Измените пины в коде на другие свободные.

## Кастомизация

### Изменение пинов:

Откройте `src/modules/ExternalNotificationModule.cpp` и измените (около строки 72):

```cpp
#define CUSTOM_LED_PIN 0          // Ваш пин
#define CUSTOM_BUZZER_PIN 1       // Ваш пин
#define CUSTOM_BUTTON_PIN 2       // Ваш пин
```

### Добавление действий для двойного/тройного клика:

Найдите в функции `runOnce()` (около строки 220):

```cpp
if (click == DOUBLE_CLICK) {
    LOG_INFO("Double click detected - performing action 1");
    // Добавьте ваш код здесь
    // Например: отправить broadcast сообщение
    // Или: переключить режим работы
}
if (click == TRIPLE_CLICK) {
    LOG_INFO("Triple click detected - performing action 2");
    // Добавьте ваш код здесь
}
```

### Изменение интервалов:

В начале файла (около строки 74):

```cpp
#define BLINK_INTERVAL 500        // Частота мигания LED (мс)
#define BEEP_INTERVAL 60000       // Частота писка (мс) - 60000 = 1 минута
#define CLICK_TIMEOUT 500         // Таймаут между кликами (мс)
```

## Ветка Git

Все изменения сделаны в ветке: `custom-lora-shuttle-v2.6.11`

Базовая версия: `v2.6.11.60ec05e`

## Проблемы и решения

### ArduinoJson не найден
ArduinoJson включен в зависимости ESP32 по умолчанию. Если возникает ошибка, добавьте в `platformio.ini`:
```ini
lib_deps = 
  bblanchon/ArduinoJson@^6.21.4
```

### WiFi не подключается
Убедитесь, что WiFi настроен в конфигурации Meshtastic:
```bash
meshtastic --set network.wifi_enabled true
meshtastic --set network.wifi_ssid "YOUR_SSID"
meshtastic --set network.wifi_psk "YOUR_PASSWORD"
```

### Кнопка не реагирует
Проверьте, что GPIO2 не используется для других целей. Кнопка должна соединять GPIO2 с GND при нажатии.

## Контакты

Для вопросов и предложений создайте Issue в репозитории.
