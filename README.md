# 🚨 GPS-Based Smart Explosive Detection Unit with Admin Portal Integration

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Made With](https://img.shields.io/badge/Made%20with-ESP32-orange)](https://github.com/ManishKDtm)
[![Platform](https://img.shields.io/badge/Platform-IoT-blueviolet)](#)
[![Status](https://img.shields.io/badge/Status-Prototype-green)](#)

A GPS-enabled smart explosive detection system integrated with a centralized **Admin Portal** for real-time threat detection, live tracking, and instant alerts. Built for enhanced security in high-risk areas such as borders, transportation hubs, and VVIP events.

---

## 🚀 Features

<details>
<summary><strong>Hardware Unit 🎛️</strong></summary>

- 💣 Explosive detection via chemical/gas sensors (MQ2/MQ9)
- 📍 Real-time GPS location tracking
- 📡 Wireless communication using GSM / Wi-Fi / LoRa
- ⚡ Microcontroller-based (ESP32 / Arduino)
- 🔋 Portable & battery-powered
</details>

<details>
<summary><strong>Admin Portal 🌐</strong></summary>

- 📌 Live map tracking with coordinates
- 📢 Real-time alerts on explosive detection
- 🔒 Admin login & secure dashboard
- 🧠 History logs & data visualization
</details>

---

## 🧠 How It Works

```mermaid
graph TD;
    A[Sensor Detects Explosive] --> B[GPS Fetches Location];
    B --> C[Data Sent to Cloud/Server];
    C --> D[Admin Portal Displays Alert];
    D --> E[Live Map Shows Device Location];
    E --> F[Data is Logged for Analysis];
