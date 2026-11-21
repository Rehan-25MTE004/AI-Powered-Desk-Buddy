# Desk Buddy – User Manual

**Desk Buddy** is an ESP32-S3 AI desk companion. Press its button, ask a question, and it answers out loud. It has an animated OLED face, plays internet radio, can describe what its camera sees, and can broadcast live video to an Android app.

This manual covers building it, flashing the firmware, and using it with the Android app (APK).

---

## Contents

1. [What you need](#1-what-you-need)
2. [Wiring](#2-wiring)
3. [Install the software and libraries](#3-install-the-software-and-libraries)
4. [Set up `secrets.h`](#4-set-up-secretsh)
5. [Board settings and upload](#5-board-settings-and-upload)
6. [First start](#6-first-start)
7. [Using Desk Buddy](#7-using-desk-buddy)
8. [The Android app (APK)](#8-the-android-app-apk)
9. [Live camera broadcast](#9-live-camera-broadcast)
10. [Troubleshooting](#10-troubleshooting)
11. [Privacy and security notes](#11-privacy-and-security-notes)
12. [Building the APK yourself (optional)](#12-building-the-apk-yourself-optional)
13. [Acknowledgments](#13-acknowledgments)

---

## 1. What you need

**Hardware**

| Part | Notes |
|---|---|
| Freenove ESP32-S3 WROOM **N16R8** board with camera connector | 16 MB flash, 8 MB PSRAM (PSRAM is required) |
| OV3660 camera module | Plugs into the board's camera connector |
| SH1106 128×64 OLED, I²C | Shows the face |
| I²S microphone, INMP441 | Voice input |
| I²S amplifier (e.g. MAX98357A) + small speaker (4–8 Ω) | Voice and radio output |
| Push button | One leg to a GPIO, the other to GND |
| USB-C cable and a good 5 V supply (2 A recommended) | A weak supply can cause reboots when the speaker plays |

**Accounts and devices**

- A **Groq API key** (free to create at <https://console.groq.com>). It is used for speech-to-text, the AI answers, camera vision and text-to-speech.
- A **2.4 GHz Wi-Fi network** (the ESP32 does not support 5 GHz).
- An **Android phone** (Android 7.0 or newer) for the app.
- A computer with the **Arduino IDE** (2.x recommended).

---

## 2. Wiring

All pin numbers are GPIO numbers and match the sketch.

| Part | Part pin | ESP32-S3 pin |
|---|---|---|
| **OLED (SH1106, I²C)** | SDA | GPIO 1 |
| | SCL | GPIO 3 |
| | VCC / GND | 3.3 V / GND |
| **Push button** | Leg 1 | GPIO 14 |
| | Leg 2 | GND (internal pull-up is used) |
| **Speaker amplifier (I²S)** | BCLK | GPIO 21 |
| | LRC (WS) | GPIO 40 |
| | DIN | GPIO 39 |
| | VIN / GND | 5 V (or 3.3 V) / GND |
| **Microphone (INMP441)** | SCK (BCLK) | GPIO 41 |
| | WS | GPIO 42 |
| | SD | GPIO 47 |
| | L/R | GND (left channel – required) |
| | VDD / GND | 3.3 V / GND |
| **Camera (OV3660)** | Freenove camera connector | No wiring needed |

The camera pins used by the sketch are for the Freenove ESP32-S3 WROOM camera connector: XCLK 15, SIOD 4, SIOC 5, D0–D7 = 11, 9, 8, 10, 12, 18, 17, 16, VSYNC 6, HREF 7, PCLK 13.

---

## 3. Install the software and libraries

### 3.1 Arduino IDE and ESP32 board package

1. Install the Arduino IDE from <https://www.arduino.cc/en/software>.
2. Open **File → Preferences** and add this to *Additional boards manager URLs*:
   `https://espressif.github.io/arduino-esp32/package_esp32_index.json`
3. Open **Tools → Board → Boards Manager**, search for **esp32** (by *Espressif Systems*) and install it. Tested with core **3.3.x**.

### 3.2 Libraries

Open **Sketch → Include Library → Manage Libraries** and install:

| Library | Author | Version needed |
|---|---|---|
| **ArduinoJson** | Benoit Blanchon | **7.x** (the sketch uses the v7 `JsonDocument` API) |
| **U8g2** | olikraus | latest |
| **NimBLE-Arduino** | h2zero | **2.x** (not 1.x) |
| **ESP8266Audio** | Earle F. Philhower, III | latest (works on ESP32 and provides the MP3 radio decoder) |

**Included with the ESP32 board package (nothing to install):** `WiFi`, `WiFiClientSecure`, `HTTPClient`, `Wire`, `Preferences`, `ESP_I2S`, `esp_camera`, `esp_http_server` and `mbedtls`.

> A message like *"Multiple libraries were found for SD.h"* is only a warning and can be ignored.

---

## 4. Set up `secrets.h`

`secrets.h` holds your private details. Put it in the same folder as `DeskBuddy.ino`. It needs exactly these three lines:

```cpp
#define WIFI_SSID      "YourWiFiName"
#define WIFI_PASSWORD  "YourWiFiPassword"
#define GROQ_API_KEY   "gsk_your_key_here"
```

- **Never upload your real `secrets.h` to GitHub.** Keep it in `.gitignore` and share only a version with blank values.
- You can also leave Wi-Fi and the key as placeholders and set them later from the Android app (Setup tab). Whatever you save from the app is stored on the device and **takes priority** over `secrets.h`.

---

## 5. Board settings and upload

In the Arduino IDE open the **Tools** menu and set:

| Setting | Value |
|---|---|
| Board | **ESP32S3 Dev Module** |
| PSRAM | **OPI PSRAM** (required) |
| Flash Size | **16MB (128Mb)** |
| Partition Scheme | **Huge APP (3MB No OTA/1MB SPIFFS)** (the default is too small) |
| USB CDC On Boot | **Enabled** if you upload through the board's native USB port |
| Upload Speed | 921600 |

Then:

1. Connect the board with USB.
2. Choose its port under **Tools → Port**.
3. Open `DeskBuddy.ino` and click **Upload**.
4. Open **Tools → Serial Monitor** and set the speed to **115200** to see messages.

> To wipe settings saved from the app (Wi-Fi, key, volume), set **Tools → Erase All Flash Before Sketch Upload** to *Enabled* for one upload, then turn it off again.

---

## 6. First start

1. Power on. The OLED shows the face.
2. Desk Buddy connects to Wi-Fi. If it works, it says **"How Can I Be Of Service Today?"**
3. The Serial Monitor prints a short list of commands.
4. The camera is **off** at start. It only turns on when you ask a camera question or start the broadcast from the app.

---

## 7. Using Desk Buddy

### 7.1 The button

| Action | What happens |
|---|---|
| **Click** | Starts listening (face shows "listening") |
| **Click again** | Sends what you said (recording stops automatically after 15 seconds) |
| Click while it is **thinking** | Cancels the request |
| Click while it is **talking** | Stops the speech |

### 7.2 What you can say

| Say | What happens |
|---|---|
| Any question ("How far is the Moon?") | Spoken answer |
| "What do you see?", "What is this?", "Take a picture", "Read this", "What color is this?" | Takes a photo and describes it |
| "Volume 7", "Louder", "Quieter", "Mute", "Max volume" | Changes volume (0–10, remembered) |
| "Play jazz" / "chill" / "80s" / "pop" / "rock" / "space" / "bass" | Starts that radio station |
| "Play some music" | Plays the last (or first) station |
| "Next station" | Switches station |
| "Stop the music" | Stops the radio |

Notes:

- Radio pauses while you talk and stops while Desk Buddy answers.
- Phrases such as "what is in…" or "what's this…" are treated as camera questions.
- Speech recognition is set to **English**.

### 7.3 Serial Monitor commands (optional)

Type these in the Serial Monitor (115200 baud) and press Enter:

| Type | What happens |
|---|---|
| Any text | Asks the AI as if you had spoken it |
| `L` | Starts listening |
| `A` | Sends the recording |
| `volume 7`, `mute`, `play jazz`, `stop the music` | Same as the voice commands |

The monitor also prints `[time]` lines showing how long speech-to-text, the AI answer and the speech took, which helps find slow steps.

### 7.4 The face

The OLED face shows idle (blinking and looking around), listening, thinking (spinning rings), talking and whistling (while the radio plays).

---

## 8. The Android app (APK)

### 8.1 Install

1. Download **`DeskBuddy.apk`** from this repository's **Releases** page onto your phone.
2. Open the file. If Android asks, allow **Install unknown apps** for your browser or file manager.
3. Tap **Install**, then **Open**.

### 8.2 Connect

1. Make sure Desk Buddy is powered on and your phone's **Bluetooth is on**.
2. Open the app and tap **Connect**.
3. Allow the permission:
   - Android 12 or newer: **Nearby devices**
   - Android 11 or older: **Location** (needed by Android for Bluetooth scanning)
4. The app finds the device named **DeskBuddy-XXXX** and connects. The Bluetooth chip turns green and the face preview follows the real face.

Only one phone can be connected at a time.

### 8.3 Tabs

| Tab | What it does |
|---|---|
| **Talk** | Type a question and get a spoken answer. Turn "Speak the answer out loud" on or off. "Make it say this" reads your text aloud. Voice questions you ask by button also appear here. Face preview buttons try each expression. |
| **Camera** | Start or stop the live camera broadcast (see section 9). |
| **Music** | Pick a radio station, stop the music, and set the volume with the slider. |
| **Setup** | Scan for Wi-Fi networks, tap one, enter the password and connect. Save or replace the Groq API key. Turn Desk Buddy off. |
| **About** | Information about the project and acknowledgments (at the bottom of Setup). |

### 8.4 Setting Wi-Fi from the app

1. Open **Setup** and tap **Scan networks**.
2. Tap your network, type the password and tap **Connect Desk Buddy to Wi-Fi**.
3. A message confirms success. The network is saved and used on the next start.

### 8.5 Setting the API key from the app

1. Open **Setup**, paste your key (starts with `gsk_`) and tap **Save key**.
2. The app reports whether the key was verified. The device only shows the last four characters afterwards.

### 8.6 Turning it off

**Setup → Turn Desk Buddy off** puts it into deep sleep. Press the button on Desk Buddy to wake it.

---

## 9. Live camera broadcast

1. Connect the app, make sure the **Wi-Fi chip** shows Desk Buddy is on Wi-Fi, and put your phone on **the same Wi-Fi network**.
2. Open the **Camera** tab and tap **Start broadcast**. The camera powers up and the live picture appears.
3. Tap **Full screen** to open the stream in your browser, or **Stop broadcast** to switch the camera off completely.

Details:

- The stream address is `http://<Desk Buddy IP>:81/stream`. It works in any browser on the same network while the broadcast is on.
- The picture **freezes while Desk Buddy is listening or answering**, so it can respond quickly. It resumes afterwards.
- Without the broadcast, the camera only turns on for camera questions and switches itself off after about 60 seconds of not being used.
- If the picture says it can't connect, check that the phone is on the same Wi-Fi (not a guest network) and tap **Retry**.

---

## 10. Troubleshooting

| Problem | What to try |
|---|---|
| Serial says *"No PSRAM"* or *"PSRAM allocation failed"* | Set **Tools → PSRAM → OPI PSRAM** and make sure the board is an N16R8 version. |
| *"Sketch too big"* / *"text section exceeds available space"* | Set **Partition Scheme** to **Huge APP**. |
| Compile errors about `JsonDocument` | Install **ArduinoJson 7.x**. |
| Compile errors about NimBLE | Install **NimBLE-Arduino 2.x**. |
| Compile error: `secrets.h: No such file` | Create `secrets.h` next to the sketch (section 4). |
| Serial Monitor is empty | Check the port, set baud to 115200, and try **USB CDC On Boot: Enabled** (or Disabled if you use the UART port). |
| *"Wi-Fi connection failed"* | Use a 2.4 GHz network, re-check the name and password, or set Wi-Fi in the app. |
| Changing Wi-Fi in `secrets.h` has no effect | Settings saved from the app win. Use the app, or upload once with **Erase All Flash** enabled. |
| *"Groq HTTP 401"* | The API key is wrong or missing. Save a new one in the app. |
| Speech fails with an HTTP 4xx message | The voice model may need its terms accepted in the Groq console, or the model name changed. The models are set near the top of the sketch. |
| No sound | Say "volume 7"; check the amplifier wiring and speaker. |
| It doesn't hear me | Check the microphone wiring, make sure **L/R is connected to GND**, and speak close to it. |
| Camera pictures are dark or blurry | Turn the lens barrel to focus (close objects need a close focus), use more light, and make sure the ribbon cable is seated. |
| App can't find Desk Buddy | Bluetooth on, permission allowed, phone close, Desk Buddy powered, and no other phone connected to it. |
| Live picture won't load | Same Wi-Fi as Desk Buddy, no guest network or router "client isolation", and the Wi-Fi chip shows it's connected. |
| Reboots when it speaks | Use a better USB cable and a 5 V / 2 A supply. Copy the lines printed before `Backtrace:` if it continues. |
| Answers feel slow | Read the `[time]` lines in the Serial Monitor to see which step is slow. |

---

## 11. Privacy and security notes

- **Voice and photos go to Groq** for processing. Photos are only sent when you ask a camera question.
- **The live broadcast is not encrypted or password-protected.** Anyone on your Wi-Fi who knows the address could view it while it is on. Use it on a network you trust and stop the broadcast when finished.
- **Bluetooth control has no pairing or password.** Anyone close enough could connect and change settings. Keep the project to a trusted environment.
- **Keep your API key private.** If it is ever uploaded or shared, delete it in the Groq console and create a new one.

---

## 12. Building the APK yourself (optional)

1. Open the `android-app` folder in **Android Studio** (File → Open, choose the folder containing `settings.gradle.kts`).
2. Wait for Gradle sync to finish. If offered an Android Gradle Plugin or Gradle upgrade, choose **Not now**.
3. Choose **Build → Build Bundle(s) / APK(s) → Build APK(s)**.
4. Find `app-debug.apk` under `app/build/outputs/apk/debug/` and copy it to your phone.

The app is native Kotlin using Jetpack Compose and Bluetooth LE.

---

## 13. Acknowledgments

Created with inspiration and support from **25MTE** — Mechatronics Engineering students at Mehran University of Engineering & Technology (MUET), Jamshoro.

Proudly representing **25MTE** · Class of 2025
