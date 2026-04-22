# Shelly Dimmer – Q-SYS Plugin

Control a Shelly Dimmer (Gen1 or Gen2) directly from Q-SYS Designer over the local network via HTTP.

---

## Requirements

- Shelly Dimmer 1, Dimmer 2 (Gen1) or Shelly Dimmer 0/10V / Dimmer 2PM (Gen2)
- Device on the same LAN as the Q-SYS Core
- Q-SYS Designer 9.x or later

---

## Installation

1. Copy `ShellyDimmer.qplug` to your plugins folder:
   - **Windows:** `%USERPROFILE%\Documents\QSC\Q-SYS Designer\Plugins`
   - **Mac:** `~/Documents/QSC/Q-SYS Designer/Plugins`
2. Restart Q-SYS Designer (or press **F5** to reload plugins).
3. Find **Shelly Dimmer** under the *Lighting* category in the component library.

---

## Properties

| Property | Description |
|---|---|
| **IP Address** | Local IP of the Shelly device |
| **Device Generation** | `Gen1` or `Gen2` — must match your hardware |
| **Poll Interval (s)** | How often Q-SYS reads device state (1–60 s) |

---

## Controls & Pins

| Control | Type | Direction | Description |
|---|---|---|---|
| `online` | Indicator | Output | Green = reachable, Red = offline |
| `status_message` | Text | Output | Human-readable state string |
| `power` | Button (Toggle) | Both | Turn the light on or off |
| `brightness` | Knob (0–100%) | Both | Dim level |
| `transition` | Knob (0–5000 ms) | Both | Fade time (converted to seconds for Gen2) |

---

## Gen1 vs Gen2 API notes

- **Gen1** uses plain HTTP GET requests: `GET /light/0?turn=on&brightness=75`
- **Gen2** uses JSON-RPC over HTTP POST: `POST /rpc/Light.Set`
- Set the **Device Generation** property correctly — the plugin handles all protocol differences internally.

---

## Changelog

| Version | Notes |
|---|---|
| 1.1.0 | Added Gen2 (JSON-RPC) support |
| 1.0.0 | Initial release — Gen1 only |
