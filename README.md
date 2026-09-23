# ESP32 Portable MP3 Player 🎵

This document contains everything about the ESP32 Portable MP3 Player project, including the hardware list, power architecture, full pinout wiring guide, and the complete Arduino codebase.

## 📌 1. Project Overview
We are building a fully portable, battery-powered MP3 player using an ESP32 microcontroller. The device features:
- A **1.8" Color TFT display** for a graphical user interface (folder and track menus).
- **Micro SD Card storage** (using the screen's built-in slot) to hold MP3 files.
- An **I2S Audio DAC** for high-quality, noise-free audio output.
- A **custom power subsystem** to safely charge a LiPo battery, boost the voltage for the ESP32, and provide clean isolated power to the audio DAC.
- **5 Tactile Buttons** for playback control and menu navigation.

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

---

## 🚀 5. The Complete Arduino Source Code

```cpp
/*
  DIY ESP32 Portable MP3 Player — Full Menu Build
  --------------------------------------------------
  Behavior:
    - Boot -> shows folder list, cursor highlights one folder
    - Next/Prev: move cursor up/down in the current list
    - Play: select highlighted item (enter folder, or start playing track)
    - While a track is PLAYING:
        - Play: pause/resume
        - Press Prev 3 times in quick succession -> stop, return to folder menu
        - Press Next 3 times in quick succession -> toggle loop-current-track
        - Otherwise, playback auto-advances through the folder in order
    - Vol Up / Vol Down always adjust volume, in any state.
*/

#include <SPI.h>
#include <SD.h>
#include <TFT_eSPI.h>
#include "Audio.h"

// ---------- Pin definitions ----------

// SD Card (Using Shared VSPI pins 18, 19, 23 configured by TFT_eSPI)
#define SD_CS     5

// I2S Audio DAC (UDA1334A)
#define I2S_WSEL  25
#define I2S_BCLK  26
#define I2S_DOUT  27

// Buttons (Using pins with internal pull-ups)
#define BTN_PLAY   32
#define BTN_NEXT   33
#define BTN_PREV   14
#define BTN_VOLUP  13
#define BTN_VOLDN  12

// Battery Monitor
#define BATT_ADC   34

// ---------- Objects ----------
TFT_eSPI tft = TFT_eSPI();
Audio audio;

// ---------- App state ----------
enum AppState { STATE_BROWSE_FOLDERS, STATE_BROWSE_TRACKS, STATE_PLAYING };
AppState appState = STATE_BROWSE_FOLDERS;

// ---------- SD contents ----------
String folders[10];
int folderCount = 0;
int folderCursor = 0;
int folderScroll = 0;

String tracks[200];
int trackCount = 0;
int trackCursor = 0;
int trackScroll = 0;

int currentFolder = 0;
int currentTrack = 0;

bool isPlaying = false;
bool isLooping = false;
int volume = 15;

// ---------- Button handling ----------
unsigned long lastButtonCheck = 0;
const unsigned long debounceMs = 180;

int comboButtonPin = -1;
int comboCount = 0;
unsigned long comboLastTime = 0;
const unsigned long comboWindowMs = 1200;

// ---------- Battery ----------
unsigned long lastBattCheck = 0;
int batteryPercent = 0;

// ---------- Layout ----------
const int lineHeight = 12;
const int listTopY = 16;

// ---------- Forward declarations ----------
void scanFolders();
void scanTracks(String folderPath);
void selectAndPlay(int index);
void continuePlayback(int index);
void handlePress(int pin);
void checkButtons();
void updateBattery();
void drawFolderMenu();
void drawTrackMenu();
void drawNowPlaying();
void refreshCurrentScreen();
void showError(const char *msg);
String baseName(String path);
bool isAudioFile(String name);

void setup() {
  Serial.begin(115200);
  delay(300);
  Serial.println("\n--- Booting MP3 Player ---");

  // Initialize buttons with internal pull-ups
  pinMode(BTN_PLAY, INPUT_PULLUP);
  pinMode(BTN_NEXT, INPUT_PULLUP);
  pinMode(BTN_PREV, INPUT_PULLUP);
  pinMode(BTN_VOLUP, INPUT_PULLUP);
  pinMode(BTN_VOLDN, INPUT_PULLUP);
  pinMode(BATT_ADC, INPUT);

  // Initialize Display
  tft.init();
  tft.setRotation(1);
  tft.fillScreen(TFT_BLACK);
  tft.setTextColor(TFT_WHITE, TFT_BLACK);
  tft.setTextSize(1);
  tft.setCursor(4, 4);
  tft.println("Booting...");

  // Initialize SD Card
  Serial.print("Mounting SD card... ");
  if (!SD.begin(SD_CS)) {
    Serial.println("FAILED");
    showError("SD card error");
    while (true) delay(1000);
  }
  Serial.println("OK");

  // Scan SD Card
  scanFolders();
  if (folderCount == 0) {
    showError("No folders found");
    while (true) delay(1000);
  }

  // Initialize Audio
  audio.setPinout(I2S_BCLK, I2S_WSEL, I2S_DOUT);
  audio.setVolume(volume);

  updateBattery();
  drawFolderMenu();
}

void loop() {
  audio.loop();
  checkButtons();
  updateBattery();
}

// ---------- SD scanning ----------
void scanFolders() {
  folderCount = 0;
  File root = SD.open("/");
  if (!root) return;

  File entry = root.openNextFile();
  while (entry && folderCount < 10) {
    if (entry.isDirectory()) {
      String name = String(entry.name());
      if (!name.startsWith(".") && name != "System Volume Information") {
        folders[folderCount] = String("/") + name;
        folderCount++;
      }
    }
    entry = root.openNextFile();
  }
  root.close();
  Serial.printf("Found %d folders\n", folderCount);
}

bool isAudioFile(String name) {
  String lower = name;
  lower.toLowerCase();
  return lower.endsWith(".mp3") || lower.endsWith(".mp4") || lower.endsWith(".m4a");
}

void scanTracks(String folderPath) {
  trackCount = 0;
  File dir = SD.open(folderPath);
  if (!dir) return;

  File entry = dir.openNextFile();
  while (entry && trackCount < 200) {
    String name = String(entry.name());
    if (!entry.isDirectory() && isAudioFile(name)) {
      tracks[trackCount] = folderPath + "/" + name;
      trackCount++;
    }
    entry = dir.openNextFile();
  }
  dir.close();
  Serial.printf("Found %d tracks in %s\n", trackCount, folderPath.c_str());
}

// ---------- Playback ----------
void selectAndPlay(int index) {
  if (trackCount == 0) return;
  currentTrack = index;
  isLooping = false;
  audio.connecttoFS(SD, tracks[currentTrack].c_str());
  isPlaying = true;
  appState = STATE_PLAYING;
  drawNowPlaying();
}

void continuePlayback(int index) {
  if (trackCount == 0) return;
  currentTrack = index;
  audio.connecttoFS(SD, tracks[currentTrack].c_str());
  isPlaying = true;
  drawNowPlaying();
}

// ---------- Buttons ----------
void checkButtons() {
  if (millis() - lastButtonCheck < debounceMs) return;

  int pins[5] = { BTN_PLAY, BTN_NEXT, BTN_PREV, BTN_VOLUP, BTN_VOLDN };
  for (int i = 0; i < 5; i++) {
    if (digitalRead(pins[i]) == LOW) {
      lastButtonCheck = millis();
      handlePress(pins[i]);
      break;
    }
  }
}

void handlePress(int pin) {
  if (pin == BTN_VOLUP) {
    volume = min(21, volume + 1);
    audio.setVolume(volume);
    refreshCurrentScreen();
    return;
  }
  if (pin == BTN_VOLDN) {
    volume = max(0, volume - 1);
    audio.setVolume(volume);
    refreshCurrentScreen();
    return;
  }

  bool triggered = false;
  if (appState == STATE_PLAYING && (pin == BTN_NEXT || pin == BTN_PREV)) {
    if (pin == comboButtonPin && (millis() - comboLastTime) <= comboWindowMs) {
      comboCount++;
    } else {
      comboCount = 1;
    }
    comboButtonPin = pin;
    comboLastTime = millis();
    if (comboCount >= 3) {
      triggered = true;
      comboCount = 0;
      comboButtonPin = -1;
    }
  } else {
    comboCount = 0;
    comboButtonPin = -1;
  }

  switch (appState) {
    case STATE_BROWSE_FOLDERS:
      if (pin == BTN_NEXT) {
        folderCursor = (folderCursor + 1) % folderCount;
        drawFolderMenu();
      } else if (pin == BTN_PREV) {
        folderCursor = (folderCursor - 1 + folderCount) % folderCount;
        drawFolderMenu();
      } else if (pin == BTN_PLAY) {
        currentFolder = folderCursor;
        scanTracks(folders[currentFolder]);
        if (trackCount == 0) {
          showError("No audio in folder");
          delay(1200);
          drawFolderMenu();
        } else {
          trackCursor = 0;
          trackScroll = 0;
          appState = STATE_BROWSE_TRACKS;
          drawTrackMenu();
        }
      }
      break;

    case STATE_BROWSE_TRACKS:
      if (pin == BTN_NEXT) {
        trackCursor = (trackCursor + 1) % trackCount;
        drawTrackMenu();
      } else if (pin == BTN_PREV) {
        trackCursor = (trackCursor - 1 + trackCount) % trackCount;
        drawTrackMenu();
      } else if (pin == BTN_PLAY) {
        selectAndPlay(trackCursor);
      }
      break;

    case STATE_PLAYING:
      if (pin == BTN_PLAY) {
        audio.pauseResume();
        isPlaying = !isPlaying;
        drawNowPlaying();
      } else if (pin == BTN_NEXT && triggered) {
        isLooping = !isLooping;
        drawNowPlaying();
      } else if (pin == BTN_PREV && triggered) {
        audio.stopSong();
        isPlaying = false;
        appState = STATE_BROWSE_FOLDERS;
        drawFolderMenu();
      }
      break;
  }
}

void refreshCurrentScreen() {
  switch (appState) {
    case STATE_BROWSE_FOLDERS: drawFolderMenu(); break;
    case STATE_BROWSE_TRACKS:  drawTrackMenu();  break;
    case STATE_PLAYING:        drawNowPlaying(); break;
  }
}

// ---------- Battery ----------
void updateBattery() {
  if (lastBattCheck != 0 && millis() - lastBattCheck < 30000) return;
  lastBattCheck = millis();

  int raw = analogRead(BATT_ADC);
  float voltage = (raw / 4095.0) * 3.3 * 2.0;
  int pct = (int)((voltage - 3.3) / (4.2 - 3.3) * 100);
  batteryPercent = constrain(pct, 0, 100);
}

// ---------- Display helpers ----------
String baseName(String path) {
  int slashIdx = path.lastIndexOf('/');
  return (slashIdx >= 0) ? path.substring(slashIdx + 1) : path;
}

void drawHeader(const char *title) {
  tft.fillRect(0, 0, tft.width(), 14, TFT_NAVY);
  tft.setTextColor(TFT_WHITE, TFT_NAVY);
  tft.setCursor(2, 3);
  tft.print(title);
  tft.setCursor(tft.width() - 40, 3);
  tft.printf("B:%d%%", batteryPercent);
}

void drawFolderMenu() {
  tft.fillScreen(TFT_BLACK);
  drawHeader("Folders");

  int visibleRows = (tft.height() - listTopY) / lineHeight;
  if (folderCursor < folderScroll) folderScroll = folderCursor;
  if (folderCursor >= folderScroll + visibleRows) folderScroll = folderCursor - visibleRows + 1;

  for (int i = 0; i < visibleRows && (i + folderScroll) < folderCount; i++) {
    int idx = i + folderScroll;
    int y = listTopY + i * lineHeight;
    bool selected = (idx == folderCursor);
    if (selected) {
      tft.fillRect(0, y, tft.width(), lineHeight, TFT_WHITE);
      tft.setTextColor(TFT_BLACK, TFT_WHITE);
    } else {
      tft.setTextColor(TFT_WHITE, TFT_BLACK);
    }
    tft.setCursor(4, y + 2);
    tft.print(baseName(folders[idx]));
  }
}

void drawTrackMenu() {
  tft.fillScreen(TFT_BLACK);
  drawHeader(baseName(folders[currentFolder]).c_str());

  int visibleRows = (tft.height() - listTopY) / lineHeight;
  if (trackCursor < trackScroll) trackScroll = trackCursor;
  if (trackCursor >= trackScroll + visibleRows) trackScroll = trackCursor - visibleRows + 1;

  for (int i = 0; i < visibleRows && (i + trackScroll) < trackCount; i++) {
    int idx = i + trackScroll;
    int y = listTopY + i * lineHeight;
    bool selected = (idx == trackCursor);
    if (selected) {
      tft.fillRect(0, y, tft.width(), lineHeight, TFT_WHITE);
      tft.setTextColor(TFT_BLACK, TFT_WHITE);
    } else {
      tft.setTextColor(TFT_WHITE, TFT_BLACK);
    }
    tft.setCursor(4, y + 2);
    tft.print(baseName(tracks[idx]));
  }
}

void drawNowPlaying() {
  tft.fillScreen(TFT_BLACK);
  drawHeader(isPlaying ? "Playing" : "Paused");

  tft.setTextColor(TFT_GREEN, TFT_BLACK);
  tft.setCursor(4, 24);
  tft.println(baseName(tracks[currentTrack]));

  tft.setTextColor(TFT_WHITE, TFT_BLACK);
  tft.setCursor(4, 44);
  tft.printf("Track %d / %d", currentTrack + 1, trackCount);

  tft.setCursor(4, 60);
  tft.printf("Vol: %d / 21", volume);

  if (isLooping) {
    tft.setTextColor(TFT_YELLOW, TFT_BLACK);
    tft.setCursor(4, 76);
    tft.println("[LOOP]");
  }

  tft.setTextColor(TFT_CYAN, TFT_BLACK);
  tft.setCursor(4, tft.height() - 12);
  tft.println("Prev x3: menu  Next x3: loop");
}

void showError(const char *msg) {
  tft.fillScreen(TFT_BLACK);
  tft.setTextColor(TFT_RED, TFT_BLACK);
  tft.setCursor(4, 20);
  tft.println(msg);
  Serial.println(msg);
}

// ---------- ESP32-audioI2S callbacks ----------
void audio_info(const char *info) {
  Serial.print("audio_info: ");
  Serial.println(info);
}

void audio_id3data(const char *info) {
  Serial.print("id3data: ");
  Serial.println(info);
}

void audio_eof_mp3(const char *info) {
  Serial.print("eof_mp3: ");
  Serial.println(info);

  if (isLooping) {
    continuePlayback(currentTrack);
  } else {
    int next = currentTrack + 1;
    if (next >= trackCount) next = 0;
    continuePlayback(next);
  }
}
```
