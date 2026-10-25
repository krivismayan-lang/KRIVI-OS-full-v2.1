KRIVI OS

A touch‑driven micro‑operating‑system for the ESP32‑2432S028 "Cheap Yellow Display" (CYD) — a tiny 320×240 touchscreen computer that boots straight into a desktop launcher and runs 50+ self‑contained apps: games and emulators, audio tools, utilities, radio/wireless labs, graphics demos, and system tools.

Current version: 1.7.2-arcade

1. What you need (materials)

Required

Item	Notes
ESP32‑2432S028 "CYD" board	The "Cheap Yellow Display." Get the ILI9341 panel version (most common). A few boards ship with an ST7789 panel — there's a separate _ST7789 image for those.
microSD card — more than 8 GB	16 GB is ideal. Must be formatted FAT32 (cards over 32 GB are often exFAT out of the box — reformat them to FAT32). This holds games, music, movies, icons and settings.
USB data cable	Matches your board's port (usually micro‑USB or USB‑C). Must be a data cable, not charge‑only — you flash and power the board over it.
A computer	Windows, macOS or Linux, to flash the firmware (esptool or the Arduino IDE).

Optional (only for certain apps)

Item	Enables
Wired/USB headphones or a powered speaker	n/a — audio is the on‑board speaker; no extra parts needed
A WiFi network (2.4 GHz)	Weather, World Clock/NTP, ISS Globe, the wireless‑lab apps
Raspberry Pi + USB cable	The "Pi Link" apps (monitor, console, trackpad, shutdown)
HC‑SR04 ultrasonic sensor	The Ultrasonic Range app
A second ESP32 running KRIVI	ESP‑NOW Radio peer messaging

The CYD has a built‑in screen, touch, SD slot and RGB LED — so for the core experience the board + a FAT32 SD card with at least 8GB or more + a USB cable is everything you need.

2. Quick start — set it up in order

Do these in this order: prepare the card, flash the firmware, insert the card, power on.

Step 1 — Format the SD card (FAT32)

Format your microSD card as FAT32. (On Windows use the built‑in formatter for ≤32 GB cards, or a tool like "guiformat"/"SD Card Formatter" for larger ones; on macOS use Disk Utility → MS‑DOS (FAT); on Linux mkfs.vfat -F 32.)

Step 2 — Put the KRIVI files on the card

Copy the contents of the project's sd_card/ folder onto the root of the card, so you end up with this layout:

/roms/            NES (.nes) and CHIP-8 (.ch8) ROMs      (add your own)
/music/           your .mp3 files                         (add your own)
/data/movies/     converted .avi (MP4 → AVI) clips        (add your own)
/data/video/      raw .mjpeg clips                         (add your own)
/icons/           launcher + weather icons                (from sd_card/)
/sys/             settings + app manifests                (from sd_card/)

At minimum copy /icons/ and /sys/ from sd_card/. The /roms, /music, /data folders can start empty — add your own games, songs and videos later. Use tools/convert_media.py to turn an MP4 into a KRIVI‑playable .avi.

Step 3 — Flash the firmware

Connect the board to your computer with the USB cable, then flash the pre‑built image from the project's prebuilt/ folder to offset 0x0:

bash
esptool.py --chip esp32 --baud 921600 write_flash 0x0 prebuilt/KRIVI_OS_v1.7.2-arcade_ILI9341.bin
No esptool? Install it with pip install esptool.
On Windows the port is like COM5; on macOS/Linux like /dev/ttyUSB0 — add --port <that> if esptool doesn't auto‑detect.
If flashing fails to start, hold the BOOT button while it connects.
ST7789‑panel board? Use the _ST7789 image instead.
Step 4 — Insert the card and power on

Put the microSD card into the board's slot, then power the board (USB is fine). KRIVI boots through its splash sequence and lands on the desktop launcher.

Step 5 — First‑run settings

Go to page 6 → System Settings and set:

Brightness and Volume (drag the bars)
WiFi (tap SET WIFI, enter your 2.4 GHz network — enables Weather, clock sync, ISS, etc.)
Tap SAVE (writes /sys/settings.json to the card) — then SYNC NTP to set the clock.

You're done. Tap any tile to launch an app; tap EXIT (top‑right) to come back.

3. What it runs on
	
Board	ESP32‑2432S028 ("CYD" — Cheap Yellow Display)
CPU	ESP32 (dual‑core Xtensa @ 240 MHz), no PSRAM
Flash	4 MB
RAM	~320 KB SRAM (this is the constraint everything is designed around)
Display	2.8" ILI9341, 320×240, landscape, locked
Touch	XPT2046 resistive (bit‑banged soft‑SPI)
Audio out	Mono speaker on GPIO26 (built‑in DAC via I²S)
Mic	Analog mic on GPIO35
Storage	microSD on the VSPI bus
Extras	RGB status LED (active‑low), light relay, NTP clock over WiFi

There is no PSRAM, so usable working memory is small. A large part of KRIVI OS's design is keeping each app inside that budget — see §7 Memory.

4. Booting & the launcher

On power‑up KRIVI runs a short boot sequence (hardware init, SD mount, settings load, optional NTP time sync) and lands on the desktop launcher.

The launcher is a 4×3 grid of app tiles across 6 pages.
Swipe left/right (or tap the page arrows) to change pages.
Tap a tile to launch its app. A splash screen shows the app's heap budget before it starts.
Every app has an EXIT button in the top‑right corner. Tapping it returns you to the launcher.

Only one app runs at a time. When you exit, the app's function returns and the launcher reclaims everything it used — audio is stopped, radios are turned off, and all its memory is freed. Nothing keeps running in the background.

The six pages:

Gaming & Emulation
Audio & Sensors
Utilities & Time
Pi Link & Radio
Graphics & Sandbox
Wireless Lab & System
5. The apps
Page 1 — Gaming & Emulation
App	What it does
NES Emulator	Plays .nes ROMs from /roms (mappers NROM/MMC1/UxROM/CNROM/MMC3). The whole ROM loads into RAM, so with no PSRAM only games up to ~180 KB fit (SMB3 is too big).
Arcade	A single menu of 10 classic games — see §6.
CHIP‑8 Virtual Machine	Runs classic CHIP‑8 .ch8 programs from /roms.
Page 2 — Audio & Sensors
App	What it does
Piano Synth	Two‑octave touch piano, square‑wave tones.
Audio Oscilloscope	Live waveform of the mic input.
VU / Decibel Meter	Sound‑level meter.
Drum Pad	8 synthesized percussion sounds.
MP3 Music Player	Plays .mp3 files through the on‑board DAC (in‑tree Helix decoder), with ID3 tags and a visualizer.
FFT Spectrum Analyzer	Real‑time frequency spectrum of the mic.
Morse Code Keyer	Tap out and decode Morse.
Voice Pitch Tracker	Detects the pitch/note you sing or hum.
Ultrasonic Range	Distance readout from an HC‑SR04‑style sensor.
Page 3 — Utilities & Time
App	What it does
Stopwatch	Lap stopwatch.
Pomodoro Timer	Work/break focus timer.
Calculator	Touch calculator.
Calendar	Month calendar.
Text Editor	On‑screen‑keyboard text editor (reads/writes SD files).
Hex Viewer	Browse any file as hex/ASCII.
Alarm Clock	Alarm that rings anywhere in the OS.
World Clock	Multiple time zones.
Currency Converter	Converts between currencies.
Unit Converter	Length, weight, temperature, etc.
Page 4 — Pi Link & Radio
App	What it does
Pi Performance Monitor	Live stats from a Raspberry Pi over USB serial (needs krivi_bridge.py).
Pi Bash Console	Runs whitelisted scripts on the Pi and streams output back.
Pi Remote Trackpad	Use the touchscreen as a trackpad for the Pi.
Pi Shell Keyboard	On‑screen keyboard that types into the Pi shell.
Pi Safe Shutdown	Cleanly powers the Pi down.
Bluetooth Macro Pad / BLE Keyboard	Act as a Bluetooth‑LE (NimBLE) HID device for a PC/phone.
WiFi Analyzer	Scans and charts nearby WiFi networks.
Ping Monitor	Pings a host and graphs latency.
ESP‑NOW Radio	Peer‑to‑peer messaging to another ESP32 over ESP‑NOW.
Page 5 — Graphics & Sandbox
App	What it does
Weather	Current conditions + forecast over WiFi (icons loaded from SD).
Sketchpad	Finger‑paint canvas.
3D Rainbow Warp	Rotating 3D graphics demo.
Falling Sand	Cellular‑automaton sand/physics sandbox.
Mandelbrot	Pinch/tap to zoom the Mandelbrot set.
Video & MP4 Player	Plays converted .avi (MJPEG) and raw .mjpeg clips from SD, with audio. Decodes straight from the SD card when RAM is tight (see §7).
Virtual Pet	A little on‑screen pet to care for.
Touch Diagnostics	Touch calibration / test screen.
System Manual	On‑device help.
Demoscene FX	Plasma / fire effects.
Matrix Digital Rain	The green "Matrix" rain.
Starfield Screensaver	Flying‑through‑stars screensaver.
Page 6 — Wireless Lab & System
App	What it does
Channel Congestion	WiFi channel‑usage overview.
BLE Scanner	Lists nearby Bluetooth‑LE devices.
Signal Mapper	Track signal strength as you move.
LAN Discovery	Finds devices on your network.
Net Health	Connectivity/latency checks.
3D ISS Globe Radar	Live International Space Station position on a rotating globe (wheretheiss.at API).
System Settings	Brightness, volume, WiFi, clock, save/reboot.
NVS Flash Editor	Inspect/edit stored key‑value settings (advanced).
Desk Clock	Full‑screen clock.
Noise Floor Log	Logs ambient noise over time.
6. The Arcade

The Arcade (on the Gaming page) is one scrollable menu that holds 10 classic games. Tap one to play, tap EXIT to return to the list. The games that live here are not duplicated as separate tiles on the home grid.

Flappy Bird — tap to flap through the pipes
Snake — eat, grow, don't bite yourself
Tetris — stack and clear lines
Pong — classic paddle bat‑and‑ball
Breakout — bounce the ball, break bricks
Minesweeper — flag the mines, clear the grid
Chess — full two‑player chess (also Tic‑Tac‑Toe)
3D Dice — roll two tumbling 3D dice
Ludo — 2–4 player pass‑and‑play, race your four tokens home
Snakes & Ladders — 2–4 player pass‑and‑play, climb and slide on a 1–100 board

Ludo and Snakes & Ladders are new and draw directly to the screen (no frame buffer), so they use almost no RAM.

7. SD card layout (reference)
/roms/            NES (.nes) and CHIP-8 (.ch8) ROMs
/music/           .mp3 files for the MP3 player
/data/movies/     converted .avi (MP4 → AVI/MJPEG) clips
/data/video/      raw .mjpeg clips
/icons/           launcher app icons (app_NN.raw)
/icons/weather/   weather condition icons
/sys/
   settings.json  all persistent settings (brightness, volume, WiFi, clock…)
   apps/app_NN.json   optional per-app parameter manifests
   swap.bin       temporary SD-as-RAM scratch file (auto-managed)

Media is prepared with the tools/convert_media.py helper in the project. Settings are written automatically from System Settings → Save.

8. Memory — how KRIVI stays inside a tiny RAM budget

The ESP32 here has ~320 KB of SRAM and no PSRAM, so the challenge is never total storage (flash is plentiful, the SD card huge) — it's fitting an app's working buffers into the small pool of RAM, specifically the largest single contiguous block available at the moment.

KRIVI handles this several ways:

One app at a time, fully reclaimed on exit. App code lives in flash (memory‑mapped) and costs zero RAM until you open it. When you exit, the app returns and the launcher frees its memory, stops audio and turns radios off. There is no background app RAM to reclaim.
Strip rendering for big graphics. Apps that would need a full‑screen off‑screen buffer (Flappy Bird, 3D Dice, the ISS globe) render the screen in horizontal strips instead — a few KB at a time rather than 50–70 KB. Same picture, a fraction of the RAM.
SD card as a RAM helper. The Video player decodes each frame straight from the SD card when the heap can't spare a frame buffer — it keeps playing instead of failing with "Not enough RAM." A reusable SwapBuf facility lets apps park a large, block‑accessed buffer in a temp file on the card (/sys/swap.bin) and use almost no heap.
Graceful allocation. Large allocations retry once after a brief yield (bigAlloc), so a momentary fragmentation blip doesn't throw a false low‑RAM error; where it makes sense, buffers shrink rather than refuse.

The honest limit: the SD card can only help buffers that are read/written in blocks. Data that's touched per‑pixel every frame (a live game frame buffer, the Falling‑Sand grid, a NES ROM) must stay in RAM — the SD card is far too slow for that. For those, the fix is keeping the buffer small (strip rendering), not swapping to SD.

9. Building from source (Arduino IDE / arduino‑cli)

Most people just flash the pre‑built image (Step 3 above). To build it yourself:

Board core: ESP32 Arduino 2.0.17 (3.x removed the built‑in‑DAC I²S mode this project relies on).
Board / FQBN: espressif:esp32:esp32 with PartitionScheme=huge_app, FlashMode=qio, CPUFreq=240, FlashSize=4M.
Libraries required:
TFT_eSPI (display) — configured via TFT_eSPI_User_Setup.h
TJpg_Decoder (JPEG frames for the video player)
ArduinoJson
ESP32‑A2DP (classic‑Bluetooth audio support)
NimBLE‑Arduino 1.4.x — the BLE apps use the 1.4 API (not the newer 2.x)
The MP3 decoder (Helix) is bundled in‑tree under KriviOS/src/helix/ — no library needed.

Open KriviOS/KriviOS.ino, select the board/partition settings above, and compile. A current build is about 70% of flash and 29% static RAM, leaving ~230 KB of heap for apps.

10. Project layout
KRIVI_OS/
  KriviOS/              the Arduino sketch (all source)
    KriviOS.ino         entry point (setup/loop)
    krivi.h             shared types, API, version
    launcher.cpp        desktop launcher + the app registry
    loader.cpp          per-app launch / heap budgeting / reclaim
    hal_touch_audio.cpp touch + I2S/DAC audio engine
    sys_services.cpp    settings, clock/NTP, SD helpers, SwapBuf
    ui.cpp              the on-screen UI toolkit
    app_*.cpp           the apps (games, audio, utils, radio, graphics…)
    app_arcade.cpp      the Arcade hub + Ludo + Snakes & Ladders
    src/helix/          bundled Helix MP3 decoder
  prebuilt/             ready-to-flash .bin images
  sd_card/              files to copy onto the microSD card
  tools/                media converter, Pi bridge, etc.
  TFT_eSPI_User_Setup.h display pin configuration
11. Troubleshooting
Blank/garbled screen after flashing → your board may use the ST7789 panel; flash the _ST7789 image.
"NO SD CARD" or apps can't find files → card isn't FAT32, isn't seated, or the folders in Step 2 aren't at the card's root.
Weather / clock / ISS don't work → set WiFi in System Settings (2.4 GHz only) and SYNC NTP.
An app says "Not enough RAM" → it needs a bigger contiguous block than is free right now; reboot to defragment, or (for video) it will fall back to playing from SD automatically.
Flashing won't start → use a data USB cable, try a lower --baud (e.g. 460800), and hold BOOT while connecting.
12. Version highlights
1.7.2 — removed duplicate game tiles from the home grid (they live in Arcade now); fixed the clipped Arcade UP/DOWN buttons.
1.7.1 — SD‑card RAM helper: video decodes straight from SD when RAM is low; added the SwapBuf facility and retrying allocations.
1.7.0 — new Arcade games hub with 10 classics, including new Ludo and Snakes & Ladders (2–4 player).
1.6.2 — major RAM pass: strip rendering for Flappy/Dice, trimmed decoder/media buffers.

KRIVI OS runs entirely on the device — it needs no companion app. Everything above is on the 320×240 touchscreen in your hand.
