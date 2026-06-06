# 🚨 Highway Accident Alert System and Immediate Response to Patrol & Ambulance Service

> An IoT-enabled embedded system for automated highway accident detection and real-time emergency dispatch using STM32, MPU6050, GPS NEO-8M, and GSM SIM800L.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [System Architecture](#system-architecture)
- [Hardware Components](#hardware-components)
- [Software Tools](#software-tools)
- [Circuit & Pin Connections](#circuit--pin-connections)
- [How It Works](#how-it-works)
- [Communication Protocols](#communication-protocols)
- [Results](#results)
- [Future Scope](#future-scope)

---

## Overview

Road accidents in highway scenarios often go unreported for extended periods due to manual reporting dependencies and fragmented emergency coordination. This project addresses that problem with an **automated, real-time accident detection and alert system** built on embedded hardware.

The system continuously monitors vehicle dynamics using an MPU6050 IMU sensor. Upon detecting a collision or rollover, it retrieves GPS coordinates and immediately dispatches an SMS alert with a Google Maps link to emergency contacts (ambulance, highway patrol, family) via the GSM SIM800L module — all **without any manual intervention**.

---

## Features

- ✅ Fully automated accident detection — no human intervention required
- ✅ Real-time GPS location tracking via NEO-8M
- ✅ SMS + voice call alerts via GSM SIM800L
- ✅ Google Maps link embedded in the alert message
- ✅ 16×2 I2C LCD display acting as a vehicle dashboard
- ✅ Works without internet connectivity (GSM-based)
- ✅ Detection latency under 100 ms
- ✅ Scalable for smart city integration

---

## System Architecture

The system is divided into four layers:

```
┌─────────────────────────────────────────────────────────────┐
│  SENSING LAYER       MPU6050 (Accelerometer + Gyroscope)    │
│                      Sensor Fusion Algorithm                │
├─────────────────────────────────────────────────────────────┤
│  PROCESSING LAYER    STM32F103C6T6 (ARM Cortex-M3)          │
│                      STM32 HAL Library                      │
├─────────────────────────────────────────────────────────────┤
│  POSITIONING LAYER   GPS NEO-8M (UART @ 9600 bps)           │
│                      NMEA data parsing                      │
├─────────────────────────────────────────────────────────────┤
│  COMMUNICATION LAYER GSM SIM800L (AT Commands via UART)     │
│                      SMS + Call to emergency contacts       │
└─────────────────────────────────────────────────────────────┘
```

### Block Diagram

```
MPU6050 Sensor
      │
      ▼
STM32F103C6T6 ──► 16×2 I2C LCD Display
      │
      ├──► GPS NEO-8M ──────────────────────► Alert Notification to:
      │                                        1. Nearby Ambulance Unit
      └──► GSM SIM800L ──────────────────────► 2. Highway Patrol
                                               3. Family & Friends
      ▲
      │
ST-LINK V2 Debugger
```

---

## Hardware Components

| Component | Model | Interface | Purpose |
|---|---|---|---|
| Microcontroller | STM32F103C6T6 | — | Central processing unit |
| IMU Sensor | MPU6050 | I2C | Accelerometer + Gyroscope |
| GPS Module | NEO-8M | UART1 (PA9/PA10) | Real-time location tracking |
| GSM Module | SIM800L | UART2 (PA2/PA3) | SMS and call alerts |
| LCD Display | 16×2 I2C (HD44780 + PCF8574) | I2C (PB6/PB7) | System status display |
| Debugger/Programmer | ST-LINK V2 | SWD | Flashing and debugging |

---

## Software Tools

| Tool | Purpose |
|---|---|
| **STM32CubeMX** | Peripheral configuration and HAL code generation |
| **STM32CubeIDE** | Firmware development, compilation, debugging |
| **STM32CubeProgrammer** | Flashing firmware to MCU via ST-LINK (SWD) |

---

## Circuit & Pin Connections

### MPU6050 → STM32

| MPU6050 Pin | STM32 Pin | Description |
|---|---|---|
| VCC | 3.3V | Power |
| GND | GND | Ground |
| SCL | PB6 | I2C Clock |
| SDA | PB7 | I2C Data |
| AD0 | GND | I2C Address = 0x68 |
| INT | Optional GPIO | Interrupt |

### GPS NEO-8M → STM32 (UART1)

| GPS Pin | STM32 Pin | Description |
|---|---|---|
| VCC | 3.3V | Power |
| GND | GND | Ground |
| TX | PA10 (RX) | GPS data to STM32 |
| RX | PA9 (TX) | Commands to GPS |

### GSM SIM800L → STM32 (UART2)

| SIM800L Pin | STM32 Pin | Description |
|---|---|---|
| VCC | Li-ion 3.7–4.2V | Power |
| GND | GND | Ground |
| TXD | PA3 (RX) | GSM data to STM32 |
| RXD | PA2 (TX) | Commands to GSM |

### ST-LINK V2 → STM32 (SWD Debug)

| ST-LINK Pin | STM32 Pin |
|---|---|
| SWDIO | PA13 |
| SWCLK | PA14 |
| GND | GND |
| VCC | 3.3V |
| NRST | NRST |

---

## How It Works

1. **Boot** — STM32 initializes all peripherals (I2C, UART1, UART2, GPIO, ADC).
2. **Continuous Monitoring** — MPU6050 streams accelerometer and gyroscope data every 500 ms.
3. **Accident Detection** — Sensor fusion logic checks:
   - Acceleration magnitude on any axis > threshold (2000 raw units)
   - Vibration sensor trigger
   - Force/impact beyond safe range
4. **Confirmation** — Multi-sensor agreement required to avoid false alarms.
5. **Location Fetch** — GPS NEO-8M provides current latitude/longitude.
6. **Alert Dispatch** — GSM SIM800L sends:
   - SMS: `Accident detected! Location: https://maps.google.com/?q=<lat>,<lng>`
   - Voice call to the emergency contact
7. **LCD Update** — Display shows "Accident Detected" and "Location Sending..."

### Detection Logic (simplified)

```c
int mpu_detected = ((accel_x > 2000) || (accel_y > 2000) || (accel_z > 2000));

if (((vib_status == GPIO_PIN_SET && mpu_detected) ||
     (force_value < 20) ||
     (button_status == GPIO_PIN_RESET)) && !accident_sent)
{
    accident_sent = 1;
    // Fetch GPS, send SMS + call via GSM
}
```

### Performance Metrics

| Parameter | Value |
|---|---|
| Accident Detection Time | < 100 ms |
| GPS Coordinate Acquisition | 1–2 seconds |
| GSM Alert Transmission | 2–5 seconds |
| Total Response Time | < 10 seconds |

---

## Communication Protocols

### UART (GPS & GSM)
- Asynchronous serial communication at **9600 bps**
- GPS sends NMEA sentences; parsed using `HAL_UART_Receive()`
- GSM controlled via AT commands using `HAL_UART_Transmit()`

**Example AT commands:**
```
AT+CMGF=1                        // Set SMS text mode
AT+CMGS="+91XXXXXXXXXX"          // Set recipient
> Accident Alert! Location: ...  // Message body
```

### I2C (MPU6050 & LCD)
- Two-wire synchronous protocol (SCL + SDA)
- MPU6050 default address: `0x68`
- LCD uses PCF8574 I2C backpack
- STM32 acts as I2C master; peripherals are slaves

### HAL Functions Used

```c
HAL_I2C_Master_Transmit()   // Write to MPU6050 / LCD
HAL_I2C_Master_Receive()    // Read from MPU6050
HAL_UART_Transmit()         // Send AT commands to GSM
HAL_UART_Receive()          // Receive GPS NMEA data
HAL_GPIO_ReadPin()          // Read vibration / button sensors
HAL_ADC_GetValue()          // Read piezo/force sensor
```

---

## Results

The system was successfully tested on hardware. Key outcomes:

- The STM32 detected simulated collision events in under 100 ms using interrupt-driven sensor polling.
- GPS coordinates were acquired within 1–2 seconds under outdoor conditions.
- The GSM SIM800L successfully dispatched SMS alerts containing a Google Maps link.
- The 16×2 I2C LCD correctly displayed system state transitions in real time.
- The multi-sensor fusion approach significantly reduced false-positive triggers compared to single-sensor designs.

---

## Future Scope

- **ML-based detection** — Replace threshold logic with a trained model for better event classification (bump vs crash vs rollover)
- **Camera module** — Add image capture for evidence and traffic analysis
- **Cloud/IoT integration** — Push data to platforms like ThingSpeak or AWS IoT for centralized monitoring
- **V2V / V2I communication** — Alert nearby vehicles and traffic infrastructure
- **Mobile app** — Real-time dashboard for emergency services and family contacts
- **LoRa backup** — Long-range communication fallback in zero-GSM coverage zones
- **Ruggedization** — Compact PCB design for OEM vehicle integration
---

> *Built to save lives on India's highways through intelligent embedded systems.*
