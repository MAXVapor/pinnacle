# Pinnacle
Open Source custom firmware alternative for the Puffco® Peak vaporizer.

With the huge third-party accessory market manufacturing replacement buckets from Quartz, Silicon Carbide and other materials the limitations of the 4 built-in temperature modes are immediately apparent.

The goal of this project is to produce an Open Source replacement firmware for the device which first establishes feature parity with the Original Firmware, followed by integrating new features requested by the community such as user-configurable Temperature Setpoints, Auto-Off, Bluetooth and LED Control.

**This project is currently intended for Developers Only.**

## Currently Implemented:

##### LED Driver
- Color Format Handler
- Blink, Breathe and FadeOut functions

##### Peripherals Driver
- Wake from Sleep (USB and Button)
- Button Debouncing
- Button Single Click, Double Click, Triple Click Detection
- Button Long Hold Detection
- Haptic Feedback (Short and Long Vibration)

##### Battery Driver
- Input Supply Detection
- Battery Charging Enable / Disable
- Charging Inductor Enable / Disable
- Battery Temperature ADC
- Battery Voltage ADC

##### UART Driver
- Serial Output for debugging
- nRF51822 BLE Driver

##### Atomizer Driver
- Read Current Shunt ADC
- Calculate Atomizer Resistance
- Calculate Atomizer Temperature

## To Do:
- Refactor SK6182 LED Driver
- Refactor ADC code to use DMA
- PID Loop / Temperature Control

### In Progress - Temperature Control
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


### Disclaimer:
The firmware is distributed in the hope that it will be useful, but without any warranty. It is provided "as is" without warranty of any kind, either expressed or implied, including, but not limited to, the implied warranties of merchantability and fitness for a particular purpose. The entire risk as to the quality and performance of the firmware is with you.

All product and company names are trademarks™ or registered® trademarks of their respective holders. Use of them does not imply any affiliation with or endorsement by them.