# Jetson AGX Orin Setup

This page documents how I set up the NVIDIA Jetson AGX Orin and upgraded it from the factory-installed JetPack version to JetPack 6. It also highlights practical constraints around JetPack versions, host operating systems, and flashing workflows. For newer AI workloads, platforms such as Thor or DGX Spark may be better options.

## Hardware
- NVIDIA Jetson AGX Orin (64GB)
- Official power supply
- USB-C cable
- External display, keyboard, and mouse

## Host Machine
- Ubuntu 22.04
- NVIDIA SDK Manager

## Initial State
The Jetson AGX Orin shipped with JetPack 5 pre-installed from the factory. However, this version was not used for my workflow.

## Upgrade to JetPack 6
To align with newer CUDA support and updated system libraries, I upgraded the device directly to JetPack 6. This required a full re-flash of the system, as in-place upgrades between major JetPack versions are not supported.


## Issues Encountered
- JetPack 5 was not available in SDK Manager on Ubuntu 22.04
- Downgrading or switching JetPack versions using package managers is not supported
- Recovery mode provides no HDMI output, which can be confusing during flashing

## Resolution
The upgrade was completed by:
- Using Ubuntu 22.04 as the host OS
- Flashing the device using NVIDIA SDK Manager
- Placing the Jetson AGX Orin into Force Recovery Mode
- Performing a clean re-flash with JetPack 6

Once flashing completed, the system booted normally and all required components were installed successfully.

## Verification
After setup, I verified the installation using the following commands:



```bash
cat /etc/nv_tegra_release
# R36 (release), REVISION: 4.7, GCID: 42132812, BOARD: generic, EABI: aarch64,
# DATE: Thu Sep 18 22:54:44 UTC 2025
# KERNEL_VARIANT: oot
TARGET_USERSPACE_LIB_DIR=nvidia
TARGET_USERSPACE_LIB_DIR_PATH=usr/lib/aarch64-linux-gnu/nvidia

lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description: Ubuntu 22.04.5 LTS
Release: 22.04
Codename: jammy




