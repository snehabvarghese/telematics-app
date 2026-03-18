# 🛵 VAJRA Smart Telematics System

> **Real-time IoT vehicle telematics platform** — live GPS tracking, remote immobilization, geofencing, and analytics for electric scooters. Built with MQTT over TLS, React, and React Native (Expo).

---

## 📸 Overview

VAJRA is a full-stack telematics solution consisting of:

- **VAJRA Web Dashboard** — a React + Vite progressive web app for fleet monitoring
- **VAJRA Mobile App** — a cross-platform Expo/React Native app for on-the-go vehicle control (Android & iOS)

Both clients subscribe to live telemetry packets from an ESP32-based GPS tracker over **MQTT (HiveMQ Cloud, TLS 1.2)** and enable real-time remote control of the vehicle.

---

## ✨ Features

### 📡 Real-Time Telemetry
- Live MQTT data stream over MQTTS (WSS:8884) with QoS 1
- Incoming binary packet parsing (GPS, ignition, voltage, signal strength, frame counter)
- UTC timestamp, HDOP/PDOP accuracy metrics
- Cellular operator identification (BSNL, Airtel, JIO, Vodafone)

### 🗺️ GPS & Live Map
- Leaflet.js map embedded in both web and mobile (WebView)
- Real-time scooter location with animated marker
- Scrollable packet history with speed & timestamp log

### 🔒 Remote Immobilizer Control
- Toggle Digital Output 1 (DO1) to cut/restore ignition signal
- Confirmation modal before activation/deactivation
- Visual feedback with live ignition state (DI1 HIGH/LOW)
- MQTT command publish to hardware device

### 📍 Polygonal Geofencing
- Draw custom polygon zones on a Leaflet map (tap to add vertices)
- Point-in-polygon detection (ray-casting algorithm) running live
- **Auto-immobilize** on zone exit — vehicle is stopped automatically
- Multiple zones supported with per-zone breach status display

### 🔋 Battery Analytics
- Live battery charge % from analog input (AI1, 0–5V mapped)
- Voltage trend line chart with min/avg/max stats
- Battery health score, temperature display, and warranty info
- Color-coded alerts: green (>50%), amber (20–50%), red (<20%)

### 📊 Data Optimizer
- Three transmission rate modes: **Saver** (60s), **Auto** (10s), **Turbo** (1s)
- Publishes optimizer mode (LOW/MID/HIGH) to device via MQTT

### 📦 Packet Viewer
- Decoded raw packet fields displayed in a clean table
- Hex/binary view of incoming frames
- Frame counter, fix status, and all analog/digital I/O values

### 📶 Network Info
- RSSI (dBm), signal bars, raw strength out of 31
- IMEI, MCC, MNC, operator name, packet status (live vs history)
- MQTT broker connection diagnostics

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Hardware Layer                        │
│   ESP32 / GPS Module → Parses NMEA → Builds Packets     │
│   Publishes over MQTT (TLS) to HiveMQ Cloud Broker      │
└───────────────────────┬─────────────────────────────────┘
                        │ MQTTS (WSS:8884)
          ┌─────────────┴─────────────┐
          ▼                           ▼
┌─────────────────┐       ┌────────────────────┐
│  VAJRA Web App  │       │  VAJRA Mobile App  │
│  React + Vite   │       │  Expo + RN (MQTT)  │
│  (Browser PWA)  │       │  Android / iOS     │
└─────────────────┘       └────────────────────┘
     Dashboard                  Dashboard
     Packet Viewer              Analytics
     Map View                   Live Map
     Device Control             Device Control
     Analytics                  Geofencing
     Network Info               Packet Log
```

---

## 🛠️ Tech Stack

| Layer            | Technology                                   |
|------------------|----------------------------------------------|
| Web Frontend     | React 19, Vite 7, React Router, Recharts     |
| Mobile App       | Expo 54, React Native 0.81, Expo Router      |
| Maps             | Leaflet.js (web), WebView + Leaflet (mobile) |
| Messaging        | MQTT.js v5 (web), MQTT.js v4 (mobile)       |
| Charts           | Recharts (web), react-native-chart-kit       |
| Icons            | Lucide React / Lucide React Native           |
| Animations       | Framer Motion (web), React Native Animated   |
| MQTT Broker      | HiveMQ Cloud (Free Tier, TLS 1.2)           |

---

## 🚀 Getting Started

### Prerequisites

- Node.js ≥ 18
- npm or yarn
- Expo CLI (for mobile): `npm install -g expo-cli`
- A HiveMQ Cloud account (or any MQTT broker supporting WSS)

---

### Web Dashboard

```bash
# Install dependencies
cd /path/to/Chong
npm install

# Start dev server
npm run dev
```

The app will be available at `http://localhost:5173`.

**Build for production:**
```bash
npm run build
```

---

### Mobile App (Expo)

```bash
cd VAJRA-mobile
npm install

# Start Expo dev server
npm start

# Run on Android
npm run android

# Run on iOS
npm run ios

# Run on LAN (for physical device)
npm run start:lan
```

Scan the QR code with **Expo Go** on your device.

---

## 📁 Project Structure

```
Chong/
├── src/                        # Web dashboard source
│   ├── App.jsx                 # Root with MQTT context + router
│   ├── pages/
│   │   ├── Dashboard.jsx       # Main vehicle status page
│   │   ├── Analytics.jsx       # Voltage/charge charts
│   │   ├── DeviceControl.jsx   # Immobilizer + geofencing (web)
│   │   ├── MapView.jsx         # Full-screen Leaflet map
│   │   ├── PacketViewer.jsx    # Raw packet field inspector
│   │   └── Network.jsx         # Cellular & MQTT diagnostics
│   └── utils/
│       ├── mqttClient.js       # MQTT connection & subscriptions
│       └── packetParser.js     # Binary packet decoder
│
└── VAJRA-mobile/               # Expo mobile app
    ├── app/
    │   ├── _layout.jsx         # Root layout + MQTT context provider
    │   ├── auth.jsx            # Login screen
    │   └── (tabs)/
    │       ├── index.jsx       # Dashboard (hero card, map, battery)
    │       ├── analytics.jsx   # Battery charge trend & packet log
    │       ├── control.jsx     # Immobilizer + geofencing
    │       ├── map.jsx         # Full-screen live map
    │       ├── packet.jsx      # Packet viewer
    │       └── _layout.jsx     # Tab navigator config
    └── src/
        └── utils/
            └── mqttClient.js   # Mobile MQTT publish helpers
```

---

## ⚙️ Configuration

### MQTT Broker

Update the broker config in `src/utils/mqttClient.js` (web) and `VAJRA-mobile/src/utils/mqttClient.js` (mobile):

```js
const BROKER_URL = 'wss://your-broker.hivemq.cloud:8884/mqtt';
const OPTIONS = {
  username: 'your-username',
  password: 'your-password',
  clientId: `vajra_${Math.random().toString(16).slice(2)}`,
};
```

### MQTT Topics

| Topic                    | Direction | Description                      |
|--------------------------|-----------|----------------------------------|
| `vajra/telemetry`        | Subscribe | Incoming vehicle telemetry packets |
| `vajra/command/immob`    | Publish   | Immobilizer ON/OFF commands       |
| `vajra/command/frequency`| Publish   | Data rate control (1/10/60 s)    |
| `vajra/command/optimizer`| Publish   | Mode: LOW / MID / HIGH           |

---

## 📱 Mobile App Screens

| Screen          | Description                                              |
|-----------------|----------------------------------------------------------|
| **Dashboard**   | Hero scooter card, live map tile, battery widget, ignition/immob status |
| **Analytics**   | Battery % trend chart, min/avg/max stats, recent packet log |
| **Control**     | Immobilizer toggle + custom polygonal geofencing management |
| **Map**         | Full-screen Leaflet map with live vehicle position       |
| **Packets**     | Real-time packet field table with decoded values         |

---

## 🌐 Web Dashboard Pages

| Page               | Description                                              |
|--------------------|----------------------------------------------------------|
| **Dashboard**      | Live scooter SVG, DI/DO status, voltage bar, GPS coords  |
| **Analytics**      | Recharts voltage trend, battery % card                   |
| **Device Control** | Immobilizer, geofencing zones, polygon draw tool         |
| **Map View**       | Leaflet map with real-time GPS marker                    |
| **Packet Viewer**  | Full raw packet inspection with all 30+ fields           |
| **Network Info**   | RSSI, operator, IMEI, MQTT diagnostics                   |

---

## 🔐 Security

- MQTT connection secured with **TLS 1.2** (MQTTS over WSS)
- JWT-based authentication on the broker
- Immobilizer state changes require explicit user confirmation modal
- Auto-immobilize geofence events logged with timestamp

---

## 📄 License

This project is proprietary. All rights reserved.

---

> Built with ❤️ as part of the **VAJRA Smart Telematics** initiative — powering smarter, safer EV fleet management.
