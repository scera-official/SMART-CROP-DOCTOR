

# 🌾 Smart Crop Doctor — Offline-First AI Crop Disease Diagnosis Device
[![Platform](https://img.shields.io/badge/Platform-ESP32--CAM%20%7C%20Android-green.svg)](https://espressif.com)
[![Language](https://img.shields.io/badge/Language-C%2B%2B%20%7C%20Kotlin-purple.svg)](https://kotlinlang.org/)
[![AI Framework](https://img.shields.io/badge/AI Engine-TinyML%20%2B%20Gemini%201.5%20Flash-blue.svg)](https://ai.google.dev/)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Authority](https://img.shields.io/badge/Authority-SCERA-orange.svg)](#)
> **Document Identifier:** SCERA-SPEC-2026-SCD01  
> **Prepared by:** SCERA Embedded Systems & AI Research Group (Lead: Sanjay)
**Smart Crop Doctor** is a battery-powered, handheld diagnostic unit engineered for real-time, in-situ plant pathology assessment in connectivity-blind agricultural fields. Built on an **ESP32-CAM** microcontroller architecture, the system employs a dual-stage hybrid inference pipeline: an ultra-low-power offline quantized **TensorFlow Lite Micro** model for sub-second localized edge detection, paired with an automated fallback to the **Google Gemini Vision API** whenever cellular or Wi-Fi connectivity is available.
---
## 📸 Key Features & Capabilities
* **100% Offline Autonomy (TinyML):** Runs an 8-bit quantized MobileNet model locally on the ESP32-CAM's PSRAM, diagnosing top regional crop diseases in under 500 ms without internet access.
* **Deterministic Cloud Fallback (Gemini API):** Automatically switches to the Google Gemini 1.5 Flash Vision API when local prediction entropy is high or an ambiguous pathogen is scanned.
* **Automated Leaf Necrosis Calculation:** Uses onboard HSV color thresholding algorithms to calculate the ratio of infected leaf surface area to estimate disease severity percentage.
* **Local Language Voice Output:** Driven by a DFPlayer Mini audio module to provide audio diagnostic steps and organic remedies for low-literacy farmers.
* **Seasonal Epidemiological Logging:** Records every diagnostic event (crop type, pathogen, confidence score, timestamp, and optional GPS data) directly to an onboard MicroSD card in JSON format.
* **Field-Ready Power & Optics:** Powered by a rechargeable 18650 Li-Ion cell with an integrated 45° optical shroud to maintain consistent focal distance and lighting.
---
## 🏗️ System Architecture
```text
+-----------------------------------------------------------------------------------+

| SMART CROP DOCTOR PAYLOAD |
| :--- |
| +--------------------+        +-----------------------+        +--------------+ |
|  | USER INPUT |  | OPTICAL SENSOR |  | POWER SYSTEM |  |
|  | - Crop Select BTN |  | ESP32-CAM (OV2640) |  | 18650 Li-Ion |  |
|  | - Capture BTN |  | UXGA / JPEG Capture |  | TP4056 + BMS |  |
| +---------+----------+        +-----------+-----------+        +------+-------+ |
| :--- | :--- | :--- | :--- |
| +---------------+---------------+ |  |
| :--- | :--- | :--- |
| v                                           v |
| +------------------------------+                +------------------+ |
|  | ESP32-WROVER MICROCONTROLLER | ---------------- | 3.3V / 5V Regs |  |
|  | 240MHz Dual Core, 8MB PSRAM | +------------------+ |
| +--------------+---------------+ |
| :--- | :--- |
| +------------------+------------------+ |
| :--- | :--- | :--- |
| v                                     v |
| [STAGE 1: OFFLINE]                 [STAGE 2: ONLINE] |
| TFLite Micro Edge Model            Gemini Vision API Gateway |
| - Quantized INT8 (< 500KB)          - HTTPS / REST Payload |
| - Conf. Threshold >= 80%           - Low Conf. or Manual Override |
| :--- | :--- | :--- |
| +------------------+------------------+ |
| :--- | :--- |
| v |
| +-----------------------------------------------------------------------------+ |
|  | OUTPUT & LOGGING LAYER |  |
|  | - 0.96" SSD1306 OLED (I2C)         - DFPlayer Mini + Speaker (UART Audio) |  |
|  | - MicroSD Card Module (SPI Log)    - Optional Neo-6M GPS (UART Telemetry) |  |
| +-----------------------------------------------------------------------------+ |

+-----------------------------------------------------------------------------------+

📊 Stage 1 (TinyML) vs. Stage 2 (Gemini API) Comparison
Stage 1: Offline TinyML Model
Inference Latency: < 450 ms
Power Consumption: ~ 160 mA @ 3.3V
Connectivity Need: Zero (100% Standalone)
Model Size: 480 KB (INT8 Quantized MobileNet)
Diagnostic Scope: Top 10 Regional Crop Pathologies
Output Detail: Class Label + Severity Score

Stage 2: Gemini 1.5 Flash Cloud API
Inference Latency: 1,200 ms – 2,500 ms (network dependent)
Power Consumption: ~ 280 mA @ 3.3V (Wi-Fi Peak)
Connectivity Need: Active Internet Link (Wi-Fi / Mobile Hotspot)
Model Size: Multi-Billion Parameter Foundation Model
Diagnostic Scope: Open-World (1,000+ Pathologies & Variants)
Output Detail: Rich Text: Pathogen ID, Stage, Remedy, Prevention


🛠️ Hardware Component Pinout Map
GPIO 4: Onboard Flash LED — Illumination Trigger (Active HIGH during capture)

GPIO 12: MicroSD Card MOSI — SPI Bus Data Out (Tri-stated during boot)

GPIO 13: MicroSD Card CS — SPI Chip Select (Active LOW)

GPIO 14: SSD1306 OLED SDA / SCK — Shared I2C / SPI Clock (Time-multiplexed)

GPIO 15: SSD1306 OLED SCL — I2C Clock 400kHz (External 4.7kΩ Pull-up)

GPIO 2: Button 1 (Crop Select) — Digital Input (Active LOW Interrupt)

GPIO 0: Button 2 (Capture Trigger) — Digital Input / Boot (Active LOW Interrupt)

GPIO 1 / 3: DFPlayer Mini Audio TX/RX — Hardware UART 9600 Baud (Serial Telemetry Stream)


💰 Bill of Materials (BOM) Cost Breakdown
Microcontroller Board + Camera: ESP32-CAM (Ai-Thinker + OV2640) — Qty: 1 — ₹650
Visual Interface Display: 0.96" Monochrome I2C OLED (SSD1306) — Qty: 1 — ₹220
Audio Output Subsystem: DFPlayer Mini Board + 2W Speaker — Qty: 1 — ₹180
Storage Module: MicroSD Card Adapter + 16GB Card — Qty: 1 — ₹320
Power Subsystem: 18650 2600mAh Li-Ion + TP4056 BMS — Qty: 1 — ₹280
Physical Structural Housing: 3D Printed PETG Chassis + Hood — Qty: 1 — ₹450
Interconnects & Passives: Switches, Resistors, PCB Board — Qty: 1 — ₹150
Total Estimated Prototype Cost: ₹2,250 (~ $27 USD)

💻 Software Setup & Installation
Firmware Deployment (ESP32-CAM)

Demonstration Protocol for Competition Judges
Offline Mode Verification:
Ensure external Wi-Fi / Hotspot is switched OFF.
Press

 Button 1 to select crop type (Tomato).
Align leaf tissue under optical shroud and press 

Button 2.
Result: Local TinyML model identifies the pathogen (e.g., Early Blight) and displays diagnosis + plays local language audio within < 500 ms.
Hybrid Cloud Fallback Verification:
Switch Wi-Fi Hotspot ON.

Scan an ambiguous or multi-pathogen mutated leaf surface.

Result: Unit detects local prediction confidence below threshold (Confidence < 0.80), triggers Gemini 1.5 Flash API via REST payload, and renders a refined multi-stage treatment plan.

Data Telemetry Audit:
Remove the MicroSD card and mount it to a reader to demonstrate formatted .json spatial-temporal disease spread logs.

📄 License & Attribution
Distributed under the MIT License. See LICENSE for details.
Developed under the Space & Agriculture Systems Research Authority (SCERA) embedded intelligence program.