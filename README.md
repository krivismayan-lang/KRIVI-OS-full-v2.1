KRIVI OS
A touch‑driven micro‑operating‑system for the ESP32‑2432S028 "Cheap Yellow Display" (CYD) — a tiny 320×240 touchscreen computer that boots into a desktop launcher and runs 50+ self‑contained apps: games and emulators, media (Bluetooth audio + internet radio), built‑in‑sensor tools, utilities, radio/wireless labs, graphics demos, and system tools.

Current version: 1.8.0

1. What you need (materials)
Required

Item	Notes
ESP32‑2432S028 "CYD" board	The "Cheap Yellow Display." Get the ILI9341 panel version (most common). Some boards ship with an ST7789 panel — a separate _ST7789 image is provided for those.
microSD card — more than 8 GB	16 GB is ideal. Must be formatted FAT32 (cards over 32 GB are often exFAT out of the box — reformat them to FAT32). Holds games, music, movies, icons and settings.
USB data cable	Matches your board's port (usually micro‑USB or USB‑C). Must be a data cable, not charge‑only.
A computer	Windows, macOS or Linux, to flash the firmware.
Optional (only for certain apps)

Item	Enables
A 2.4 GHz WiFi network	Internet Radio, OTA Update, Weather, World Clock/NTP, ISS Globe, wireless‑lab apps
Bluetooth headphones / speaker	The Bluetooth + MP3 player (A2DP)
Raspberry Pi + USB cable	The "Pi Link" apps
A second ESP32 running KRIVI	ESP‑NOW Radio peer messaging
The CYD has a built‑in screen, touch, speaker, microphone, light sensor (LDR), RGB LED and SD slot — so the core kit is just board + a FAT32 SD card + a USB cable.

2. Quick start — set it up in order
Step 1 — Format the SD card (FAT32)
Format your microSD card as FAT32. (Windows: the built‑in formatter for ≤32 GB, or "SD Card Formatter" for larger; macOS: Disk Utility → MS‑DOS (FAT); Linux: mkfs.vfat -F 32.)

Step 2 — Put the KRIVI files on the card
Copy the contents of the project's sd_card/ folder onto the root of the card:

/roms/            NES (.nes) and CHIP-8 (.ch8) ROMs      (add your own)
/music/           your .mp3 files                         (add your own)
/data/movies/     converted .avi (MP4 -> AVI) clips        (add your own)
/data/video/      raw .mjpeg clips                         (add your own)
/icons/           launcher + weather icons                (from sd_card/)
/sys/             settings + app manifests                (from sd_card/)
At minimum copy /icons/ and /sys/. The /roms, /music, /data folders can start empty.

Step 3 — Flash the firmware
Connect the board over USB, then flash the pre‑built image from prebuilt/ to offset 0x0:

esptool.py --chip esp32 --baud 921600 write_flash 0x0 prebuilt/KRIVI_OS_v1.8.0_ILI9341.bin
No esptool? pip install esptool.
Add --port COM5 (Windows) or --port /dev/ttyUSB0 (macOS/Linux) if it isn't auto‑detected.
If flashing won't start, hold the BOOT button while it connects.
ST7789‑panel board? Use the _ST7789 image.
Step 4 — Insert the card and power on
Insert the microSD card, power the board over USB. KRIVI boots to the desktop launcher.

Step 5 — First‑run settings
On page 6 → System Settings:

Drag Brightness and Volume.
Tap SET WIFI and follow the prompts (see §3 for open and school/enterprise networks).
Tap PASSWORD to set a device lock (optional — see §4).
Tap SAVE (writes /sys/settings.json), then SYNC NTP to set the clock.
3. WiFi — home, open, and school/enterprise
KRIVI joins three kinds of network, configured from System Settings → SET WIFI or by editing /sys/settings.json:

// Home / WPA2 password network:
"wifi": { "ssid": "MyHome", "pass": "mypassword" }

// Open network (cafe, library) - leave the password blank:
"wifi": { "ssid": "CafeFreeWiFi" }

// School / enterprise (WPA2-Enterprise, PEAP/TTLS) - add a username:
"wifi": { "ssid": "eduroam", "user": "you@school.edu", "pass": "yourpassword" }
Adding a user turns on enterprise mode (what most school and university networks use). An optional "identity" sets a separate anonymous outer identity. A blank password means an open network.

4. Device password (lock)
KRIVI can require a password at boot. It's stored on the SD card — blank means no lock:

"security": { "pass": "1234" }
Set or clear it from System Settings → PASSWORD, or edit settings.json directly. When a password is set, a lock screen appears on boot and must be entered before the launcher opens.

5. The apps
Page 1 — Gaming & Emulation
App	What it does
NES Emulator	Plays .nes ROMs from /roms (mappers 0/1/2/3/4). No PSRAM, so only games up to ~180 KB fit.
Arcade	A single menu of 16 classic games — see §6.
CHIP‑8 Virtual Machine	Runs classic CHIP‑8 .ch8 programs.
Page 2 — Media & Sensors
App	What it does
Bluetooth + MP3	Three tabs: pair Bluetooth headphones/speakers (A2DP, auto‑reconnect); play /music MP3s to them; FTP‑upload music over WiFi.
Internet Radio	Streams an MP3 radio station over WiFi to the speaker. Presets built in; add your own in /sys/radio.txt as `Name
Light Meter	Live ambient‑light reading (0–100 %) from the built‑in LDR, with a rolling graph.
Auto‑Brightness	Matches the screen backlight to the room using the LDR; drag bars to set min/max.
Night Light	Turns the RGB LED into a warm light that switches on when the room goes dark.
Page 3 — Utilities & Time
Stopwatch · Pomodoro Timer · Calculator · Calendar · Text Editor · Hex Viewer · Alarm Clock · World Clock · Currency Converter · Unit Converter.

Page 4 — Pi Link & Radio
Pi Performance Monitor · Pi Bash Console · Pi Remote Trackpad · Pi Shell Keyboard · Pi Safe Shutdown · Bluetooth Macro Pad (BLE HID) · BLE Keyboard (BLE HID) · WiFi Analyzer · Ping Monitor · ESP‑NOW Radio.

Page 5 — Graphics & Sandbox
Sketchpad · 3D Rainbow Warp · Sandbox (see §7) · Mandelbrot · Video & MP4 Player · Virtual Pet (fully redrawn graphics) · Touch Diagnostics · System Manual · Demoscene FX · Matrix Digital Rain · Starfield Screensaver · Weather.

Page 6 — Wireless Lab & System
Channel Congestion · BLE Scanner · Signal Mapper · LAN Discovery · Net Health · 3D ISS Globe Radar · System Settings · NVS Flash Editor · Desk Clock · Noise Floor Log · OTA Update (see §8).

6. The Arcade (16 games)
One scrollable menu on the Gaming page, 4 per page. Tap to play, tap EXIT to return. These games aren't duplicated as separate home‑screen tiles.

Flappy Bird · Snake · Tetris · Pong · Breakout · Minesweeper · Chess / Tic‑Tac‑Toe · 3D Dice · 2048 · Connect Four (vs bot or 2P) · Checkers · Reversi (vs bot or 2P) · Simon · Sudoku · Solitaire (Klondike) · Space Invaders.

(Ludo and Snakes & Ladders were removed; the eight bold titles are new in 1.8.0.)

7. Sandbox (falling‑sand, 33 materials)
A cellular‑automaton physics playground. Materials are organised into category tabs (POWDER · LIQUID · SOLID · GAS · NATURE · SPECL) so the toolbar stays tidy; pick a tab, then a material, then draw. Adjustable brush size, erase and clear.

Powders: Sand, Salt, Gunpowder, Snow, Ash, Soil
Liquids: Water, Oil, Acid, Lava, Liquid Nitrogen, Midas, Sap
Solids: Stone, Wood, Glass, Metal, Clay, Wall, Gold
Gases: Fire, Smoke, Steam, Methane
Nature: Ice, Plant, Virus, Uranium, Cloner
Special: TNT, C4, Electricity, Antimatter
Reactions include: fire spreading through wood/oil/plant/gunpowder/methane; water + fire → steam; lava + water → stone; lava melting sand → glass and ice/snow → water; acid dissolving solids; salt and liquid nitrogen freezing/melting; TNT/C4 exploding; Midas turning things to gold; cloner duplicating; electricity conducting through metal and water; virus spreading to organics; and antimatter annihilating anything it touches.

8. OTA Update (and the 4 MB reality)
The OTA Update app flashes new firmware over WiFi: it joins your network, shows a web address, and you upload a KRIVI app image (KriviOS.ino.bin) from a browser.

Important: true over‑the‑air updating needs a second app (OTA) partition to flash into. KRIVI is ~2.3 MB, and a 4 MB CYD cannot hold two copies that big, so this build uses a single‑app layout — the app detects this and tells you plainly instead of failing mid‑upload. OTA works on an 8/16 MB CYD or a slimmed build; on a standard 4 MB board, update over USB (or from the SD card) instead.

9. Memory — how KRIVI stays inside a tiny RAM budget
~320 KB of SRAM, no PSRAM. KRIVI keeps each app inside that:

One app at a time, fully reclaimed on exit. Code lives in flash and costs zero RAM until opened; on exit the launcher frees its memory, stops audio and turns radios off.
Strip rendering for big graphics (Flappy, 3D Dice, ISS globe) — a few KB at a time instead of a full‑screen buffer.
SD card as a RAM helper — the video player decodes frames straight from SD when RAM is tight; SwapBuf parks large block‑accessed buffers on the card; bigAlloc retries once to ride out fragmentation blips.
The Sandbox's three 12 KB grids and all decoders go through bigAlloc, so a tight moment degrades instead of crashing.
A current build is ~73 % flash and ~29 % static RAM, leaving ~230 KB of heap.

10. Building from source
Board core: ESP32 Arduino 2.0.17 (3.x removed the built‑in‑DAC I²S mode this project needs).
FQBN: espressif:esp32:esp32 with PartitionScheme=huge_app, FlashMode=qio, CPUFreq=240, FlashSize=4M.
Libraries:
TFT_eSPI (display; configured via TFT_eSPI_User_Setup.h)
TJpg_Decoder (video frames)
ArduinoJson
ESP32‑A2DP (Bluetooth audio)
NimBLE‑Arduino 1.4.x (the BLE HID apps use the 1.4 API, not 2.x)
The Helix MP3 decoder is bundled in‑tree under KriviOS/src/helix/.
Open KriviOS/KriviOS.ino, select the settings above, and compile.

11. Troubleshooting
Blank/garbled screen → try the _ST7789 image.
"NO SD CARD" / missing files → card isn't FAT32, isn't seated, or the folders aren't at the card root.
Locked out → the password is in /sys/settings.json under "security":{"pass":".."}; clear it there to remove the lock.
Internet Radio won't play → MP3 (Icecast/SHOUTcast) streams only, not AAC/OGG; check WiFi and the URL in /sys/radio.txt.
OTA says "no OTA partition" → expected on a 4 MB CYD; update over USB/SD.
Flashing won't start → use a data cable, a lower --baud (e.g. 460800), and hold BOOT.
12. Version highlights
1.8.0 — Media & Sensors page (Bluetooth + MP3, Internet Radio, Light Meter, Auto‑Brightness, Night Light); removed the old audio/decibel apps; Arcade expanded to 16 games (added 2048, Connect Four, Checkers, Reversi, Simon, Sudoku, Solitaire, Space Invaders; removed Ludo & Snakes & Ladders); Falling Sand → Sandbox with 33 materials and a tabbed picker; redrawn Virtual Pet; WiFi open + school/enterprise support; device password lock; OTA Update app.
1.7.x — Arcade games hub; SD‑card RAM helper (video‑from‑SD, SwapBuf); duplicate‑tile cleanup.
1.6.2 — major RAM pass: strip rendering, trimmed decoder/media buffers.
KRIVI OS runs entirely on the device — no companion app. Everything above is on the 320×240 touchscreen in your hand.
