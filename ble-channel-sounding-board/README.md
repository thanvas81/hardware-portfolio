# BLE Channel Sounding Sensor Board

**Custom nRF54L15 board for sub-metre indoor positioning with Bluetooth Channel Sounding, plus environmental and motion sensing.**

Diploma thesis, School of Electrical and Computer Engineering, National Technical University of Athens (2025).
*Design of a Wireless Indoor Tracking System for Multi-Sensor Data Acquisition with BLE Channel Sounding*. Supervisor: Prof. E. Hristoforou.

<p align="center">
  <img src="images/pcb_3d_top.png" alt="PCB 3D render, top side" width="45%">
  &nbsp;
  <img src="images/pcb_3d_bottom.png" alt="PCB 3D render, bottom side" width="45%">
</p>

## Highlights

- **17 cm mean ranging error** across three anchors at 3–4.5 m, measured over a one-hour indoor capture
- **Bluetooth Channel Sounding** (Bluetooth 6.0) distance estimation using IFFT and phase-based ranging, instead of RSSI
- **4-layer, 38 × 25 mm PCB** designed in KiCad around the nRF54L15 SoC, with a continuous inner ground plane for RF performance
- **Multi-sensor**: 3-axis accelerometer + magnetometer (motion wake-up) and temperature / humidity / pressure
- **Portable power**: USB-C or single-cell Li-ion, with charger, power-path and buck-boost to a stable 3.3 V

## System overview

```mermaid
flowchart LR
    subgraph Tag["Tag (reflector) – this board"]
        S1["LSM303AH<br/>accel + magnetometer"] -->|SPI| MCU["nRF54L15<br/>(ME54BS01 module)"]
        S2["BME280<br/>temp / humidity / pressure"] -->|I²C| MCU
        PWR["USB-C / Li-ion<br/>charger + buck-boost"] --> MCU
    end
    subgraph Anchors["Anchors (initiators)"]
        A1["Initiator 1"]
        A2["Initiator 2"]
        A3["Initiator 3"]
    end
    MCU <-->|"BLE Channel Sounding"| A1 & A2 & A3
    MCU -->|"BLE advertising<br/>sensor data"| A1
    A1 & A2 & A3 -->|"UART → USB"| PC["Host PC<br/>logging & visualisation"]
```

## Hardware

| Block | Part | Notes |
|---|---|---|
| SoC / radio | Minew **ME54BS01** (Nordic **nRF54L15**) | BLE 6.0 Channel Sounding, integrated antenna |
| Motion | ST **LSM303AH** | 3-axis accelerometer + magnetometer on SPI, interrupt lines for motion wake-up |
| Environment | Bosch **BME280** | Temperature, humidity, pressure on I²C |
| Charger | Microchip **MCP73871** | Li-ion charging with power-path management and NTC input |
| Regulation | TI **TPS63031** buck-boost, TI **TLV70018** LDO | 3.3 V from 3.0–4.2 V battery or 5 V USB |
| USB | USB-C receptacle, Nexperia **PRTR5V0U2X** ESD, Microchip **MCP2221A** | USB-to-UART bridge for logging |
| Debug | Tag-Connect **TC2030** | SWD programming without a header |

Schematic is split into hierarchical sheets: power supply, charger, buck-boost, LDO, USB-C, USB-to-serial, temperature sensor, accelerometer/magnetometer.

**Files**

### PCB layers

4-layer stackup, 1.6 mm FR4: signals on the outer layers, a continuous ground plane on inner 1 for RF stability and clean return paths, and 3.3 V / 5 V power on inner 2.

| Top copper | Inner 1: ground plane |
|---|---|
| ![Top copper](images/layer_F_Cu.png) | ![Inner layer 1](images/layer_In1_Cu.png) |
| **Inner 2: power plane (3.3 V / 5 V)** | **Bottom copper** |
| ![Inner layer 2](images/layer_In2_Cu.png) | ![Bottom copper](images/layer_B_Cu.png) |

### Files

- [Schematic (PDF)](schematics/BLE_sensors_schematic.pdf)
- [Bill of materials](fabrication/BLE_sensors_BOM.xlsx)
- [Gerbers for fabrication](fabrication/BLE_sensors_gerbers.zip)
- [KiCad 9 project](kicad/): open `BLE_sensors.kicad_pro`

## Firmware

Written in C on **Zephyr RTOS / nRF Connect SDK**. Two roles:

**Tag (reflector)** – this board. Reads the sensors every 2 s, broadcasts them in non-connectable advertising, answers Channel Sounding requests, and sleeps when no motion is detected.

```mermaid
flowchart TD
    A["Start + init (BLE + sensors)"] --> B["Start advertising (CS reflector + data)"]
    B --> C["Every 2 s: read environment + battery"]
    C --> D["Send via non-connectable advertising"]
    B --> E["On connect → configure CS (reflector role)"]
    B --> G["Motion detected? → stay awake"]
    G --> H["No motion → sleep"]
    H --> I["Motion interrupt → wake & resume"]
```

**Anchor (initiator)**. Scans for the tag, connects, runs continuous Channel Sounding procedures, computes the distance and logs it over UART with the received sensor data.

```mermaid
flowchart TD
    A["Start → scan"] --> B{"Ranging peer?"}
    B -- no --> A
    B -- yes --> C["Connect → discover → CS capabilities"]
    C --> D["Configure CS + start continuous"]
    D --> E["Process ranging data"]
    E --> F["Log distance (2 s)"]
    F --> D
    C --> G["Subscribe custom data"]
    G --> H["Receive and log env/sensor data"]
    H --> G
```

## Results

Three initiators were mounted on the walls at 3.0 m, 4.0 m and 4.5 m from a stationary tag, and distances were logged for one hour.

| Anchor | Distance | Mean absolute error |
|---|---|---|
| Initiator 1 | 4.0 m | 0.142 m |
| Initiator 2 | 4.5 m | 0.177 m |
| Initiator 3 | 3.0 m | 0.193 m |
| **Overall** | | **0.171 m** |

<p align="center">
  <img src="images/test_setup.png" alt="Test setup: three wall-mounted initiators and a tag on the table" width="45%">
  &nbsp;
  <img src="images/ranging_error_heatmap.png" alt="Local error heat map around the target" width="45%">
</p>

The IFFT-based estimator gave the most stable distance estimates. Full method and measurements are in the thesis.

## Documents

- [Thesis (PDF)](docs/thesis.pdf)
- [Presentation slides](docs/thesis_presentation.pptx)
- [Flowcharts (Mermaid sources)](docs/flowcharts/)

## Folder layout

```
images/        3D renders, copper layers, test setup, results
schematics/    schematic PDF
kicad/         KiCad 9 project (schematic sheets + PCB)
fabrication/   Gerbers and BOM
docs/          thesis, slides, flowchart sources
```

---

[← Back to all projects](../README.md)
