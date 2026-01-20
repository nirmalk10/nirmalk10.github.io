# Hardware Setup Overview

This page documents the hardware components used in my SDR-based cellular and NTN experimentation setup. The goal is to clearly list each major component, its purpose, and how it fits into the overall system.

---

## Software Defined Radio (SDR)

### Ettus Research B206 SDR
- Used as the primary SDR platform
- Supports wideband operation suitable for cellular and experimental wireless work
- Connected to the host system for baseband processing and RF experimentation

**Image placeholder:**

![B206 SDR front view](/assets/images/hardware/b206-front.jpg)


Cellular Modem
Quectel RM520N-GL

DigiKey Part: 2958-RM520NGLAA-M20-SGASA-ND

Manufacturer: Quectel Wireless Solutions Co., Ltd.

Manufacturer Part Number: RM520NGLAA-M20-SGASA

This modem is used for 5G NR experimentation and supports advanced cellular features required for modern network testing.

Image placeholder:

![Quectel RM520N-GL module](/assets/images/hardware/quectel-rm520.jpg)


  M.2 Development Board
GL.iNet M.2 5G Development Board

Provides an M.2 interface for the Quectel RM520N-GL

Simplifies power delivery, USB connectivity, and antenna connections

Used as the primary development and integration platform for the modem

Product link:
https://store-us.gl-inet.com/products/m2-5g-development-board

Image placeholder:

![GL.iNet M.2 5G development board](/assets/images/hardware/m2-dev-board.jpg)



  RF Power Splitters / Dividers
Power Divider (SMA, RoHS)

DigiKey Part: 3157-ZN4PD1-63-S+-ND

Manufacturer Part: ZN4PD1-63-S+

Used for RF signal splitting in the test setup

2-Way Power Splitter (500–5000 MHz)

DigiKey Part: 3157-ZN2PD2-50-S+-ND

Manufacturer Part: ZN2PD2-50-S+

Supports wideband RF operation across multiple GHz

These components are used to route RF signals between SDRs, antennas, and measurement equipment.

Image placeholder:

![RF power splitters](/assets/images/hardware/rf-splitters.jpg)



  SIM / USIM / ISIM Cards
sysmoISIM SJA5 (USIM / ISIM)

10-pack with ADM keys

Supports USIM and ISIM functionality

Used for cellular authentication and network testing

Product link:
https://shop.sysmocom.de/sysmoISIM-SJA5-9FV-SIM-USIM-ISIM-Card-10-pack-with-ADM-keys-9FV-chip/sysmoISIM-SJA5-9FV-10p-adm

Image placeholder:

![sysmoISIM cards](/assets/images/hardware/sysmoisim.jpg)


  Notes

All RF components were selected to support wideband cellular experimentation

This setup allows flexible testing across SDR-based and modem-based workflows

Images are included to visually document physical connections and hardware layout



