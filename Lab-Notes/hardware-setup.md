# Hardware Setup Overview

This page documents the hardware components used in my SDR-based cellular and NTN experimentation setup. The goal is to clearly list each major component, its purpose, and how it fits into the overall system.
---
The latest setup instructions for dgx-spark can be found here : https://nvlabs.github.io/sionna/rk/setup.html#your-first-call
## Compute Platform

### NVIDIA Jetson AGX Orin (64GB)

The NVIDIA Jetson AGX Orin is used as the primary compute platform for SDR-based cellular and NTN experimentation. It provides sufficient CPU, GPU, and memory resources to run OpenAirInterface (OAI), Sionna Research Kit workflows, and supporting Docker containers. The platform is well-suited for edge and research deployments that require GPU acceleration in a compact form factor.

While the AGX Orin includes **64 GB of internal storage**, this capacity is very limited for practical development. Source builds, container images, logs, and datasets can quickly exhaust the internal storage, making external storage necessary for sustained experimentation.



![Jetson AGX Orin](/assets/images/hardware/jetson-agx-orin.png)

---

## Storage

### NVMe SSD

An NVMe SSD was added to significantly expand storage capacity and improve I/O performance.

- **Model:** WD_BLACK SN770  
- **Capacity:** 2 TB  
 

The NVMe drive is used to store source code, Docker images, build artifacts, logs, and experimental data. Moving these workloads off the internal storage improves reliability and reduces build and runtime friction when working with large containers and repeated rebuilds.



![WD_BLACK SN770 NVMe SSD](/assets/images/hardware/wd-black-sn770.png)



## Software Defined Radio (SDR)

### Ettus Research B206 SDR
- Used as the primary SDR platform
- Supports wideband operation suitable for cellular and experimental wireless work
- Connected to the host system for baseband processing and RF experimentation



![B206 SDR front view](/assets/images/hardware/b206-front.png)


###  Cellular Modem
###  Quectel RM520N-GL

DigiKey Part: 2958-RM520NGLAA-M20-SGASA-ND

Manufacturer: Quectel Wireless Solutions Co., Ltd.

Manufacturer Part Number: RM520NGLAA-M20-SGASA

This modem is used for 5G NR experimentation and supports advanced cellular features required for modern network testing.



![Quectel RM520N-GL module](/assets/images/hardware/quectel-rm520.png)


###   M.2 Development Board
###  GL.iNet M.2 5G Development Board

Provides an M.2 interface for the Quectel RM520N-GL

Simplifies power delivery, USB connectivity, and antenna connections

Used as the primary development and integration platform for the modem

Product link:
https://store-us.gl-inet.com/products/m2-5g-development-board



![GL.iNet M.2 5G development board](/assets/images/hardware/m2-dev-board.png)



###   RF Power Splitters / Dividers
Power Divider (SMA, RoHS)

DigiKey Part: 3157-ZN4PD1-63-S+-ND

Manufacturer Part: ZN4PD1-63-S+

Used for RF signal splitting in the test setup

2-Way Power Splitter (500–5000 MHz)

DigiKey Part: 3157-ZN2PD2-50-S+-ND

Manufacturer Part: ZN2PD2-50-S+

Supports wideband RF operation across multiple GHz

These components are used to route RF signals between SDRs, antennas, and measurement equipment.


![RF power splitters](/assets/images/hardware/rf-splitters.png)

![RF power splitters](/assets/images/hardware/rf-splitter2.png)

###   SIM / USIM / ISIM Cards
sysmoISIM SJA5 (USIM / ISIM)

10-pack with ADM keys

Supports USIM and ISIM functionality

Used for cellular authentication and network testing

Product link:
https://shop.sysmocom.de/sysmoISIM-SJA5-9FV-SIM-USIM-ISIM-Card-10-pack-with-ADM-keys-9FV-chip/sysmoISIM-SJA5-9FV-10p-adm



![sysmoISIM cards](/assets/images/hardware/sysmoisim.png)


  Notes

All RF components were selected to support wideband cellular experimentation

This setup allows flexible testing across SDR-based and modem-based workflows

Images are included to visually document physical connections and hardware layout

                    ┌──────────────────────────┐
                    │        Host PC            │
                    │     (your laptop/desktop) │
                    └───────────┬──────────────┘
                                │ USB 3.0
                                │
                ┌───────────────▼────────────────┐
                │  Quectel RM520N-GL 5G Modem     │
                │  (M.2 + GL.iNet Dev Board)     │
                └───────┬───────┬───────┬────────┘
                        │       │       │
                      ANT0    ANT1    ANT2    ANT3
                        │       │       │
                        └───────┴───────┴─────────┐
                                                  │
                                     ┌────────────▼────────────┐
                                     │   4-Way RF Power Splitter │
                                     │   ZN4PD1-63-S+ (SMA)      │
                                     └────────────┬────────────┘
                                                  │
                                          ┌───────▼────────┐
                                          │  20 dB Attenuator│
                                          └───────┬────────┘
                                                  │
                                     ┌────────────▼────────────┐
                                     │   2-Way RF Splitter      │
                                     │   ZN2PD2-50-S+           │
                                     └───────┬────────┬────────┘
                                             │        │
                                           TX (SMA)  RX (SMA)
                                             │        │
                                  ┌──────────▼────────▼──────────┐
                                  │       USRP B206-mini          │
                                  └──────────┬────────────────────┘
                                             │ USB 3.0
                                             │
                              ┌──────────────▼──────────────┐
                              │       Jetson AGX Orin         │
                              │     (OAI gNB / AI workloads)  │
                              └──────────────────────────────┘


