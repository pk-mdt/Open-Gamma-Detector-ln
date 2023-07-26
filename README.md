# Open Gamma Detector

**This is a revised hardware design from the original 3.0 version of the project. The project has been redesigned in Kicad and incorporates an impedance controlled 4 layer board design, as well as micro SMA connectors to the SiPM**

These changed from the original project are hardware only, the original software works and has been fully tested on these revisions.

Revised custom layout and hardware for a hackable scintillation counter and multichannel analyzer (MCA) all-in-one device using a popular NaI(Tl) scintillation crystal and a silicon photomultiplier (SiPM). Extremely affordable design for a DIY gamma spectroscopy setup with a total parts cost of under around 200 USD.

The detector uses a standard serial-over-USB connection so that it can be integrated into as many other projects as possible, for example data logging with a Raspberry Pi, weather stations, Arduino projects, etc.

If you want a barebone version of the Open Gamma Detector to count pulses or something like that, you can have a look at the [Mini SiPM Driver](https://github.com/OpenGammaProject/Mini-SiD) board.

## Features

Here are some of the most important specs:

* Compact design: Total size 77 x 50 mm.
* All-in-one: No external parts (e.g. sound card) required to record gamma spectra.
* Easily programmable using drag-and-drop firmware files or the standard Arduino IDE.
* Low-voltage device: No HV needed like with photomultiplier tubes.
* Can use SiPMs in the voltage range of 27.5 V to 33.8 V.
* 4096 ADC channels with built-in 3 V voltage reference.
* Revised Energy resolution of ~6% @ 662 keV possible; highly dependent on your SiPM/scintillator assembly.
* Default (Energy) Mode: About 15 µs total dead time while measuring energy.
* Geiger Mode: About 5 µs total dead time without energy measurements.
* Low power consumption: ~25 mA @ 5 V with default firmware.
* Additional broken-out power pins and I2C, SPI and UART headers for custom parts (e.g. display, µSD card, etc.).
* Built-in True Random Number Generator.
* Simple OLED support out of the box (SSD1306 and SH110x).
* Built-in customizable ticker support.

## Working Principle

Here is a nice flow chart to describe how the device roughly works:

![Working Principle Flow Chart](docs/flow.drawio.svg)

## Hardware

Hardware design has been done with Kicad and all the needed files for you to import the project as well as the schematic can be found in the `hardware` folder. There is also a Gerber file available for you to go directly to the PCB manufacturing step.

Detailed information about the hardware of the detector as well as the potentiometer settings and assembly can be found in the [hardware directory](/hardware/).

## Software

The software aims to be as simple as possible to understand and maintain. To achieve this I decided to use an off-the-shelf microcontroller - the [Raspberry Pi Pico](https://www.raspberrypi.com/products/raspberry-pi-pico/). This board can be programmed with the Arduino IDE over micro-USB and is powerful (dual core, fast ADC, plenty of memory, ...) enough for the purpose and also exceptionally cheap.

To program the Pico and/or play around with the firmware, head to the [software directory](/software/). There you will also find documentation on the serial interface (!), display support, and much more.

## Example Spectra

Coming Soon

## Known Limitations

1. The Raspberry Pi Pico's ADC has some pretty [severe DNL issues](https://pico-adc.markomo.me/INL-DNL/#dnl) that result in four channels being much more sensitive (wider input range) than the rest. For now the simplest solution was to discard all four of them, by printing a `0` when any of them comes up in the measurement (to not affect the cps readings). You can turn this behavior off by using the `set correction` command. This is by no means perfect or ideal, but it works for now until this gets fixed in a later hardware revision of the RP2040 (wish us luck!).

2. Due to the global parts shortage many chips are much harder to come by, if at all that is. Parts that are listed in the BOM should be available easily and with high reliability and stock so that they don't run out quickly. Please let me know if you cannot find a part anymore.

3. The power supply is currently **not** temperature corrected, meaning changes in the ambient temperature with a constant voltage affect the gain of the SiPM. This will naturally result in a different DC bias, energy range and S/N ratio. This effect is negligible around room temperature, though. The temperature dependence of the gain is -0.8%/°C (21°C) for the MicroFC SiPMs.

## Troubleshooting and FAQ

Please have a look at [REFERENCE.md](REFERENCE.md) for some general guidance.

If this doesn't help you, feel free to reach out and create an issue or open a discussions thread.

## Some Ideas

### Coincidence Measurements

Using multiple detector boards with some firmware modifications it should be theoretically possible to implement a coincidence measurement feature. By respectively connecting the `VSYS`, `GND` and one of the broken-out pins to each other on both boards you have everything you need to get started. The pins would be used for an interrupt from the child detector to the parent to trigger a pulse if both timings coincide.

At the moment, though, I couldn't get a coincidence mode feature running due to some weird timing issues. There might be a firmware update in the future to implement this feature. If you have any ideas, let me know!

### Cooling the SiPM

During operation all the electronics including the SiPM naturally heat up ever so slightly. Due to the detector board (where most of the power is dissipated) not being directly connected to the SiPM self-heating is negligible, though. Therefore air or water cooling alone won't improve performance considerably, because it won't heat up much above ambient temps, if at all that is. However, you could cool the SiPM PCB with a Peltier module to sub-ambient temperatures. According to the [datasheet AND9770 (Figure 27)](https://www.onsemi.com/pub/Collateral/AND9770-D.PDF) every 10°C reduction in temperature decreases the dark count rate by 50%! But be sure to correct the SiPM voltage (overvoltage) in this case as it also changes with temperature.

Note that the required breakdown voltage of the SiPM increases linearly with 21.5 mV/°C, see the [C-Series SiPM Sensors Datasheet](https://www.onsemi.com/pdf/datasheet/microc-series-d.pdf). This means that you would also need to temperature correct the PSU voltage if you wanted to use it with considerably different temperatures.

### Shielding Background Radiation

Shielding the ambient background can be done ideally using a wide enough layer of lead (bricks) all around the detector with a thin layer of lower-Z material on the inside such as copper to avoid backscattering. The SiPM and the sample can then be put into the structure to get the best measurements possible (low background).

See Wikipedia: [Lead Castle](https://en.wikipedia.org/w/index.php?title=Lead_castle&oldid=991799816)

---

Thanks for reading.
