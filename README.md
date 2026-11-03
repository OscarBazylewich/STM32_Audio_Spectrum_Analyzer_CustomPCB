# STM32 Audio Spectrum Analyzer - Custom PCB

A compact 4-layer KiCad board for my audio spectrum analyzer, built around an STM32L4Q5CGT6 with an on-board I2S microphone, a header for an ST7789 LCD module and USB-C power.

[![License: GPL-3.0](https://img.shields.io/badge/license-GPL--3.0-blue.svg)](LICENSE)
![MCU](https://img.shields.io/badge/MCU-STM32L4Q5CGT6-03234B?logo=stmicroelectronics&logoColor=white)
![KiCad](https://img.shields.io/badge/KiCad-10-314CB0?logo=kicad&logoColor=white)

<p align="center">
  <img width="800" alt="KiCad 3D render of the board front: the 2-inch LCD module outline on the left, and the STM32, crystal, BOOT0 and RESET buttons, Tag-Connect pads, LDO and USB-C connector on the right" src="https://github.com/user-attachments/assets/85aaed98-7ecc-47bb-b375-da99e446172a" />
  <br />
  <em>Front of the board in the KiCad 9.0 3D viewer, with all 3D models loaded.</em>
</p>

This project uses an STM32L4Q5CGT6 microcontroller to reduce PCB size, with the added benefit of having similar functionality to most of ST's STM32 performance-line MCUs that are relevant to this project. The original prototype was done on a Nucleo-F767ZI board (see [Related projects](#related-projects)). This board is designed to take power and USB data from USB-C, read audio from an on-board ICS-43434 I2S microphone and drive a Waveshare 2-inch ST7789 LCD module over SPI.

**Status:** design files only (schematic Revision A, dated 2026-03-31). There are no fabrication outputs, assembly data, test results or firmware for this board in this repo.

## Features

- STM32L4Q5CGT6: Arm Cortex-M4, up to 120 MHz, 1024 KB flash, 320 KB RAM, LQFP-48 package
- USB-C for 5 V power and USB 2.0 full-speed data, with ESD protection
- SPX3819 3.3 V LDO (500 mA) feeding a single +3V3 rail for the whole board
- On-board ICS-43434 24-bit I2S MEMS microphone wired to the SAI2 pins
- 1x8 header for a Waveshare 2-inch ST7789 LCD module over SPI1
- Tag-Connect TC2030 footprint for SWD, plus BOOT0 and RESET buttons

## Hardware

### Main parts

| Ref | Part | What it does |
| --- | --- | --- |
| U4 | STM32L4Q5CGT6 (LQFP-48) | Microcontroller |
| X1 | 16 MHz crystal (9 pF, ±10 ppm) | HSE clock |
| U1 | SPX3819M5-L-3-3 | 3.3 V, 500 mA LDO |
| J1 | USB-C receptacle, USB 2.0, 16 pins | Power and USB data |
| U2 | USBLC6-2SC6 | ESD protection on D+ / D- |
| MK1 | TDK InvenSense ICS-43434 | I2S MEMS microphone (bottom side) |
| U3 | 1x8 pin header, 2.54 mm | Plugs in the Waveshare 2-inch ST7789 LCD module |
| S1, S2 | Tactile switches | BOOT0 and RESET |
| J2 | Tag-Connect TC2030 footprint | SWD, SWO, NRST and 3.3 V (pads and locating holes only, not in the BOM) |

Every BOM part has its manufacturer, MPN and LCSC part number recorded in the schematic.

### Pinout

| Interface | Signal | MCU pin | Connects to |
| --- | --- | --- | --- |
| Display | DIN | PA7 (SPI1_MOSI) | U3 pin 3 |
| Display | CLK | PA5 (SPI1_SCK) | U3 pin 4 |
| Display | CS | PA3 (GPIO) | U3 pin 5 |
| Display | DC | PA2 (GPIO) | U3 pin 6 |
| Display | RST | PA1 (GPIO) | U3 pin 7 |
| Display | BL | PA0 (GPIO) | U3 pin 8 |
| Microphone | WS | PB12 (SAI2_FS_A) | MK1 pin 1 |
| Microphone | SCK | PB13 (SAI2_SCK_A) | MK1 pin 4 |
| Microphone | SD | PB15 (SAI2_SD_A) | MK1 pin 6 |
| USB | D- | PA11 (USB_OTG_FS_DM) | J1 through U2 |
| USB | D+ | PA12 (USB_OTG_FS_DP) | J1 through U2 |
| SWD | SWDIO | PA13 | J2 pin 2 |
| SWD | RESET | NRST | J2 pin 3 |
| SWD | SWCLK | PA14 | J2 pin 4 |
| SWD | SWO | PB3 | J2 pin 6 |

Power pins: U3 pin 1 and J2 pin 1 are +3V3, U3 pin 2 and J2 pin 5 are GND. On the mic, pin 5 is VDD (+3V3), pin 3 is GND and LR (pin 2) is tied to GND. The schematic only names the interface (SPI or SAI), not the peripheral, so the peripheral names in brackets come from the KiCad STM32L4Q5CGTx symbol.

### Board

- 94.5 x 39.7 mm with 1 mm rounded corners and four M3 mounting holes
- 4 layers: signals on top, two inner GND planes, signals plus a +3V3 pour on the bottom
- About 1.6 mm thick, HAL lead-free finish, green solder mask, white silkscreen
- All parts on the top side except the microphone, which is on the bottom

## Circuit

- **Power path.** VBUS from USB-C goes through a 2.2 µH series inductor (L1), with a 2.2 µF capacitor on each side, into the SPX3819 LDO. The LDO's EN pin is tied to its input, so it's on whenever USB power is present. One +3V3 rail supplies everything: VDD, VDDA, VDDUSB, the display and the microphone. VBAT is also tied to +3V3, so there's no backup battery.
- **USB.** 5.1 kΩ pull-downs on CC1 and CC2 mark the board as a USB sink. D+ and D- pass through the USBLC6-2SC6 to PA12 and PA11. The SBU pins aren't connected.
- **Clock.** A 16 MHz crystal on PH0/PH1 with 10 pF C0G load capacitors. Its footprint has a front-copper keep-out area that blocks copper pour under the crystal. There's no 32.768 kHz LSE crystal (PC14/PC15 are unconnected).
- **Microphone.** The ICS-43434 sits on the bottom side, and its footprint has a 0.5 mm acoustic port hole through the board. WS, SCK and SD go to PB12, PB13 and PB15.
- **Display.** The display uses a pin header footprint so an LCD module that I already have (the Waveshare 2-inch) can plug straight in. The PCB has a silkscreen outline marked "2inch LCD Module" and four 3.2 mm holes for the module's mounting holes. That module takes up the left two-thirds of the board, and the electronics fit in the right-hand third.
- **Boot and reset.** S1 connects BOOT0 (PH3) to +3V3, and a 10 kΩ resistor holds it low otherwise. S2 pulls NRST to GND. Both lines have a 100 nF filter capacitor.

## Getting started

1. Install [KiCad](https://www.kicad.org/) 10.0 or newer. The files use the KiCad 10 format, so KiCad 9 and earlier can't open them.
2. Clone the repo:

   ```sh
   git clone https://github.com/OscarBazylewich/STM32_Audio_Spectrum_Analyzer_CustomPCB.git
   ```

3. Open `STM32L4Q5CGT-AudioSpectrumAnalyzer.kicad_pro` in KiCad 10. The symbols and footprints are embedded, so it opens without any extra libraries.
4. Optional, for the full 3D view (**View > 3D Viewer**): install the [JLCPCB KiCad Library](https://github.com/CDFER/JLCPCB-Kicad-Library) from the Plugin and Content Manager. The models for X1, S1, S2, L1 and C1 are found through `${KICAD8_3RD_PARTY}`. If those parts show up without bodies, point that variable (**Preferences > Configure Paths**) at the folder that contains `3dmodels/com_github_CDFER_JLCPCB-Kicad-Library`.
5. Run the ERC in the schematic and the DRC on the board before generating outputs.
6. In the PCB Editor, go to **File > Fabrication Outputs > Gerbers**, pick the copper, mask, paste, silkscreen and Edge.Cuts layers, then click **Generate Drill Files**. The saved plot settings write to the project folder, so I'd set an output folder like `gerbers/` first.

Or from the command line:

```sh
kicad-cli pcb export gerbers --output gerbers/ STM32L4Q5CGT-AudioSpectrumAnalyzer.kicad_pcb
kicad-cli pcb export drill --output gerbers/ STM32L4Q5CGT-AudioSpectrumAnalyzer.kicad_pcb
```

## Repository structure

```text
.
├── STM32L4Q5CGT-AudioSpectrumAnalyzer.kicad_pro   # KiCad 10 project: design rules, net classes, BOM fields
├── STM32L4Q5CGT-AudioSpectrumAnalyzer.kicad_sch   # Single-sheet schematic (Revision A)
├── STM32L4Q5CGT-AudioSpectrumAnalyzer.kicad_pcb   # 4-layer PCB layout, 94.5 x 39.7 mm
├── .gitattributes                                  # Lets GitHub count KiCad files in its language stats
├── LICENSE                                         # GNU GPL v3.0
└── README.md
```

## Gallery

The KiCad 9.0 renders include all 3D models but have square board corners. The KiCad 10.0 renders don't include all 3D models (the crystal, buttons, L1 and C1 are missing), but they have the added ground pour and the rounded corners of the current board outline.

<table>
  <tr>
    <th>KiCad 9.0: front</th>
    <th>KiCad 9.0: back</th>
  </tr>
  <tr>
    <td><img width="400" alt="KiCad 9.0 render of the board front with all component models" src="https://github.com/user-attachments/assets/85aaed98-7ecc-47bb-b375-da99e446172a" /></td>
    <td><img width="400" alt="KiCad 9.0 render of the board back with the ICS-43434 microphone, LCD header pins and mounting holes" src="https://github.com/user-attachments/assets/d1b51f1c-da5a-4e81-a80d-72842b435e69" /></td>
  </tr>
  <tr>
    <th>KiCad 10.0: front</th>
    <th>KiCad 10.0: back</th>
  </tr>
  <tr>
    <td><img width="400" alt="KiCad 10.0 render of the board front with rounded corners; the crystal, buttons, inductor and tantalum capacitor models are missing" src="https://github.com/user-attachments/assets/2df19aa7-58f1-4832-af66-1122759ae797" /></td>
    <td><img width="400" alt="KiCad 10.0 render of the board back with the ICS-43434 microphone and LCD header pins" src="https://github.com/user-attachments/assets/eb0d8f54-d216-4bad-933a-f12b7d15c262" /></td>
  </tr>
</table>

## Related projects

- [STM32_Audio_Spectrum_Analyzer_ST7789_and_INMP441](https://github.com/OscarBazylewich/STM32_Audio_Spectrum_Analyzer_ST7789_and_INMP441): the original prototype on a Nucleo-F767ZI with an ST7789 display and an INMP441 microphone, including its firmware.

## Credits

- Design by Oscar Bazylewich.
- The footprints and 3D models for X1, S1, S2, L1 and C1, and the symbols for X1, S1 and S2, come from the [JLCPCB KiCad Library](https://github.com/CDFER/JLCPCB-Kicad-Library) by CDFER. The Waveshare LCD module symbol and board outline come from a custom library. Everything else uses the standard KiCad libraries.

## License

Released under the GNU General Public License v3.0. See [LICENSE](LICENSE).
