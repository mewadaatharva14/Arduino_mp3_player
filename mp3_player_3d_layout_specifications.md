# MP3 Player 3D Layout - Standardized Physical Dimensions

Precise physical measurements and pin mappings, standardized for the 3D Layout Planner.

**Note:** The 1.8" TFT screen has its own SD card pins on the right side, which means it has a built-in Micro SD slot on the back. We can decide later whether to use the screen's slot or the separate blue SD module to save space.

---

## Conventions
*(Apply to every module section that uses this format)*

* **View:** TOP view (component side facing the viewer) unless a section says otherwise.
* **Origin:** Top-left corner of the board outline in that view. X increases to the right, Y increases downward. Units are mm.
* **Z axis:** Z = 0 is the bottom face of the PCB, and +Z points toward the top side.
* **Coordinates:** Pin or pad centre points. Values marked (est.) were estimated from photos and need caliper confirmation.
* **Mirroring:** Bottom-view photos are mirrored left-to-right. All positions below are already corrected to the top view.
* **Status values:** In use, Optional, Not in use. The planner skips modules marked Not in use.

---

## 1. ESP32 38-Pin DevKit (CP2102, Type-C)
* **Status:** In use
* **Board size:** 50 mm (L) x 25 mm (W)
* **Reference view:** TOP view, USB-C port facing TOP (USB-C end is Y = 0)
* **Origin:** Top-left corner of the board outline. X: 0 to 25, Y: 0 to 50
* **Pin type / direction:** Through-hole 2.54 mm male headers on the BOTTOM face, pins pointing straight DOWN (-Z). Solder joints are visible on the top face.
* **Pin pitch:** 2.54 mm
* **Pin rows:** 2 rows x 19 pins = 38 pins. Row spacing is about 22.4 mm centre to centre (est.). Left row X = 1.3, right row X = 23.7 (est.).
* **First pin Y:** 2.14 mm from the USB-C end (est., assumes symmetric margins). Pin n is at Y = 2.14 + (n - 1) x 2.54.
* **Intended layer:** Middle

### Left edge pins (X = 1.3), top to bottom

| # | Label | GPIO | Y (mm) | Notes |
|---|---|---|---|---|
| 1 | CLK | 6 | 2.14 | Internal flash, do not use |
| 2 | SD0 | 7 | 4.68 | Internal flash, do not use |
| 3 | SD1 | 8 | 7.22 | Internal flash, do not use |
| 4 | P15 | 15 | 9.76 | Strapping pin |
| 5 | P2 | 2 | 12.30 | Strapping pin |
| 6 | P0 | 0 | 14.84 | Strapping pin, BOOT button |
| 7 | P4 | 4 | 17.38 | |
| 8 | P16 | 16 | 19.92 | |
| 9 | P17 | 17 | 22.46 | |
| 10 | P5 | 5 | 25.00 | Strapping pin (used as SD CS in the sketch) |
| 11 | P18 | 18 | 27.54 | VSPI SCK |
| 12 | P19 | 19 | 30.08 | VSPI MISO |
| 13 | GND | - | 32.62 | Ground |
| 14 | P21 | 21 | 35.16 | I2C SDA default |
| 15 | RX | 3 | 37.70 | UART0 RX |
| 16 | TX | 1 | 40.24 | UART0 TX |
| 17 | P22 | 22 | 42.78 | I2C SCL default |
| 18 | P23 | 23 | 45.32 | VSPI MOSI |
| 19 | GND | - | 47.86 | Ground |

### Right edge pins (X = 23.7), top to bottom

| # | Label | GPIO | Y (mm) | Notes |
|---|---|---|---|---|
| 1 | 5V | - | 2.14 | 5V in/out (VIN) |
| 2 | GND | - | 4.68 | Silkscreen says GND. Verify with continuity test before relying |
| 3 | SD3 | 10 | 7.22 | Internal flash, do not use |
| 4 | SD2 | 9 | 9.76 | Internal flash, do not use |
| 5 | P13 | 13 | 12.30 | |
| 6 | GND | - | 14.84 | Ground |
| 7 | P12 | 12 | 17.38 | Strapping pin |
| 8 | P14 | 14 | 19.92 | |
| 9 | P27 | 27 | 22.46 | |
| 10 | P26 | 26 | 25.00 | |
| 11 | P25 | 25 | 27.54 | |
| 12 | P33 | 33 | 30.08 | |
| 13 | P32 | 32 | 32.62 | |
| 14 | P35 | 35 | 35.16 | Input only |
| 15 | P34 | 34 | 37.70 | Input only |
| 16 | VN | 39 | 40.24 | Input only (silkscreen: SVN) |
| 17 | VP | 36 | 42.78 | Input only (silkscreen: SVP) |
| 18 | EN | - | 45.32 | Reset / enable |
| 19 | 3V3 | - | 47.86 | 3.3V out |

**Quick pin order (same data as the tables, top to bottom, USB-C at top):**
* **Left edge:** CLK, SD0, SD1, P15, P2, P0, P4, P16, P17, P5, P18, P19, GND, P21, RX, TX, P22, P23, GND
* **Right edge:** 5V, GND, SD3, SD2, P13, GND, P12, P14, P27, P26, P25, P33, P32, P35, P34, VN, VP, EN, 3V3

### Components on the top side
* **USB-C receptacle:** Centred at X = 12.5, on the Y = 0 edge. Overhangs board edge by ~4–5 mm (est.). Plug opening is ~9 mm wide.
* **EN button:** About (19.8, 3.1) (est.), right of USB-C port.
* **BOOT (IO0) button:** About (5.0, 3.1) (est.), left of USB-C port.
* **CP2102 USB-UART chip and power regulator area:** Between USB-C port and ESP-32 module, roughly Y = 5 to 22.
* **ESP-32 module (ESP-WROOM-32 style, ~18 x 25.5 mm):** Centred at X = 12.5, roughly Y = 24 to 49 (est.).
* **PCB antenna:** Far end of module, roughly Y = 43 to 49 (est.). Keep clear of battery, metal parts, and wires.

### Components on the bottom side
* Only the two black 2.54 mm header bodies (~2.5 mm tall (est.)), running along both long edges.

---

## 2. 1.8" Color TFT LCD Screen
* **Status:** In use
* **Board size:** 58 mm (L) x 33 mm (W)
* **Reference view:** TOP view (display side facing user, pins along vertical edges)
* **Origin:** Top-left corner of the display PCB
* **Pin type / direction:** Solder pads (Top to bottom)
* **Intended layer:** Top (Facing user)

### Left edge pads (Display Interface), top to bottom

| # | Label | Type | Notes / Description |
|---|---|---|---|
| 1 | LED | Power In | Backlight power (+3.3V / +5V) |
| 2 | SCK | Input | SPI Clock |
| 3 | SDA | Input | SPI Data (MOSI) |
| 4 | A0 | Input | Data/Command control (DC) |
| 5 | RESET | Input | Display reset |
| 6 | CS | Input | Display Chip Select |
| 7 | GND | Power | Ground |
| 8 | VCC | Power In | System Power (3.3V / 5V) |

### Right edge pads (Built-in Micro SD Slot), top to bottom

| # | Label | Type | Notes / Description |
|---|---|---|---|
| 1 | SD_CS | Input | SD Card Chip Select |
| 2 | SD_MOSI | Input | SD SPI Data Input |
| 3 | SD_MISO | Output | SD SPI Data Output |
| 4 | SD_SCK | Input | SD SPI Clock |

---

## 3. Micro SD Card Reader Module (Separate Module)
* **Status:** Optional
* **Board size:** 42 mm (L) x 22 mm (W)
* **Reference view:** TOP view
* **Origin:** Top-left corner of the board outline
* **Pin type / direction:** Straight male header pins pointing straight DOWN (-Z)
* **Intended layer:** Flexible (Top or Bottom)

### Left edge pins, top to bottom

| # | Label | Type | Notes / Description |
|---|---|---|---|
| 1 | CS | Input | SPI Chip Select |
| 2 | SCK | Input | SPI Clock |
| 3 | MOSI | Input | SPI Master Out Slave In |
| 4 | MISO | Output | SPI Master In Slave Out |
| 5 | VCC | Power In | Power Supply (5V / 3.3V) |
| 6 | GND | Power | Ground |

---

## 4. UDA1334A I2S DAC (Audio Module)
* **Status:** In use
* **Board size:** 36 mm (L) x 25 mm (W)
* **Reference view:** TOP view (component side facing viewer), 3.5mm AUX Jack overhang on the RIGHT edge
* **Origin:** Top-left corner of the board outline
* **Pin type / direction:** Solder pads / headers along top and bottom long edges
* **Intended layer:** Flexible (Middle / Top)

### Top edge pads, left to right

| # | Label | Type | Notes / Description |
|---|---|---|---|
| 1 | SCLK | Input | System Clock (optional; internal PLL active if disconnected) |
| 2 | SF1 | Input | Format select bit 1 |
| 3 | MUTE | Input | High = Mute audio output |
| 4 | SF0 | Input | Format select bit 0 |
| 5 | PLL | Input | Phase-Locked Loop control |
| 6 | DEEM | Input | De-emphasis filter control |

### Bottom edge pads, left to right

| # | Label | Type | Notes / Description |
|---|---|---|---|
| 1 | VIN | Power In | 3.3V – 5V Power Input |
| 2 | 3VO | Power | Ground |
| 3 | GND | Power Out | Regulated 3.3V output from internal LDO |
| 4 | WSEL | Input | Word Select / LRCLK (Left/Right Clock) |
| 5 | DIN | Input | Serial Data Input |
| 6 | BCLK | Input | Bit Clock |
| 7 | LOUT | Analog Out | Left Channel Audio Output |
| 8 | AGND | Power | Analog Ground |
| 9 | ROUT | Analog Out | Right Channel Audio Output |

### Physical components & overhangs
* **3.5mm Female AUX Jack:** Centered on the RIGHT edge, extending ~5–6 mm beyond PCB edge.
* **Capacitors & UDA1334A IC:** Surface-mount components positioned on the left-central section of top side.

---

## 5. TP4056 Battery Charger & Protection Module
* **Status:** In use
* **Board size:** 28 mm (L) x 17 mm (W)
* **Reference view:** TOP view (component side facing viewer), USB-C port on the LEFT edge
* **Origin:** Top-left corner of the board outline
* **Pin type / direction:** Solder pads (Top view)
* **Intended layer:** Bottom

### Left edge pads (USB-C side)

| # | Label | Type | Status | Notes / Description |
|---|---|---|---|---|
| 1 | IN+ | Power In | Not in use | Optional 5V power input pad |
| 2 | USB-C | Connector | Not in use | USB-C Power input port (left edge) |
| 3 | IN- | Power | Not in use | Optional ground input pad |

### Right edge pads, top to bottom

| # | Label | Type | Status | Notes / Description |
|---|---|---|---|---|
| 1 | OUT+ | Power Out | In use | Positive output to boost module / system rail |
| 2 | B+ | Battery | In use | Positive lead to LiPo Battery |
| 3 | B- | Battery | In use | Negative lead to LiPo Battery |
| 4 | OUT- | Power Out | In use | Ground output to system ground rail |

---

## 6. LiPo Battery
* **Status:** In use
* **Board size:** 60 mm (L) x 40 mm (W) x 3 mm (Thickness)
* **Reference view:** TOP view
* **Origin:** Top-left corner of battery outline
* **Pin type / direction:** Red (+ / B+) and Black (- / B-) wire leads exiting top edge
* **Intended layer:** Bottom

---

## 7. Tactile Buttons (x5)
* **Status:** In use
* **Dimensions (Per Button):** 6 mm x 6 mm x 5 mm
* **Layout:** Flexible (e.g., D-pad style or inline row)
* **Pin type / direction:** Through-hole / SMT legs
* **Intended layer:** Top (Facing user)

---

## 8. MT3608 DC-DC Step-Up Boost Converter Module
* **Status:** In use
* **Board size:** 36 mm (L) x 17 mm (W)
* **Reference view:** TOP view (component side facing viewer), potentiometer on left, "220" inductor on right
* **Origin:** Top-left corner of the board outline
* **Pin type / direction:** Solder pads on left and right edges
* **Intended layer:** Flexible (Bottom / Middle)

### Left edge pads (Output side)

| # | Label | Type | Status | Notes / Description |
|---|---|---|---|---|
| 1 | VOUT+ | Power Out | In use | Positive boosted output voltage |
| 2 | VOUT- | Power Out | In use | Output ground / common ground |

### Right edge pads (Input side)

| # | Label | Status | Notes / Description |
|---|---|---|---|
| 1 | VIN+ | Power In | In use | Positive input voltage (from TP4056 OUT+) |
| 2 | VIN- | Power | In use | Input ground (from TP4056 OUT-) |

---

## 9. AMS1117-3.3 LDO Step-Down Power Supply Module
* **Status:** In use
* **Board size:** 12 mm (L) x 9 mm (W)
* **Reference view:** TOP view (component side facing viewer), 3-pin male header on the LEFT edge
* **Origin:** Top-left corner of the board outline
* **Pin type / direction:** 2.54 mm right-angle male header pins pointing LEFT / OUTWARD
* **Intended layer:** Flexible (Middle / Bottom)

### Header pins edge (Top to bottom)

| # | Label | Type | Status | Notes / Description |
|---|---|---|---|---|
| 1 | VIN | Power In | In use | 5V DC power input |
| 2 | OUT | Power Out | In use | Regulated 3.3V DC power output |
| 3 | GND | Power | In use | Common ground |