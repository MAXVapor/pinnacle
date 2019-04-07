# Pinnacle
Open Source custom firmware alternative for the Puffco® Peak vaporizer.

With the huge third-party accessory market manufacturing replacement buckets from Quartz, Silicon Carbide and other materials the limitations of the 4 built-in temperature modes are immediately apparent.

The goal of this project is to produce an Open Source replacement firmware for the device which first establishes feature parity with the Original Firmware, followed by integrating new features requested by the community such as user-configurable Temperature Setpoints, Auto-Off, Bluetooth and LED Control.

**This project is currently intended for Developers Only.**

## Currently Implemented:
- Front Button
- Battery Voltage ADC
- USB Cable Detect
- Battery Charging
- Input Supply Detection
- Atomizer Heat PWM
- Power Inductors
- Current Shunt ADC / Atomizer Temperature
- Battery Temperature ADC
- LED Control
- Serial Port
- Vibration Motor
- SWD Debug
- Test Point

## To Do:
- Refactor ADC code to use DMA
- Refactor LED code to use DMA
- Basic Function Boilerplate
- PID Loop
- nRF51822 BLE Driver
- Phone App for BLE Controls
- Temperature Profiles

## Stock LED Indicators
- Flashing Rainbow -> No Atomizer
- Flashing Red 5x -> Atomizer Short
- Solid Red -> NTC Thermistor High Temp (Battery Overheat)
- Battery Voltage -> Green (+7.5v), Amber (7.4v-7.2v), Red (7.1v-3.4v)

## Custom LED Indicators
- Solid Blue -> Reset Atomizer Resistance

## Temperature Control
Either we have decent temperature accuracy, or a long string of coincidences and magic numbers in our math just happened to arrive us within 3C of the measured temperature.

If you are a smart person, feel free to check our calculations on this one.

We are making a few assumptions to determine the Atomizer Temperature:

* No Ambient Temperature Sensor, we use Battery Temp and hope its cold
* Room Temperature ADC Value: ~1080
* 3.3v vRef / 12 Bit ADC (4096)
* 5v Output
* 10x Current Shunt OpAmp
* Tungsten Coil (TCR .0045)
* Coil Resistance Should be ~0.6 Ohms
* The code below gives us 0.574 Ohms

![Curent Pinout](thermal_tc.png)

```
double coil_current = 0;
double coil_resistance = 0;
double coil_cold = 0;
coil_current = (adc1 * 3.3) / 4096.0;
coil_current = coil_current * 10.0;
coil_resistance = (5.0 / coil_current);

if(coil_cold <= 0 && adc1 < 1100) {
	coil_cold = coil_resistance;
} 

// TCR Tungsten? (TCR 0.0045)
coil_temp = (coil_resistance - coil_cold) / (0.0045 * coil_cold);
coil_temp = (coil_temp + batt_temp);
```

## MCU and Pinout:
The device is controlled by a ST Microelectronics STM32F030 Microprocessor
This graphic represents the hardware pinout of known peripherals.

![Curent Pinout](stm32f_pinout.png)

* PA0 / BUTTON [INPUT] - Front Panel Button
* PA1 / BATV [ANALOG] - Battery Voltage
* PA2 / CBLDET [INPUT] - Indicates USB Cable Presence
* PA3 / CHGOK [INPUT] - Logic Low indicates Charging Complete
* PA4 / ACOK [INPUT] - Logic Low indicates Valid Input Power Supply
* PA5 / CHGEN [OUTPUT] - Logic Low enables Battery Charging
* PA6 / HEAT [OUTPUT] - PWM Output to Atomizer MOSFET
* PA7 / IND1 [OUTPUT] - Enables Inductor
* PB0 / SHUNT [ANALOG] - Monitors Atomizer Current (10x Gain), must send Heat pulse
* PB1 / NTCT [ANALOG] - NTC Thermistor monitors Battery Pack Temperature
* PA8 / LED [OUTPUT] - Controls onboard LEDs
* PA9 / USART1_TX [UART] - Serial Port Transmit
* PA10 / USART1_RX [UART] - Serial Port Receive
* PA11 / MOTOR [OUTPUT] - Haptic Feedback / Vibration Motor
* PA12 / IND2 [OUTPUT] - Enables Inductor, required for Battery Voltage Monitoring
* PA13 / SWDIO [SWD] - STLink IO
* PA14 / SWCLK [SWD] - STLink Clock
* PA15 / TEST [OUTPUT] - Test Point on board

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
This document assumes you are running on Mac OS or Linux, if you need to setup on Windows, feel free to submit contributions to this document.

You will need a Python3 environment and to pip install pyusb or libusb. We also assume you have a basic environment setup with the normal tools: homebrew, git, etc.

If you wish to do normal debugging and set breakpoints, etc. This document will assume you have VSCode installed.

Make sure your device is connected to the ST-Link with the custom USB to SWD Cable.

### Backup OFW (Original Firmware)
You should take a dump of your original, unmodified device firmware before you go any further and keep it in a safe place.

- Verify MCU Connectivity

`pystlink -i` should return output similar to this:

```
DEVICE: ST-Link/V2 V2J33S7
SUPPLY: 3.24V
CORE:   CortexM0
MCU:    STM32F030x6/STM32F031x6/STM32F038x6
FLASH:  32KB
SRAM:   4KB-->
```

Pull down a copy of [PySTLink](https://github.com/pavelrevak/pystlink.git).

- Backup Device Flash: `pystlink read:flash:flash.bin`

- Backup Device SRAM: `pystlink read:sram:sram.bin`

If you ever wish to return your device to its original factory state, you will need to reflash this file back to the device and we cannot help you if you lose it.

### Compiler

You can install GCC for ARM, an STLink utility and a GDB debugger with these Homebrew commands

`brew tap nitsky/stm32`

`brew install arm-none-eabi-gcc`

`brew install --HEAD stlink`

### Debugging

If you wan't to open your device and wire up a TTL to Serial converter, you can send printf statements to the serial port.

But since we already have an STLink connection and we plan to use the serial port for the Bluetooth module / some people might not want to open their device, we can setup the real debugger.

- Section In Progress

### Flashing Firmware

You can erase and reprogram the STM32F Flash with a single command.

`pystlink flash:erase flash:file_to_flash.bin`

You will see output such as:

```
DEVICE: ST-Link/V2 V2J33S7
SUPPLY: 3.24V
CORE:   CortexM0
MCU:    STM32F030x6/STM32F031x6/STM32F038x6
FLASH:  32KB
SRAM:   4KB
Erasing FLASH: [========================================] done in 0.10s
Loaded 32768 Bytes from file_to_flash.bin file
Writing FLASH: [========================================] done in 1.57s
```

### Disclaimer:
The firmware is distributed in the hope that it will be useful, but without any warranty. It is provided "as is" without warranty of any kind, either expressed or implied, including, but not limited to, the implied warranties of merchantability and fitness for a particular purpose. The entire risk as to the quality and performance of the firmware is with you.

All product and company names are trademarks™ or registered® trademarks of their respective holders. Use of them does not imply any affiliation with or endorsement by them.
