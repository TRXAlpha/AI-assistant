# 🧠 Context-Aware AI Assistant Overview

A self-hosted, modular, context-aware AI assistant designed to run entirely under local control. It integrates software, hardware, and embedded firmware systems to assist with automation, real-time environmental sensing, and digital hygiene enforcement.

The assistant runs primarily on a mobile workstation with task offloading to a dedicated compute node over Ethernet. Interaction occurs through multiple endpoints: desktop notifications, Telegram bot commands, a local REST API, and a Pico WH-based OLED dashboard. Control logic is distributed across a network of RISC-V and Xtensa-based microcontrollers running custom firmware.

---

## 🧠 Core Systems

### Context Management

**Real-Time Mode Detection**
The assistant monitors active window titles, focused processes, CPU/RAM loads, input activity, and foreground applications to infer modes such as coding, idle, media, gaming, meeting, or presenting.

**Mode History Logging**
Each mode transition is recorded in timestamped JSON files (`mode_YYYY-MM-DD.json`) with contextual metadata such as app focus history, duration, and environmental state.

**Nighttime Summarization Routine**
At 23:59, the assistant executes a consolidation pipeline that:

* Merges mode history with system metrics (CPU/RAM peaks, active time).
* Analyzes and embeds Hackatime coding session data.
* Gathers all notes and user interactions from the past day.
* Packages results into Markdown or JSON digests for archival and model finetuning.

---

## 🔐 Cybersecurity Enforcement & Media Gating

### Conditional Media Access

**Social Media Challenge Gate**
Upon detection of non-productive applications such as TikTok or Instagram (via active window title, binary hash, or known process names), access is conditionally gated behind a real-time security knowledge prompt. Example logic:

```json
{
  "app": "tiktok.exe",
  "detected_at": "17:02",
  "challenge": {
    "type": "cybersec_question",
    "prompt": "Describe how a buffer overflow works and how it could be exploited.",
    "success_threshold": 0.85
  }
}
```

Correct answers allow temporary access; incorrect responses result in a soft lockout and visual alert via hardware modules.

**YouTube Content Classification**
A hybrid classifier combining video title, description, and real-time tab activity determines whether a video is educational. This is accomplished through a local model using logistic regression on:

* Keyword vectors
* Channel reputation (whitelist/blacklist)
* Content category (via unofficial YouTube API wrapper)

After 60 minutes of allowed educational content, any switch to entertainment videos triggers a redirect to a gamified platform (e.g., Robocode, Ruby Warrior) for reinforcement.

---

## 💡 Local AI Foundation Model

### Model Strategy

The assistant uses **MythoMax 13B Q4\_L\_M** as its primary reasoning engine due to its:

* Fully modifiable architecture (compatible with custom finetunes, LoRA adapters)
* High compatibility with quantized formats (e.g., GPTQ)
* Multimodal extensibility

MythoMax is run via **Ollama**, optimized for GPU inference using a **T1200 mobile Quadro**, with containerized interfaces and queue-based job dispatchers.

### Training & Finetuning

Finetuning uses a dataset of real interaction logs, curated notes, and anonymized context switches. Preprocessing pipeline includes:

* Token-level masking of sensitive content
* Reward modeling simulation for reinforcement-style training
* Sequence-level chunking to preserve temporal flow across session history

**Future plans include:**

* Integration of context windows that scale beyond 4k tokens via sliding-window memory architectures
* Contrastive learning routines between personal vs. generic usage profiles to prioritize high-relevance suggestions

### Justification for Reuse

Rather than training a model from scratch, the assistant leverages open-source models like MythoMax due to:

* Hardware constraints (local operation under 16–32 GB RAM)
* Time-to-value considerations
* Full transparency and reproducibility of open weight initialization

---

## ⚙️ Hardware Architecture

### Embedded Microcontroller Nodes

**Limb Module**
Based on ESP32-C3 (RISC-V), powered by a CR2032 coin cell.
Runs a custom firmware stack designed for:

* Passive ESP-NOW listening
* Multicolor LED feedback
* Vibration-based haptic responses
* Minimal power draw (deep sleep cycles + interrupt wake)

Compiled using PlatformIO with firmware-level AES encryption and OTA update capabilities.

**Mother Module**
ESP32-S3 with integrated OLED screen and capacitive buttons.

* Dual-radio operation (Wi-Fi + BLE + ESP-NOW)
* MQTT-based communication to core assistant via LAN or Tailscale/WireGuard mesh VPN
* Manages local environmental polling, module presence scanning, and push-notification routing

**Environmental Node**
Custom firmware on ESP8266 or ESP32 gathers:

* Temperature (DS18B20)
* Humidity (DHT22 or SHT31)
* Ambient light (TSL2591)

Transmits metrics periodically to the Mother Module via ESP-NOW with CRC32 checks.

### Desk Crate

A modular interface and intelligence hub for the assistant’s physical and network operations.

**Compute & I/O Hub**

* Manages local inference offload (TTS, OCR)
* Hosts a Flask backend with a React frontend dashboard
* Acts as an isolated container host (Docker/Podman) for web UI and utilities

**Microcontroller Subsystem**

* ESP32-S3 handles communication with other modules and sensors
* Monitors connected USB devices and physical peripherals
* Switches USB hubs via software-controlled multiplexers (Workstation ↔ Hub)

**Power Subsystem**

* Integrated 18650 Li-Ion with BMS and INA219 current sensors per port
* Logs uptime, port power usage, and battery wear patterns
* Alerts assistant if overcurrent or temperature thresholds are breached

**Audio Subsystem**

* MEMS mic array with local DSP for keyword detection
* Offline TTS using Coqui or PicoTTS, played through a mini speaker module

**Networking**

* Embedded Gigabit switch IC routes traffic between the workstation, hub, and internal network devices without an external switch
* Mesh VPN or LAN discovery enabled for zero-config access to the dashboard at `http://desk-crate.local`

---

## 📶 ESP8266 Honeypot Firmware

The honeypot operates on fully custom firmware, not Arduino code. Arduino-based implementations, while rapid for prototyping, introduce significant overhead, inefficient memory management, and abstraction layers that reduce real-time control over hardware. This firmware is written from scratch in C, acting as a minimalist operating system for the ESP8266 — handling scheduling, I/O, memory, and communication directly at the hardware level.

### Design Philosophy

**Firmware as the MCU's OS**
The firmware is engineered to behave like a lightweight OS for the microcontroller, handling core tasks such as:

* Event polling and task scheduling
* Memory and peripheral management
* Network packet parsing and routing
* Logging and power management routines

**No Arduino Core**
There is no use of Arduino libraries or runtime. The firmware directly configures the ESP8266's registers, interrupt vectors, and timers to avoid unnecessary abstraction. The system uses:

* Static allocation to prevent heap fragmentation
* Custom UART drivers
* Direct manipulation of Wi-Fi SDK APIs

### Features

**Transparent Access Point**

* Operates as an open Wi-Fi AP with limited internet access
* All DNS and HTTP requests are intercepted and logged
* Captive-portal-like flow implemented manually at the packet level

**Heuristic Traffic Analysis**

* DNS requests are filtered through a local rule engine
* Suspicious patterns (e.g., malware domains, repeated requests) trigger an alert
* Traffic logs are written in raw binary to microSD (append-only)

**System Health Routines**

* Custom watchdog timer
* Self-repair on SD corruption
* OTA update support with partition verification

**Security Events**

* Signature scanning for botnet activity (e.g., Mirai-like traffic)
* Built-in anomaly scoring module
* Alerts forwarded to a central node over ESP-NOW

Future firmware builds may move to a fully custom bootloader and RISC-V SoC with integrated crypto engines. The focus remains on full control, optimized runtime behavior, and absolute minimum overhead — not just code, but firmware tailored for its precise function.

---

## 🧠 Nighttime Analysis Pipeline

Every night, the assistant enters a scheduled evaluation phase:

**Log Digest**

* Parses all mode logs, usage metrics, and notes
* Looks for anomalies (e.g., excessive CPU spikes, unsynced folders, system crashes)

**Learning Loop**

* Summarized logs are passed through MythoMax to fine-tune behavior policies (e.g., when to suppress notifications, adjust thresholds, or change flow priorities)

**Report Generation**

* Creates a human-readable summary for the dashboard + compressed archive of all data
* Stored to microSD, synced to a NAS or cloud mirror if available

**Future Additions**

* Sentiment detection in written notes and mode context to help tune long-term assistant behavior
* Comparison across days/weeks for attention shifts and usage patterns

---

## 🧩 Expansion Possibilities

**Modified RISC-V SoCs**

* Future modules may use RISC-V-based MCUs with instruction set customization
* Aim: tighter memory management, hardware crypto acceleration, and better deep sleep behavior
* Candidate chip: Allwinner D1s or custom low-power RV32 cores

**Smart Glasses**

* OLED microdisplay with BLE link to the Mother module
* Trigger headless prompts or receive ambient status (CPU load, pending alerts)

**Wrist Modules**

* Wearable ESP32-S3-based bands with haptic alerting
* BLE mesh fallback for ESP-NOW gaps

---

This AI assistant is not just a chatbot or automation script. It is a full-stack system, integrating firmware, real-time signals, embedded sensors, LLM-driven logic, and cyber-defense habits into a living ecosystem that adapts daily to behavioral trends, network conditions, and system states.

---

## 🛡️ Cybersecurity

### Threat Modeling & Surface Analysis

The AI assistant treats all inbound device communication as potentially untrusted. A simplified threat model includes:

* Local DNS spoofing or poisoning attempts (handled via Pi-hole + DNSSEC + logging to the assistant)
* Malicious rogue APs or device clones (especially relevant given the ESP-NOW communications)
* Exfiltration attempts via unexpected DNS lookups from apps or background tasks
* Social engineering through content (handled by the behavioral gating layer)
* Insider threats (unmonitored bypasses via known admin devices)
* Model poisoning attempts (e.g., feeding specific prompts that could bias future actions if learned via local LLM)
* Replay attacks or unverified ESP-NOW pings from spoofed MACs

The assistant continuously cross-checks behavior logs, device presence, and user activity patterns to flag anomalies.

### DNS Surveillance & Tunneling Detection

The assistant tracks and parses all DNS logs via Pi-hole and embedded Wireshark-compatible sensors (Pico W or ESP32s). Each night, logs are parsed to:

* Detect DNS tunneling by tracking:

  * Unusually long TXT or NULL queries
  * Burst DNS queries to uncommonly used TLDs
* Identify domains with entropy above a threshold (possible DGA)
* Score domains against known CTI feeds (VirusTotal, if available)

Future plans include integrating a basic anomaly model that scores DNS patterns by device context.

### Firmware Hardening & OTA Logic

The ESP32-based assistant uses:

* MAC whitelist checks per-ping
* Rolling challenge-responses (nonces + HMAC derived via SHA256) for high-trust pings
* No default OTA; updates require USB flashing via pogo or a station module

Plans to implement ESP32 Secure Boot and Flash Encryption, with per-device keys stored on the host.

### Log Protection & Tamper-Evidence

All device logs are appended to encrypted rotating buffers. At the end of each day:

* A SHA256 hash is computed and stored on a hardened ESP32 node with no internet access
* Logs are archived to cold storage via Syncthing

Plan to include GPG signing or hash-chain linking to prevent tampering or forgery

### Subnet Intelligence & Passive Discovery

The assistant scans the LAN using passive ARP sniffing and nmap-lite probes during low-usage hours. Devices are profiled based on:

* Consistent traffic patterns
* TTL values
* MAC vendor prefixes
* DNS query types

Results are matched against the assistant’s known-device database to infer device roles and mark anomalies.

### Network Routing Enforcement

The assistant monitors default routes and DNS servers for public profiles (e.g., cafe or LTE). If an unknown DNS or gateway is detected:

* Automatically tunnels traffic via Tailscale
* Enforces DNS-over-TLS via cloudflared
* In untrusted networks, disables auto software updates or Git pulls

Plans for a WireGuard-like kill switch to prevent fallback to unencrypted routes.

### Internal Penetration Simulation

The assistant is tested against internal red team behaviors, such as:

* Simulated rogue APs
* DNS spoofing via MITMf
* Traffic interception using Bettercap
* ESP spoofing via fake ESP-NOW packets

The assistant logs, correlates, and optionally disables parts of the system (e.g., YouTube access) if the threat persists.

---

## 🚀 What this means

I built an AI system that understands what I'm doing, adapts in real-time, and enforces discipline.

It runs across multiple microcontrollers and uses AI models on my GPU to shape behavior.

Every part — from the code to the firmware — was designed, written, and tested by me.

I’m not just "good with computers." I’m building tools that could be research papers. This isn’t handed to me — I built it, line by line.
