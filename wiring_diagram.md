# Detailed Wiring Reference

## Raspberry Pi Zero 2W → L298N Motor Driver

| RPi Pin (BCM) | Physical Pin | L298N Pin | Function |
|--------------|-------------|-----------|---------|
| GPIO 12 | Pin 32 | ENA | Left Motor Speed (PWM) |
| GPIO 17 | Pin 11 | IN1 | Left Motor Direction 1 |
| GPIO 18 | Pin 12 | IN2 | Left Motor Direction 2 |
| GPIO 13 | Pin 33 | ENB | Right Motor Speed (PWM) |
| GPIO 27 | Pin 13 | IN3 | Right Motor Direction 1 |
| GPIO 22 | Pin 15 | IN4 | Right Motor Direction 2 |
| GND | Pin 6 | GND | Common Ground |

### L298N → Motors

    L298N OUT1 ──────────────────────┐
                                      ├──► Front-Left TT Motor (+/-)
    L298N OUT2 ──────────────────────┘

    L298N OUT3 ──────────────────────┐
                                      ├──► Front-Right TT Motor (+/-)
    L298N OUT4 ──────────────────────┘

> Rear motors are wired in **parallel** with their respective front motors.
> Swap motor wires if a motor spins in the wrong direction.

---

## Raspberry Pi Zero 2W → Arduino UNO

| RPi Connection | Target Port | Function |
|---------------|-------------|---------|
| Micro-USB (Data Port) | Arduino USB-B | Serial Comm & 5V Power to Arduino |

---

## Arduino UNO → Sensors & Servo

| Arduino Pin | Target Component | Target Pin | Function |
|------------|------------------|------------|---------|
| Analog A0 | MQ-2 Sensor | AOUT | Analog Gas Level 1 |
| Analog A1 | MQ-135 Sensor | AOUT | Analog Gas Level 2 |
| Analog A2 | MQ-136 Sensor | AOUT | Analog Gas Level 3 |
| Digital 4 | HC-SR04 Ultrasonic| TRIG | Trigger Pulse |
| Digital 5 | HC-SR04 Ultrasonic| ECHO | Distance Echo Receive |
| Digital 9 (PWM) | Servo Motor | Signal | Positional Control |

### Arduino Routing

    Arduino A0 ──────────────────────► MQ-2 AOUT
    Arduino A1 ──────────────────────► MQ-135 AOUT
    Arduino A2 ──────────────────────► MQ-136 AOUT

    Arduino D4 ──────────────────────► HC-SR04 TRIG
    Arduino D5 ◄────────────────────── HC-SR04 ECHO

    Arduino D9 ──────────────────────► Servo Signal (Yellow/Orange)

---

## Power Rail Summary

Since the buck converter is removed, you must carefully separate the high-current motor power from your sensitive 5V logic boards to prevent system crashes.

| Rail | Source | Powers |
|------|--------|--------|
| **V-Motor (7.4V/11.1V)** | Lithium-ion Battery | L298N 12V Terminal (Motor Power) |
| **5V (Main)** | 5V Power Bank / Adapter | RPi Zero 2W (via PWR IN Micro-USB port) |
| **5V (USB)** | RPi USB Data Port | Arduino UNO (via blue USB cable) |
| **5V (Sensors)** | Arduino 5V Pin | All 3 MQ Sensors (VCC), HC-SR04 (VCC), Servo (Red wire) |

> **Crucial Power Notes:**
> * **Do NOT** power the Raspberry Pi directly from the L298N's 5V output. The L298N cannot supply enough current for the Pi, Arduino, and 3 MQ gas sensors (which have power-hungry internal heaters). Use a dedicated 5V power bank for the Pi's "PWR IN" port.
> * **Common Ground:** The ground wire from the Li-ion battery, the L298N, the Arduino, and the Raspberry Pi MUST all be connected together. If grounds are not shared, the motor control signals will not work.
