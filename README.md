# Desk Buddy – ESP32-S3 AI Voice Assistant Desk Robot

Desk Buddy is an open-source AI desk companion built on the ESP32-S3. It listens,
answers out loud using Groq (Whisper speech-to-text, LLM, Orpheus text-to-speech),
shows an animated OLED face, plays internet radio, and can describe what it sees
through its camera or broadcast a live video stream. A native Android app
(Kotlin + Jetpack Compose) handles Bluetooth LE control, Wi-Fi setup and the live camera view.

## Features
- Voice assistant with fast streaming speech playback
- Animated OLED face (idle, listening, thinking, talking, whistling)
- Camera vision on demand (OV3660) and live MJPEG broadcast to the app
- Internet radio stations with voice control
- Android app: chat, face preview, music, Wi-Fi and API key setup

## Repository layout
- `/firmware` – Arduino sketch for the ESP32-S3
- `/android-app` – Android Studio project

## Acknowledgments
Created with inspiration and support from **25MTE** — Mechatronics Engineering students at
Mehran University of Engineering & Technology (MUET), Jamshoro.
Proudly representing 25MTE · Class of 2025
