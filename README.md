Markdown
#  Autonomous Sewer & Pipeline Inspection Robot

<div align="center">

**An intelligent, dual-controller inspection robot designed for hazardous environments. Features real-time gas detection, autonomous obstacle avoidance, and a live web-based dashboard.**

</div>

---

##  Table of Contents

- [Overview](#-overview)
- [Key Features](#-key-features)
- [Hardware Architecture](#-hardware-architecture)
- [System Architecture (Software)](#-system-architecture-software)
- [Circuit & Wiring Reference](#-circuit--wiring-reference)
- [Power Distribution](#-power-distribution)
- [Autonomous Logic](#-autonomous-logic)
- [Installation & Setup](#-installation--setup)
- [License](#-license)

---

##  Overview

Our Robot is a 4-wheeled rover built to navigate GPS-denied, hazardous environments like sewers, underground drains, and pipelines. To ensure maximum stability and real-time responsiveness, the system uses a **dual-controller architecture**:
1. **Raspberry Pi Zero 2W:** Acts as the brain, hosting a Flask web server, rendering the UI dashboard, and communicating over Wi-Fi.
2. **Arduino UNO:** Acts as the nervous system, handling all real-time motor PWM, gas sensor ADC readings, and ultrasonic ping timing.

This setup completely eliminates the need for external ADC chips and keeps motor control logic safe from network latency.

---

##  Key Features

* **Dual Navigation Modes:** Switch seamlessly between **Manual Control** via the web dashboard and **Autonomous Mode** where the robot drives itself.
* **Active Obstacle Avoidance:** Uses an HC-SR04 ultrasonic sensor mounted on an SG90 servo to "look" around and avoid dead-ends.
* **Real-Time Gas Detection:** Three dedicated MQ sensors monitor the environment for explosive and toxic gases (LPG, Methane, CO₂, NH₃, H₂S).
* **High-Speed Telemetry:** The Arduino streams multiplexed sensor data (Gas levels, Distance, Current Mode) to the Pi over serial every 200 milliseconds.
* **Browser-Based Dashboard:** A sleek, dark-mode web interface displaying live gas trending charts and responsive D-pad controls.

---

## 🛠️ Hardware Architecture

### Processing & Communication
| Component | Role |
|-----------|------|
| **Raspberry Pi Zero 2W** | Web server, Wi-Fi connectivity, command routing |
| **Arduino UNO** | Motor driver logic, 5V sensor polling, servo control |

### Motion & Sensors
| Component | Specification / Role |
|-----------|----------------------|
| **ZK-4WD Chassis** | 4-wheel drive platform with TT Gear Motors |
| **L298N Motor Driver** | Dual H-bridge for skid-steer driving |
| **MQ-2 / MQ-135 / MQ-136** | Analog gas sensors detecting hazardous fumes |
| **HC-SR04 + SG90 Servo** | "Radar" system for autonomous pathfinding |

---

## 🧠 System Architecture (Software)

    ┌──────────────────────────────────────────────────────────┐
    │                    OPERATOR (Browser)                    │
    │            http://<robot-ip>:5000                        │
    └─────────────────────┬────────────────────────────────────┘
                          │ Wi-Fi Command / Telemetry
                          ▼
    ┌──────────────────────────────────────────────────────────┐
    │              Raspberry Pi Zero 2W                        │
    │         Flask Web Server (app.py)                        │
    └─────────────────────┬────────────────────────────────────┘
                          │ USB Serial (115200 Baud)
                          ▼
    ┌──────────────────────────────────────────────────────────┐
    │                 Arduino UNO (C++)                        │
    │  ┌───────────────┐  ┌───────────────┐  ┌─────────────┐   │
    │  │  Motor Ctrl   │  │  Gas Sensors  │  │ Ultrasonic  │   │
    │  │   D2 to D5    │  │   A0 to A2    │  │   D6 & D7   │   │
    │  └──────┬────────┘  └──────┬────────┘  └──────┬──────┘   │
    └─────────┼──────────────────┼──────────────────┼──────────┘
              ▼                  ▼                  ▼
       ┌─────────────┐    ┌──────────────┐   ┌────────────┐
       │    L298N    │    │ MQ-2, MQ-135 │   │  HC-SR04   │
       │Motor Driver │    │    MQ-136    │   │  & Servo   │
       └─────────────┘    └──────────────┘   └────────────┘

---

##  Circuit & Wiring Reference

### Raspberry Pi Zero 2W → Arduino UNO
| RPi Connection | Target Port | Function |
|---------------|-------------|---------|
| Micro-USB (Data Port) | Arduino USB-B | Serial Comm (115200 baud) & 5V Power |

### Arduino UNO → L298N Motor Driver
| Arduino Pin | L298N Pin | Function |
|------------|-----------|---------|
| Digital 2 | IN1 | Left Motor FWD |
| Digital 3 | IN2 | Left Motor REV |
| Digital 4 | IN3 | Right Motor FWD |
| Digital 5 | IN4 | Right Motor REV |

### Arduino UNO → Sensors & Servo
| Arduino Pin | Target Component | Target Pin | Function |
|------------|------------------|------------|---------|
| Analog A0 | MQ-2 Sensor | AOUT | Gas Level 1 |
| Analog A1 | MQ-135 Sensor | AOUT | Gas Level 2 |
| Analog A2 | MQ-136 Sensor | AOUT | Gas Level 3 |
| Digital 6 | HC-SR04 | TRIG | Ultrasonic Trigger |
| Digital 7 | HC-SR04 | ECHO | Ultrasonic Echo |
| Digital 11 | SG90 Servo | Signal | Radar Sweep |

---

##  Power Distribution

To prevent the power-hungry motors from crashing the logic boards, the system relies on separated power sources:

| Rail | Source | Powers |
|------|--------|--------|
| **V-Motor (~12V)** | Lithium-ion Battery | L298N 12V Terminal (Motor Power) |
| **5V (Main)** | 5V Power Bank | RPi Zero 2W (via PWR IN Micro-USB port) |
| **5V (USB)** | RPi USB Data Port | Arduino UNO (via blue USB cable) |
| **5V (Sensors)** | Arduino 5V Pin | MQ Sensors (VCC), HC-SR04 (VCC), Servo |

>  **CRITICAL:** The ground (`GND`) wires of the Li-ion battery, the L298N, the Arduino, and the Raspberry Pi **MUST** all be connected together. If grounds are isolated, the motor control signals will fail.

---

##  Autonomous Logic

When switched to `MODE:A` (Autonomous), the Arduino takes complete control of driving. 
* The robot drives forward as long as the path is clear for at least 30 cm.
* If an obstacle is detected within 30 cm, the robot halts.
* The servo sweeps the ultrasonic sensor to 150° (Left) and 30° (Right) to measure distances.
* The robot compares the left and right distances; it turns toward the direction with the most clearance. 
* If both sides are blocked, it executes a backup and reverse-turn maneuver.

---

##  Installation & Setup

1. **Flash Arduino:** Upload the `arduino.ino` sketch to your Arduino UNO using the Arduino IDE.
2. **Setup Pi:** Install Raspberry Pi OS Lite on the Pi Zero 2W and enable Wi-Fi.
3. **Install Dependencies:**
   ```bash
   sudo apt update
   sudo apt install python3-pip
   pip3 install flask pyserial
Run the Server:

4. Bash
python3 app.py
5. Connect: Open a web browser on a device connected to the same Wi-Fi network and navigate to http://<raspberry-pi-ip>:5000.

 License
This project is licensed under the MIT License.
