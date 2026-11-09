# Smart Helmet: IoT Wear-Detection, Alcohol Interlock & Live Telemetry

A patent-pending embedded safety system integrated inside a motorcycle helmet. Combines physical wear confirmation, breath alcohol concentration (BrAC) sensing, and a fail-safe wireless ignition interlock with real-time GPS telemetry logging.

[![Patent Specification](https://img.shields.io/badge/Patent-Complete%20Specification%20(Form%202)-blue)]()
[![Hardware](https://img.shields.io/badge/Controller-ESP32%20%7C%20FreeRTOS-red)]()
[![Wireless](https://img.shields.io/badge/Interlock-433MHz%20RF%20%7C%20BLE-green)]()

![Smart Helmet Teardown Overview](full_internal_layout.jpeg)
*Figure 1: Full internal hardware layout showing perimeter contact switches, GPS module, power boost regulation, and chin-bar sensing harness.*

---

## 1. System Overview & Patent Claims

* **Patent Title:** Smart Helmet with Alcohol Detection and Ignition Locking System (Form 2, The Patents Act 1970).
* **Core Function:** Enforces a physical Two-Factor Safety Interlock before enabling motorcycle ignition:
  1. **Factor 1 (Wear Verification):** Physical head contact closure via distributed mechanical limit switches.
  2. **Factor 2 (Sobriety Verification):** Breath alcohol analysis below legal BAC limits via an active heating element sensor.
* **Failsafe Wireless Control:** Transmits encrypted cyclic authorization packets over 433 MHz RF / BLE to an isolated relay unit tapped into the motorcycle ignition coil/starter circuit.

---

## 2. Sensor Integration & Mechanical Architecture

| Chin-Bar Sensor Arrangement | Crown GPS & Peripheral Wiring |
| :---: | :---: |
| ![Chinbar Sensors](chinbar_sensors.jpeg) | ![ESP32 Harness](esp32_rf_gps_wiring.jpeg) |

*Figure 2: (Left) Ultrasonic facial-proximity transceiver integrated into the chin bar. (Right) Crown GPS module with skyward-oriented ceramic patch antenna.*

### Mechanical Wear Confirmation (Anti-Cheating)
* **The Vulnerability:** Optical or single-point switches can be easily fooled by placing an object or bag inside the helmet.
* **The Engineering Solution:** Deployed a series array of four micro roller lever switches distributed along the crown and occipital lobes. The motorcycle ignition relay energizes **only when head circumference compression closes all contact pairs simultaneously**, ensuring true human wear.

### Breath Alcohol Concentration (BrAC) Subsystem
| MQ-3 Breathalyzer Integration | Rear Status Indication |
| :---: | :---: |
| ![MQ3 Macro](mq3_sensor_integration.jpeg) | ![Status LED Bar](helmet_rear_led.jpeg) |

*Figure 3: (Left) MQ-3 gas sensor positioned at the mouth airstream path. (Right) High-visibility rear LED state-of-charge and interlock status indicator.*

* **Sensor Placement:** Mounted an MQ-3 SnO2 semiconductor sensor in the chin-bar cavity directly in front of the rider's airway.
* **False-Positive Mitigation:** To reject environmental interference (hand sanitizers, perfumes, volatile hydrocarbons), the firmware executes a baseline rolling-average calibration routine on boot, sampling breath spikes only after the wear switches register contact.

---

## 3. Power Architecture & Voltage Regulation

![Power Supply and Charging Subsystem](power_and_switches.jpeg)
*Figure 4: Lithium-ion pouch cell, BMS protection board, step-up boost converter, and mechanical switches.*

* **Energy Storage:** 3.7V high-capacity Li-ion cell integrated into the rear shell cavity with a dedicated USB charge management and over-discharge protection circuit.
* **Rail Isolation & Stepping:** The MQ-3 sensor internal heating coil draws steady-state current (~150mA at 5.0V), which degrades battery voltage rapidly and can cause brownouts on microcontrollers:
  * Integrated an onboard step-up boost converter delivering a stiff 5.0V regulated power rail for the sensor heater and RF transmitter.
  * 3.3V low-dropout regulation powers the ESP32 and GPS module, preventing ADC sampling jitter during telemetry bursts.

---

## 4. Telemetry Pipeline & Wireless Interlock Logic

```text
[Wear Switches CLOSED] ──┐
                         ├──> [ESP32 Decision Engine] ──> [433MHz / BLE] ──> [Motorcycle Relay: START ENABLED]
[Alcohol Level < LIMIT] ──┘                                      │
                                                                 └──> [Firebase RTDB / Live GPS Map]
