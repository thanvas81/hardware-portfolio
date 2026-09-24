# Handheld High-Irradiance LED Controller

**Battery-powered, USB-C PD charged controller for a high-power red LED array, with Bluetooth control, per-string constant-current drive and active thermal management.**

Freelance hardware design (2026), in Altium Designer. Three boards: main controller, hexagonal LED panel, IR temperature sensor.

<p align="center">
  <img src="images/main_board_top.png" alt="Main board, 3D render, top" width="48%">
  &nbsp;
  <img src="images/main_board_bottom.png" alt="Main board, 3D render, bottom" width="48%">
</p>

## Highlights

- **USB-C Power Delivery** input with a PD sink controller, charging a **2S Li-ion** pack through a buck-boost charger with power-path, so the device runs while it charges
- **4-switch buck-boost** controller generating the high-power LED rail from the battery
- **7 constant-current LED channels**, one driver per LED string to avoid current hogging, with PWM dimming
- **nRF52840** Bluetooth LE module for control, settings and logging
- **Thermal management**: non-contact IR temperature sensing plus a PWM fan with tach feedback
- **Protection**: reverse-polarity / over-voltage load disconnect, TVS, fuse

## System overview

```mermaid
flowchart LR
    USB["USB-C"] --> PD["PD sink<br/>TPS25730"]
    PD --> CHG["2S charger + power-path<br/>BQ25792"]
    CHG <--> BAT[("2S Li-ion")]
    CHG --> PROT["Reverse-polarity / OVP<br/>LM74502"]
    PROT --> BB["4-switch buck-boost<br/>LM5175"]
    BB --> DRV["7× constant-current<br/>LED drivers AL8843"]
    DRV --> PANELS["7× LED panels<br/>(7 LEDs each)"]
    CHG --> BUCK["3.3 V buck<br/>AP62250"]
    BUCK --> MCU["nRF52840 BLE module"]
    MCU -. "PWM dimming" .-> DRV
    MCU -. "I²C" .-> CHG
    MCU -. "PWM + tach" .-> FAN["Fan"]
    IR["IR temperature sensor board<br/>MLX90632"] -. "I²C" .-> MCU
```

## Main controller board

| Block | Part |
|---|---|
| USB-C input | 24-pin USB-C receptacle, **TPS25730** USB PD sink controller |
| Battery | **BQ25792**: I²C-controlled 1–4 cell buck-boost charger with power-path, 2S configuration |
| Protection | **LM74502** reverse-polarity and over-voltage controller with load disconnect, TVS, board-mount fuse |
| LED power rail | **LM5175** 4-switch synchronous buck-boost controller with external 60 V MOSFETs |
| LED drivers | 7× **AL8843** constant-current buck drivers, one per LED string, FPC outputs to the panels |
| Logic supply | **AP62250** 3.3 V buck |
| MCU / radio | Minew **MS88SF** module (Nordic **nRF52840**), Tag-Connect programming |
| Thermal | Fan output with PWM control and tach input; header for the IR sensor board |

Hierarchical schematic: USB-C, PD, charger, protection switch, LED power supply, 3.3 V buck, MCU, and one LED-driver sheet instantiated 7 times.

## LED panel

<img src="images/led_panel.png" alt="Hexagonal LED panel" width="300" align="right">

Hexagonal board carrying **7 high-power red LEDs** (OSRAM) in series, on a hexagonal lattice so that several panels tile a round illumination area. Connected to its driver channel through an FPC.

<br clear="right">

## IR temperature sensor

<p>
  <img src="images/ir_sensor_top.png" alt="IR sensor board, top" width="150">
  <img src="images/ir_sensor_bottom.png" alt="IR sensor board, bottom" width="150">
</p>

Small board with a **Melexis MLX90632** non-contact infrared temperature sensor, connected to the main board over an FPC (I²C). Used for temperature monitoring alongside the fan control.

## My role

System architecture, component selection, power-path design, schematics and PCB layout for all three boards in Altium Designer.

## Schematics

- [Main controller board (PDF)](schematics/main_board_schematic.pdf)
- [LED panel (PDF)](schematics/led_panel_schematic.pdf)
- [IR sensor board (PDF)](schematics/ir_sensor_schematic.pdf)

---

[← Back to all projects](../README.md)
