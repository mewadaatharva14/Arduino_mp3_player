# DIY Portable MP3 Player 🎵

## 📖 1. Project Overview & Vision

**What it is:** A pocket-sized, battery-powered music player you build yourself — completely offline, plays only your own local files, no subscriptions, no Bluetooth, no internet dependency. Once music is on the SD card, the whole thing works standalone forever.

### The Core Experience:
* **MicroSD Storage:** Insert a microSD card loaded with music organized into folders (albums/playlists).
* **Pure UI:** A small color screen shows a simple, readable menu — first your list of folders, then the tracks inside whichever one you select.
* **Tactile Control:** Navigate the menu with physical buttons only — no touchscreen, no phone app, no companion software needed day-to-day.
* **Wired Audio:** Select a track and it plays through a dedicated headphone jack, wired directly (no wireless compression or dropouts).

### Playback Behavior:
* **Continuous Play:** Once a folder starts playing, songs auto-advance through it in order, then loop back to the first track and keep going — like a continuous album playthrough.
* **Menu Shortcut:** A quick triple-press gesture on the PREV button jumps back out to the folder menu at any time.
* **Track Looping:** A triple-press on the NEXT button locks the current song into repeat, with a small on-screen indicator `[LOOP]` so you know it's active.
* **Always Available:** Standard single-button controls for play/pause and volume up/down throughout.

### Power & Portability:
* **Modern Charging:** Rechargeable via USB-C, running off an internal 3.7V LiPo battery.
* **Monitoring:** Battery level is shown on the screen at all times.
* **Form Factor:** Designed to be genuinely pocket-portable — compact enough to carry casually, not a dev-board-on-a-breadboard situation.

---

## 🛠 2. Hardware Modules Used
1. **ESP32 38-Pin DevKit (Type-C):** The main brain of the player.
2. **1.8" Color TFT LCD Screen (ST7735):** With built-in Micro SD card slot on the back.
3. **UDA1334A I2S DAC:** Converts digital audio from the ESP32 into analog audio for headphones.
4. **TP4056 Battery Charger:** Safely charges the LiPo battery via USB.
5. **MT3608 5V Boost Converter:** Steps up the 3.7V battery voltage to 5V for the ESP32.
6. **AMS1117-3.3V LDO Regulator:** Steps down the 5V to a clean 3.3V exclusively for the Audio DAC.
7. **3.7V LiPo Battery (40x60x3mm):** Powers the system.
8. **5x Tactile Push Buttons (6x6x5mm):** For user input.

---

## 🔌 3. Complete Wiring Guide

### Power Subsystem (Battery & Regulators)
| From Module | Pin | ➔ | To Module | Pin |
| :--- | :--- | :--- | :--- | :--- |
| **Battery** | `B+` | ➔ | **TP4056** | `B+` |
| **Battery** | `B-` | ➔ | **TP4056** | `B-` |
| **TP4056** | `OUT+` | ➔ | **Boost Converter** | `VIN+` |
| **TP4056** | `OUT-` | ➔ | *(Common Ground)* | Connect to all GNDs below |
| **Boost Converter** | `VOUT+` | ➔ | **ESP32** | `5V` (VIN) |
| **Boost Converter** | `VOUT+` | ➔ | **LDO 3.3V** | `VIN` |
| **LDO 3.3V** | `OUT` | ➔ | **DAC (Audio)** | `VIN` |

*(Note: TP4056 `OUT-`, Boost `VOUT-`, LDO `GND`, ESP32 `GND`, TFT `GND`, and DAC `GND` should all be tied together to form a common ground network).*

### Audio DAC (UDA1334A)
| DAC Pin | ➔ | Connects To |
| :--- | :--- | :--- |
| `VIN` | ➔ | **LDO 3.3V** `OUT` |
| `GND` | ➔ | Common Ground |
| `AGND` | ➔ | Tie directly to DAC `GND` |
| `SCLK` | ➔ | Tie directly to DAC `GND` *(Required for I2S mode)* |
| `WSEL` | ➔ | **ESP32** `P25` |
| `BCLK` | ➔ | **ESP32** `P26` |
| `DIN` | ➔ | **ESP32** `P27` |

### TFT Display & SD Card (Shared SPI Bus)
*Note: The screen and the SD card share the exact same Clock and MOSI pins.*

| TFT / SD Pin | ➔ | Connects To | Notes |
| :--- | :--- | :--- | :--- |
| `VCC` | ➔ | **ESP32** `3V3` | Display power |
| `LED` | ➔ | Tie to TFT `VCC` | Backlight power |
| `GND` | ➔ | Common Ground | |
| `SCK` & `SD_SCK` | ➔ | **ESP32** `P18` | Shared SPI Clock |
| `SDA` & `SD_MOSI` | ➔ | **ESP32** `P23` | Shared SPI MOSI |
| `SD_MISO` | ➔ | **ESP32** `P19` | SD SPI MISO |
| `SD_CS` | ➔ | **ESP32** `P5` | SD Chip Select |
| `CS` | ➔ | **ESP32** `P22` | Display Chip Select |
| `A0` (DC) | ➔ | **ESP32** `P21` | Display Data/Command |
| `RESET` | ➔ | **ESP32** `P17` | Display Reset |

### Tactile Buttons & Battery Monitor
*Note: The buttons use the ESP32's internal pull-up resistors. Connect one leg of each button to the ESP32 pin, and the other leg to GND.*

| Feature | ➔ | Connects To |
| :--- | :--- | :--- |
| `PLAY/PAUSE` Button | ➔ | **ESP32** `P32` |
| `NEXT` Button | ➔ | **ESP32** `P33` |
| `PREV` Button | ➔ | **ESP32** `P14` |
| `VOL UP` Button | ➔ | **ESP32** `P13` |
| `VOL DOWN` Button | ➔ | **ESP32** `P12` |
| `BATT_ADC` Monitor | ➔ | **ESP32** `P34` (Via voltage divider) |

---

## 💻 4. Software Setup

### Required Libraries
1. `TFT_eSPI` (By Bodmer) - For the screen.
2. `ESP32-audioI2S` (By schreibfaul1) - For audio decoding.

### Arduino IDE Settings
Because the audio decoding library is very large, you must change the Partition Scheme in the Arduino IDE before uploading:
* Go to **Tools** ➔ **Partition Scheme** ➔ Select **"Huge APP (3MB No OTA / 1MB SPIFFS)"**

### TFT_eSPI Configuration
You must edit the `User_Setup.h` file inside the TFT_eSPI library folder so it matches our hardware. Ensure it contains these exact lines:
```cpp
#define ST7735_DRIVER
#define TFT_WIDTH  128
#define TFT_HEIGHT 160

#define TFT_CS   22
#define TFT_DC   21
#define TFT_RST  17
#define TFT_MOSI 23
#define TFT_SCLK 18
#define TFT_MISO 19

#define SPI_FREQUENCY 16000000
```
