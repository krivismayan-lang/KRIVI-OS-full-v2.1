KRIVI OS

A touch‑driven micro‑operating‑system for the ESP32‑2432S028 "Cheap Yellow Display" (CYD) — a tiny 320×240 touchscreen computer that boots into a desktop launcher and runs 50+ self‑contained apps: games and emulators, media (Bluetooth audio + internet radio), built‑in‑sensor tools, utilities, radio/wireless labs, graphics demos, and system tools.

Current version: 1.9.8

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
/maps/tiles/      Maps tiles per style: <style>/z/x/y.png  (auto-filled; see §7b)
/data/maps/       Maps favourites (favorites.txt)
/data/video/      raw .mjpeg clips                         (add your own)
/icons/           launcher + weather icons                (from sd_card/)
/sys/             settings + app manifests                (from sd_card/)

At minimum copy /icons/ and /sys/. The /roms, /music, /data folders can start empty.

Step 3 — Flash the firmware

Connect the board over USB, then flash the pre‑built image from prebuilt/ to offset 0x0:

bash
esptool.py --chip esp32 --baud 921600 write_flash 0x0 prebuilt/KRIVI_OS_v1.9.8_ILI9341.bin
No esptool? pip install esptool.
Add --port COM5 (Windows) or --port /dev/ttyUSB0 (macOS/Linux) if it isn't auto‑detected.
If flashing won't start, hold the BOOT button while it connects.
ST7789‑panel board? Use the _ST7789 image.
Prefer the Arduino IDE? See §10 — open KriviOS/KriviOS.ino and upload.
Step 4 — Insert the card and power on

Insert the microSD card, power the board over USB. KRIVI boots to the desktop launcher.

Step 5 — First‑run settings

On page 6 → System Settings:

Drag Brightness and Volume.
Tap WI‑FI NETWORKS, then tap your network in the list (see §3).
Tap PASSWORD to set a device lock (optional — see §4).
Tap SAVE (writes /sys/settings.json), then SYNC NTP to set the clock.
3. WiFi — home, open, and school/enterprise

The easy way: System Settings → WI‑FI NETWORKS scans and lists the networks around you (signal bars, 🔒 for locked ones). Tap one: open networks join straight away, locked ones ask for the password, school username+password networks ask for both. It's saved automatically. After joining it checks for real internet and tells you if the network needs a web sign‑in page (with the board's MAC address to give your IT person). OTHER joins a hidden network.

You can also edit /sys/settings.json by hand. KRIVI joins three kinds of network:

jsonc
// Home / WPA2 password network:
"wifi": { "ssid": "MyHome", "pass": "mypassword" }

// Open network (cafe, library) - leave the password blank:
"wifi": { "ssid": "CafeFreeWiFi" }

// School / enterprise (WPA2-Enterprise, PEAP/TTLS) - add a username:
"wifi": { "ssid": "eduroam", "user": "you@school.edu", "pass": "yourpassword" }

Adding a user turns on enterprise mode (what most school and university networks use). An optional "identity" sets a separate anonymous outer identity. A blank password means an open network.

4. Device password (lock)

KRIVI can require a password at boot. It's stored on the SD card — blank means no lock:

jsonc
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
AI Assistant	Chat with Google Gemini (free key in settings.json → ai.key, or tap KEY).
Trading	Candlestick chart + watchlist; crypto with no key, stocks with a free Finnhub key.
News	Headlines with the real article photo; feeds in /sys/news.txt.
FrogFind!	A portrait web browser (see §7c).
Page 3 — Utilities & Time

Stopwatch · Pomodoro Timer · Calculator · Calendar · Text Editor · Hex Viewer · Alarm Clock · World Clock · Currency Converter · Unit Converter · Maps (see §7b).

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

7b. Maps (Google-style map, app 86)

Search places, get car / bike / walking directions and keep favourites, all on the CYD, with a Google-Maps-like look (white roads on a light map, blue dot, red pin, yellow star for saved places, white +/− buttons). It needs no API key: it uses the free CARTO "Voyager" map style (built from OpenStreetMap data) plus Nominatim (search), OSRM at routing.openstreetmap.de (routes) and a free IP-location lookup. Google's own tiles can't be used (paid key, and their terms forbid caching/redrawing them). It has its own tiny PNG decoder and JSON reader, so there are no extra libraries to install.

Map: drag to pan, + / − to zoom, tap to drop a destination pin. MENU → Style switches Voyager (Google-like) / Light grey / OpenStreetMap standard. Tiles you view are cached on the SD card in /maps/tiles/<style>/z/x/y.png (each style has its own folder).
FIND searches places/addresses (results biased to where the map is). ROUTE asks car / bike / foot, draws the route and shows distance and time; STEPS is the turn-by-turn list (tap a step to jump there). SAVED keeps favourites (/data/maps/favorites.txt). MENU also has ONLINE/OFFLINE mode, units, route start, "what is at the pin?" and clear.
ME / live location: found from your IP address, so it is city-level only (often a few km off). To be exact, move the map onto your real position and use MENU → Set my location = map centre; it is remembered.
Offline: set MENU → Mode = OFFLINE and it only uses tiles already on the SD card. Pre-download an area on your PC with python3 tools/maps_fetch.py --place "Your City" --radius-km 8 --zoom 10-15 --contact you@example.com --out <SD>/maps/tiles (--style light|osm, and --dry-run first). Please keep downloads small and put your own contact in ua= in /sys/maps.txt — the tile servers are free community services.
Settings live in /sys/maps.txt (style, tile server for style=custom, route servers, units, mode). Map data © OpenStreetMap contributors; Voyager/Light © CARTO.
7c. FrogFind! — a web browser held like a phone (app 91)

FrogFind is "the search engine for vintage computers": it fetches DuckDuckGo results and turns any web page into plain, simple HTML. That is exactly what a 320 KB microcontroller can show, so KRIVI uses it as its browser.

Hold the board upright. The app runs in portrait (240×320). If it's upside‑down for you, tap Flip on its home screen (remembered).
Search or open a site: tap the search box or the address bar. A full phone‑style keyboard opens: a–z, Shift (tap twice for Caps Lock), 123 for numbers and punctuation, #+= for the remaining symbols, space, delete (hold to keep deleting) and a big Enter. A pop‑up shows the key under your finger — slide to another key before lifting to change it. Words are searched; things like wikipedia.org open as a site.
Read: tap blue links to follow them, drag up/down to scroll, or use Up/Down. Back, Home and Reload are on the bottom bar; the red X exits.
Pictures: an article's "View page images: [1] [2]" links open in a JPEG viewer (tap to switch between fit and 1:1).
Long pages are split into parts — tap the button at the bottom for the next part.
Needs WiFi. Pages download over plain HTTP to /sys/cache on the SD card (without a card, smaller pages still work from RAM). Recent searches are kept in settings.json under frogfind.
The tools/frogfind_test/ folder holds the PC tests for the browser's parser, link handling, keyboard and touch mapping (1,390 checks) and draws preview screenshots with the real TFT_eSPI fonts.
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
Libraries (versions this build was tested with — install these exact versions with Tools → Manage Libraries):
Library	Version	Note
TFT_eSPI (Bodmer)	2.5.43	then copy TFT_eSPI_User_Setup.h over Arduino/libraries/TFT_eSPI/User_Setup.h
TJpg_Decoder (Bodmer)	1.1.0	
ArduinoJson (Benoit Blanchon)	7.2.1	
ESP32‑A2DP (Phil Schatzmann)	1.8.10	⚠️ 1.8.11 and newer need ESP32 core 3.x and fail with ESP_A2D_AUDIO_STATE_SUSPEND was not declared
NimBLE‑Arduino (h2zero)	1.4.1	the BLE apps use the 1.4 API, not 2.x
The Helix MP3 decoder is bundled in‑tree under KriviOS/src/helix/.

Arduino IDE, step by step

File → Preferences → Additional boards manager URLs: https://espressif.github.io/arduino-esp32/package_esp32_index.json
Tools → Board → Boards Manager → install esp32 by Espressif Systems, version 2.0.17.
Install the libraries above (exact versions), then copy TFT_eSPI_User_Setup.h → Arduino/libraries/TFT_eSPI/User_Setup.h (replace it).
Open KriviOS/KriviOS.ino (keep the whole KriviOS folder together — every .cpp/.h in it is part of the sketch).
Tools: Board ESP32 Dev Module, Partition Scheme Huge APP (3MB No OTA/1MB SPIFFS), Flash Mode QIO, Flash Size 4MB, CPU 240MHz, Upload Speed 921600 (or 460800).
Pick the port and click Upload (hold BOOT if it won't connect).

A build is ~77 % of the 3 MB app slot, 30 % static RAM.

Partition table: the sketch folder contains partitions.csv, and the Arduino IDE uses it automatically (it overrides the Partition Scheme menu). Its app0 slot is exactly 3 MB — the same limit the IDE checks with Huge APP — so a too-big sketch is reported as "Sketch too big" rather than producing an image the board can't start. Before shipping a build, python3 tools/check_image.py <KriviOS.ino.bin> [merged.bin] double-checks that the image fits its slot.

11. Troubleshooting
Blank/garbled screen → try the _ST7789 image.
"NO SD CARD" / missing files → card isn't FAT32, isn't seated, or the folders aren't at the card root.
Locked out → the password is in /sys/settings.json under "security":{"pass":".."}; clear it there to remove the lock.
Internet Radio won't play → MP3 (Icecast/SHOUTcast) streams only, not AAC/OGG; check WiFi and the URL in /sys/radio.txt.
OTA says "no OTA partition" → expected on a 4 MB CYD; update over USB/SD.
FrogFind says "No WiFi" / "Couldn't load the page" → join a network in Settings → WI‑FI NETWORKS; school networks that need a web sign‑in page can't be used (the board will say so after joining).
FrogFind screen is upside‑down → tap Flip on the FrogFind home screen.
Red LED on, screen dark, nothing starts after flashing → the program didn't fit its partition (that was the v1.9.7 bug, fixed in 1.9.8). Flash 1.9.8, keep the partitions.csv that comes with the sketch, and select Huge APP. If an SD card is in, also try booting without it.
Compile error ESP_A2D_AUDIO_STATE_SUSPEND → install ESP32‑A2DP 1.8.10 (see §10).
Flashing won't start → use a data cable, a lower --baud (e.g. 460800), and hold BOOT.
12. Version highlights
1.9.8 — Fixes v1.9.7 not starting (red LED, dark screen): the OS had grown 16 bytes past its 2.37 MB flash slot. The slot is now 3 MB (720 KB free), matching the IDE's size check; new tools/check_image.py size guard.
1.9.7 — New FrogFind! portrait web browser with a phone‑style keyboard (a–z, Shift/Caps, 123, #+=, delete, Enter, key pop‑ups); Wi‑Fi network list in Settings; Wi‑Fi reconnect fix for open/school networks; PC test suite for the browser.
1.9.4 — Trading (candlesticks + watchlist) and News (real photos) apps.
1.9.0–1.9.3 — AI Assistant (Gemini), icons for every app, Maps tile fixes.
1.8.1 — New Maps app (Google-style map via CARTO Voyager: search, car/bike/walk routes with turn list, favourites, IP location, online + offline tiles, selectable map style).
1.8.0 — Media & Sensors page (Bluetooth + MP3, Internet Radio, Light Meter, Auto‑Brightness, Night Light); removed the old audio/decibel apps; Arcade expanded to 16 games (added 2048, Connect Four, Checkers, Reversi, Simon, Sudoku, Solitaire, Space Invaders; removed Ludo & Snakes & Ladders); Falling Sand → Sandbox with 33 materials and a tabbed picker; redrawn Virtual Pet; WiFi open + school/enterprise support; device password lock; OTA Update app.
1.7.x — Arcade games hub; SD‑card RAM helper (video‑from‑SD, SwapBuf); duplicate‑tile cleanup. 
1.6.2 — major RAM pass: strip rendering, trimmed decoder/media buffers.

KRIVI OS runs entirely on the device — no companion app. Everything above is on the 320×240 touchscreen in your hand.
