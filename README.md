# Pinnacle
Open Source custom firmware replacement for the Puffco® Peak vaporizer.

The stock firmware of the device has several limitations, the most obvious of which are non-configurable Temperature Settings.

With the huge third-party accessory market manufacturing replacement buckets from Quartz, Silicon Carbide and other materials the limitations of the 4 built-in temperature modes are immediately apparent.

The goal of this project is to produce an Open Source replacement firmware for the device which first establishes feature parity with the Original Firmware, followed by integrating new features requested by the community such as user-configurable Temperature Setpoints and LED Control.


## Programming Hardware:
The manufacturer has made it quite easy to start hacking on the device, and it does not require you to take apart your device to download or upload code.

The "Micro USB" port on the back of the unit provides a charging port for the battery, but does not actually expose a USB interface.

With a little bit of knowledge of the board layout and typical STM32F configurations, we were able to determine the D+ and D- lines on the Micro-USB port are actually a SWD (Serial Wire Debug) interface to the processor.

To make the custom programming cable, you will need the following items:

- ST-Link V2 (Can be found on eBay for about $5 USD)
- Micro-USB Cable

You will need to cut the Micro-USB cable and strip the wires to attach them to the ST-Link. We recommend attaching Dupont pin headers.

If you have more patience than I do, you can order a "Micro USB to dupont" cable on ebay and wait for the slow shipping.

- STLink 5v -> USB Red Wire
- STLink GND -> USB Black Wire
- STLink SWD IO -> USB Green Wire
- STLink SWD CLK -> USB White Wire


## Development Toolchain Setup

###PySTLink Software
- In Progress

###Save Your Original Firmware
- In Progress

###ST32MCubeMX Project
- In Progress

###Compiler
- In Progress

###Flashing Custom Firmware
- In Progress



## MCU and Pinout:
The device is controlled by a ST Microelectronics STM32F030 Microprocessor

- Pinout: In Progress

## Currently Implemented:
- Vibration Motor Control
- Power Button Control
- LED Control

## To Do:
- NTC Battery Temperature Sensor
- Battery Voltage and Charge Detection
- Heat Output Control

## Planned Features:
- Optional BLE Module via On-Board Serial Port

### Disclaimer:
The firmware is distributed in the hope that it will be useful, but without any warranty. It is provided "as is" without warranty of any kind, either expressed or implied, including, but not limited to, the implied warranties of merchantability and fitness for a particular purpose. The entire risk as to the quality and performance of the firmware is with you.

All product and company names are trademarks™ or registered® trademarks of their respective holders. Use of them does not imply any affiliation with or endorsement by them.
