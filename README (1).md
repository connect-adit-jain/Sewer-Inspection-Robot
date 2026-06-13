#  Autonomous Sewer Inspection Robot

<div align="center">


**An intelligent, remotely operated robot designed for real-time sewer and underground pipeline inspection - detecting hazardous gases, mapping thermal anomalies, and streaming live video, all from a browser-based control interface.**

</div>

---

##  Table of Contents

- [Overview](#-overview)
- [Key Features](#-key-features)
- [Hardware Components](#-hardware-components)
- [System Architecture](#-system-architecture)
- [Circuit & Wiring](#-circuit--wiring)
- [Power Architecture](#-power-architecture)
- [Software Stack](#-software-stack)
- [Project Structure](#-project-structure)
- [Installation & Setup](#-installation--setup)
- [Usage](#-usage)
- [Web Control Interface](#-web-control-interface)
- [Sensor Data](#-sensor-data)
- [Future Improvements](#-future-improvements)
- [License](#-license)

---

##  Overview

Our Robot is a 4-wheeled autonomous inspection robot built to navigate hazardous, GPS-denied environments such as sewers, underground drains, and pipelines. It is controlled remotely over Wi-Fi through a browser-based interface and transmits real-time sensor data including gas concentrations, thermal imagery, and live video feed.

The robot targets a critical real-world problem: **manual sewer inspection is dangerous, time-consuming, and expensive**. MinerBot provides a safer, smarter, and scalable alternative — fully operable by a technician from a laptop or mobile device above ground.

---

##  Key Features

| Feature | Details |
|--------|---------|
| Browser-Based Control | Control the robot from any device on the same network — no app needed |
|  Thermal Imaging | MLX90640 32×24 IR thermal array detects hotspots, leaks, and blockages |
| Gas Detection | Three MQ sensors detect methane, H₂S, CO₂, NH₃, smoke, and LPG |
|  Live Camera Feed | Pi Camera streams real-time video for visual inspection |
|  PWM Speed Control | Variable speed via L298N PWM inputs for precise maneuvering |
|  12V Li-ion Power | Dedicated power rails for motors, logic, and sensors |
|  Wi-Fi SSH Access | Headless operation via SSH over USB gadget mode or Wi-Fi |
|  Real-Time Dashboard | Sensor readings displayed live on the web control interface |

---

##  Hardware Components

### Core Controller
| Component | Specification |
|-----------|--------------|
| **Raspberry Pi Zero 2W** | Quad-core 64-bit ARM Cortex-A53 @ 1GHz, 512MB RAM |

### Chassis & Motion
| Component | Specification |
|-----------|--------------|
| **ZK-4WD Chassis** | 4-wheel drive, MDF plate base |
| **TT Gear Motors (×4)** | DC motors with gear reduction |
| **L298N Motor Driver** | Dual H-bridge, supports 12V/2A per channel |

### Sensors
| Component | Model | Detects |
|-----------|-------|---------|
| **Gas Sensor 1** | MQ-2 | LPG, methane, smoke, hydrogen |
| **Gas Sensor 2** | MQ-135 | CO₂, NH₃, benzene, air quality |
| **Gas Sensor 3** | MQ-136 | Hydrogen Sulfide (H₂S) - sewer-specific |
| **Thermal Camera** | MLX90640 | 32×24 IR array, I²C interface |


### Power & Signal Conditioning
| Component | Role |
|-----------|------|
| **12V Li-ion Battery Pack** | Primary power source |
| **LM2596 Buck Converter** | Steps 12V → 5V for RPi and MQ sensors |

---

##  System Architecture

```
┌──────────────────────────────────────────────────────────┐
│                    OPERATOR (Browser)                    │
│            http://<robot-ip>:5000                        │
└─────────────────────┬────────────────────────────────────┘
                      │ Wi-Fi / USB Gadget
                      ▼
┌──────────────────────────────────────────────────────────┐
│              Raspberry Pi Zero 2W                        │
│         Flask Web Server (app.py)                        │
│  ┌───────────────┐  ┌───────────────┐  ┌─────────────┐  │
│  │  Motor Ctrl   │  │  Gas Sensors  │  │   Thermal   │  │
│  │  GPIO 12/13   │  │  MCP3008 SPI  │  │  MLX90640   │  │
│  │  GPIO 17/18   │  │  MQ-2/135/136 │  │    I²C      │  │
│  └──────┬────────┘  └──────┬────────┘  └──────┬──────┘  │
└─────────┼──────────────────┼──────────────────┼──────────┘
          │                  │                  │
          ▼                  ▼                  ▼
   ┌─────────────┐    ┌──────────────┐   ┌────────────┐
   │    L298N    │    │   MCP3008    │   │ MLX90640   │
   │Motor Driver │    │  (SPI ADC)   │   │(32×24 IR)  │
   └──────┬──────┘    └──────┬───────┘   └────────────┘
          │                  │
    ┌─────┴────┐       ┌─────┴──────────┐
    │ 4× TT   │       │ MQ-2  MQ-135   │
    │ Motors  │       │ MQ-136         │
    └─────────┘       └────────────────┘
```

---

## Circuit & Wiring

### GPIO Pin Mapping (Raspberry Pi Zero 2W)

| GPIO Pin | BCM | Function | Connected To |
|----------|-----|----------|-------------|
| GPIO 12 | 12 | PWM0 - Motor A Speed | L298N ENA |
| GPIO 13 | 13 | PWM1 - Motor B Speed | L298N ENB |
| GPIO 17 | 17 | Motor A Direction 1 | L298N IN1 |
| GPIO 18 | 18 | Motor A Direction 2 | L298N IN2 |
| GPIO 27 | 27 | Motor B Direction 1 | L298N IN3 |
| GPIO 22 | 22 | Motor B Direction 2 | L298N IN4 |
| GPIO 10 | 10 | SPI MOSI | MCP3008 DIN |
| GPIO 9 | 9 | SPI MISO | MCP3008 DOUT |
| GPIO 11 | 11 | SPI CLK | MCP3008 CLK |
| GPIO 8 | 8 | SPI CE0 | MCP3008 CS |
| GPIO 2 | 2 | I²C SDA | MLX90640 SDA |
| GPIO 3 | 3 | I²C SCL | MLX90640 SCL |

### Motor Wiring (L298N)

```
Left Motors (Front-Left + Rear-Left)  ──┐
                                         ├──► L298N Channel A (OUT1, OUT2)
Right Motors (Front-Right + Rear-Right) ─┐
                                          ├──► L298N Channel B (OUT3, OUT4)
```
> Both motors on each side are wired **in parallel** to the same output channel.

### MQ Sensor Analog Input via MCP3008

```
MQ-2   Analog Out ──► Voltage Divider (10kΩ/20kΩ) ──► MCP3008 CH0
MQ-135 Analog Out ──► Voltage Divider (10kΩ/20kΩ) ──► MCP3008 CH1
MQ-136 Analog Out ──► Voltage Divider (10kΩ/20kΩ) ──► MCP3008 CH2
```

---

## 🔋 Power Architecture

```
12V Li-ion Battery Pack
         │
         ├──────────────────────────────────► L298N VSS (12V Motor Power)
         │                                            │
         │                                    4× TT Motors
         │
         └──► LM2596 Buck Converter (12V → 5V)
                        │
                        ├──────────────────────────────► Raspberry Pi Zero 2W (5V/2.5A)
                        │                                        │
                        │                               GPIO 3.3V Rail
                        │                                        │
                        │                                ┌───────┴────────┐
                        │                            MLX90640         MCP3008
                        │                           (3.3V I²C)      (3.3V SPI)
                        │
                        └──────────────────────────────► MQ Sensors (5V Heater + Logic)
```

>  **Important:** MQ sensors output 0–5V analog. Use 10kΩ + 20kΩ voltage dividers before connecting to MCP3008 inputs to stay within the 3.3V ADC reference range.

---

## 💻 Software Stack

| Layer | Technology |
|-------|-----------|
| OS | Raspberry Pi OS Lite 64-bit |
| Language | Python 3 |
| Web Framework | Flask |
| GPIO Control | RPi.GPIO |
| ADC Reading | spidev (MCP3008) |
| Thermal Camera | adafruit-circuitpython-mlx90640 |
| Camera Streaming | picamera2 |
| Remote Access | SSH over USB Gadget Mode |

---

##  Project Structure

```
MinerBot/
│
├── README.md                  # ← You are here
├── requirements.txt           # Python dependencies
├── .gitignore                 # Git ignore rules
├── LICENSE                    # MIT License
├── config.py                  # GPIO pin & hardware configuration
│
├── main.py                    # Core robot logic (motors, sensors, thermal)
├── app.py                     # Flask web server & control interface
│
├── docs/
│   ├── wiring_diagram.md      # Detailed wiring reference
│   └── setup_guide.md         # Step-by-step setup instructions
│
└── scripts/
    └── setup.sh               # Auto-install script for RPi
```

---

##  Installation & Setup

### Prerequisites

- Raspberry Pi Zero 2W with GPIO headers soldered
- Raspberry Pi OS Lite (64-bit) flashed to microSD
- Python 3.9+
- SSH access enabled (via USB gadget mode or Wi-Fi)

### Step 1 — Enable Required Interfaces

SSH into the Pi and run:

```bash
sudo raspi-config
```

Enable the following:
- **Interface Options → SPI** (for MCP3008)
- **Interface Options → I2C** (for MLX90640)
- **Interface Options → Camera** (for Pi Camera)

### Step 2 — Clone the Repository

```bash
git clone https://github.com/<your-username>/MinerBot.git
cd MinerBot
```

### Step 3 — Install Dependencies

```bash
pip install -r requirements.txt
```

Or use the auto-setup script:

```bash
chmod +x scripts/setup.sh
./scripts/setup.sh
```

### Step 4 — Run MinerBot

```bash
python app.py
```

The web control server starts at `http://<raspberry-pi-ip>:5000`

>  **Finding your Pi's IP:** Use the [Fing app](https://www.fing.com/) on your phone while on the same Wi-Fi network. Look for a device named `raspberrypi`.

---

##  Usage

### Connecting to MinerBot

1. Power on the robot.
2. Connect your laptop/phone to the **same Wi-Fi network** as the Pi.
3. Open a browser and go to: `http://<pi-ip-address>:5000`
4. Use the on-screen directional controls to navigate.

### SSH Access (for development)

```bash
# USB Gadget Mode
ssh pi@raspberrypi.local

# Via IP address (more reliable)
ssh pi@<ip-address>
```

### Running in Background (persistent session)

```bash
nohup python app.py &
# Or use tmux:
tmux new -s minerbot
python app.py
# Detach: Ctrl+B then D
```

---

## 🌐 Web Control Interface

The Flask server exposes the following API endpoints:

| Endpoint | Method | Action |
|----------|--------|--------|
| `/` | GET | Main control dashboard |
| `/move/forward` | POST | Move forward |
| `/move/backward` | POST | Move backward |
| `/move/left` | POST | Turn left |
| `/move/right` | POST | Turn right |
| `/move/stop` | POST | Stop all motors |
| `/sensors` | GET | Get live sensor JSON data |
| `/thermal` | GET | Get thermal image data |
| `/video_feed` | GET | Live MJPEG camera stream |

---

## 📊 Sensor Data

### Gas Sensor Thresholds (Reference)

| Sensor | Gas Detected | Safe Threshold | Danger Level |
|--------|-------------|----------------|-------------|
| MQ-2 | Methane / LPG | < 300 ppm | > 1000 ppm |
| MQ-135 | H₂S / NH₃ / CO₂ | < 50 ppm | > 200 ppm |
| MQ-136 | Hydrogen Sulfide | < 10 ppm | > 50 ppm |

> Sensor outputs are raw ADC values (0-1023). Calibration with known gas concentrations is recommended for precise ppm readings.

### Thermal Imaging

- **Sensor:** MLX90640 (32×24 pixel IR array)
- **Temperature Range:** -40°C to +300°C
- **Field of View:** 55° × 35°
- **Update Rate:** Up to 32 Hz
- Used to detect **pipe leaks, hot gas vents, blockages, and structural anomalies**

---

##  Future Improvements

- [ ] Autonomous navigation using ultrasonic/LiDAR sensors
- [ ] GPS/IMU-based position logging
- [ ] Cloud dashboard for remote monitoring (MQTT/Node-RED)
- [ ] Onboard data logging to SD card (CSV export)
- [ ] AI-based blockage/anomaly detection (TensorFlow Lite)
- [ ] Waterproofing for full sewer submersion
- [ ] Battery level monitoring and low-battery alerts
- [ ] Multi-robot coordination

---

##  Author

**Adit**-Student Developer, Kota, Rajasthan, India  
Built as part of a robotics innovation project.

---

## License

This project is licensed under the **MIT License** - see the [LICENSE](LICENSE) file for details.

---

<div align="center">

If you found this project useful, please consider giving it a star!

**MinerBot - Making Underground Inspection Safer, Smarter, and Accessible.**

</div>
