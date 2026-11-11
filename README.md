# STM32 Audio Spectrum Analyzer - Custom PCB

This repository contains a small KiCad board (4 layers) for an audio spectrum analyzer. The board has an STM32L4Q5CGT6, an I2S microphone, a pin header for a Waveshare 2inch LCD Module (ST7789) and USB-C power.

[![License: GPL-3.0](https://img.shields.io/badge/license-GPL--3.0-blue.svg)](LICENSE)
![MCU](https://img.shields.io/badge/MCU-STM32L4Q5CGT6-03234B?logo=stmicroelectronics&logoColor=white)
![KiCad](https://img.shields.io/badge/KiCad-10-314CB0?logo=kicad&logoColor=white)

<p align="center">
  <img width="800" alt="KiCad 3D render of the top side of the board. The outline of the Waveshare 2inch LCD Module is on the left side of the board. The MCU, the crystal, the BOOT0 and RESET switches, the Tag-Connect pads, the LDO and the USB-C receptacle are on the right side." src="https://github.com/user-attachments/assets/85aaed98-7ecc-47bb-b375-da99e446172a" />
</p>

This project uses an STM32L4Q5CGT6 microcontroller (MCU) to decrease the dimensions of the board. This MCU also has almost the same functions as most STM32 performance-line MCUs from ST that are applicable to this project. The initial prototype used a Nucleo-F767ZI board (refer to [Related projects](#related-projects)).

The board has these functions:

- It receives power and USB data through the USB-C receptacle.
- It reads audio from an ICS-43434 I2S microphone on the board.
- It controls the LCD module with SPI.

**Note:** This repository contains only project files (schematic Revision A, date 2026-03-31). It does not contain fabrication outputs, assembly data, test results or firmware for this board.

## Features

- STM32L4Q5CGT6: Arm Cortex-M4, a maximum of 120 MHz, 1024 KB flash, 320 KB RAM, LQFP-48 package
- USB-C for 5 V power and USB 2.0 full-speed data, with ESD protection
- SPX3819 3.3 V LDO (500 mA) that supplies one +3V3 rail for all the parts on the board
- ICS-43434 MEMS microphone (I2S, 24-bit) on the board, connected to the SAI2 pins
- 1x8 pin header for a Waveshare 2inch LCD Module (ST7789) on SPI1
- Tag-Connect TC2030 footprint for SWD
- BOOT0 and RESET switches

## Hardware

### Primary parts

| Ref | Part | Function |
| --- | --- | --- |
| U4 | STM32L4Q5CGT6 (LQFP-48) | Microcontroller |
| X1 | 16 MHz crystal (9 pF, ±10 ppm) | HSE clock |
| U1 | SPX3819M5-L-3-3 | 3.3 V, 500 mA LDO |
| J1 | USB-C receptacle, USB 2.0, 16 pins | Power and USB data |
| U2 | USBLC6-2SC6 | ESD protection on D+ / D- |
| MK1 | TDK InvenSense ICS-43434 | I2S MEMS microphone (bottom side) |
| U3 | 1x8 pin header, 2.54 mm | Connection for the Waveshare 2inch LCD Module |
| S1, S2 | Tactile switches | BOOT0 and RESET |
| J2 | Tag-Connect TC2030 footprint | SWD, SWO, NRST and +3V3 (pads and locating holes only, not in the BOM) |

The schematic contains the name of the manufacturer, the MPN and the LCSC part number of each BOM part.

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

The power pins are:

- U3 pin 1 and J2 pin 1 are +3V3.
- U3 pin 2 and J2 pin 5 are GND.
- On the microphone, pin 5 is VDD (+3V3) and pin 3 is GND. LR (pin 2) connects to GND.

### Board

- 94.5 x 39.7 mm, with 1 mm rounded corners and four M3 mounting holes
- 4 layers: a top layer for signals, two inner GND planes, and a bottom layer for signals and a +3V3 pour
- Approximately 1.6 mm thick, with a HAL lead-free finish, a green solder mask and a white silkscreen
- The microphone is on the bottom side. All other parts are on the top side.

## Circuit

- **Power path.** VBUS from the USB-C receptacle goes through a 2.2 µH series inductor (L1) into the SPX3819 LDO. Each side of L1 has a 2.2 µF capacitor. The LDO is on when USB power is available, because its EN pin connects to its input. One +3V3 rail supplies power to all items on the board: VDD, VDDA, VDDUSB, the Waveshare 2inch LCD Module and the microphone. VBAT also connects to +3V3, and the board has no battery for VBAT.
- **USB.** 5.1 kΩ pull-down resistors on CC1 and CC2 identify the board as a USB sink. D+ and D- go through the USBLC6-2SC6 to PA12 and PA11. The SBU pins have no connection.
- **Clock.** A 16 MHz crystal with C0G load capacitors (10 pF) connects to PH0/PH1. The crystal footprint has a front-copper keep-out area. This area prevents copper pour below the crystal. The board has no 32.768 kHz LSE crystal (PC14/PC15 have no connection).
- **Microphone.** The footprint of the ICS-43434 has a 0.5 mm hole through the board for the acoustic port of the microphone. WS, SCK and SD connect to PB12, PB13 and PB15.
- **Display.** The board has a pin header footprint for the Waveshare 2inch LCD Module. The board has a silkscreen outline with the label "2inch LCD Module". The board also has four 3.2 mm holes for the mounting holes of the LCD module. The LCD module is on the left two-thirds of the board, and all other parts are on the right third.
- **BOOT0 and RESET.** S1 connects BOOT0 (PH3) to +3V3. When S1 is open, a 10 kΩ resistor keeps BOOT0 low. S2 connects NRST to GND. Each of the two lines has a 100 nF filter capacitor.

## How to start

1. Install [KiCad](https://www.kicad.org/) 10.0 or a higher version.

   **Note:** The project files are in the KiCad 10 format. KiCad 9 and previous versions cannot open these files.

2. Clone the repository:

   ```sh
   git clone https://github.com/OscarBazylewich/STM32_Audio_Spectrum_Analyzer_CustomPCB.git
   ```

3. Open `STM32L4Q5CGT-AudioSpectrumAnalyzer.kicad_pro` in KiCad.

   **Note:** The project files contain the symbols and the footprints. Thus, KiCad opens the project without other libraries.

4. If you want the full 3D view (**View > 3D Viewer** in the PCB Editor), install the [JLCPCB KiCad Library](https://github.com/CDFER/JLCPCB-Kicad-Library) with the Plugin and Content Manager.

   **Note:** KiCad uses the `${KICAD8_3RD_PARTY}` path variable to find the 3D models for X1, S1, S2, L1 and C1.

   If KiCad does not show the 3D models of these parts, do these steps:

   1. Open **Preferences > Configure Paths**.
   2. If the list does not contain `KICAD8_3RD_PARTY`, add a path variable with this name.
   3. Set `${KICAD8_3RD_PARTY}` to the folder that contains `3dmodels/com_github_CDFER_JLCPCB-Kicad-Library`.

5. In the Schematic Editor, do the ERC.
6. In the PCB Editor, do the DRC.
7. In the PCB Editor, go to **File > Fabrication Outputs > Gerbers**.
8. Set an output folder, for example `gerbers/`.

   **Note:** The output folder in the saved plot settings is the project folder. If you do not set an output folder, KiCad writes the output files to the project folder.

9. Select the copper, mask, paste, silkscreen and Edge.Cuts layers.
10. Click **Plot**.
11. Click **Generate Drill Files...**.
12. In the **Generate Drill Files** dialog, click **Generate**.

As an alternative to steps 7 to 12, use these commands in the repository folder:

```sh
kicad-cli pcb export gerbers --output gerbers/ STM32L4Q5CGT-AudioSpectrumAnalyzer.kicad_pcb
kicad-cli pcb export drill --output gerbers/ STM32L4Q5CGT-AudioSpectrumAnalyzer.kicad_pcb
```

## Repository structure

```text
.
├── STM32L4Q5CGT-AudioSpectrumAnalyzer.kicad_pro   # KiCad 10 project: design rules, net classes, BOM fields
├── STM32L4Q5CGT-AudioSpectrumAnalyzer.kicad_sch   # Schematic on one sheet (Revision A)
├── STM32L4Q5CGT-AudioSpectrumAnalyzer.kicad_pcb   # 4-layer PCB layout, 94.5 x 39.7 mm
├── .gitattributes                                  # Lets GitHub count the KiCad files in its language statistics
├── LICENSE                                         # GNU GPL v3.0
└── README.md
```

## 3D renders

The KiCad 9.0 renders include all 3D models, but the board corners are not rounded. The KiCad 10.0 renders do not include all 3D models (the 3D models of X1, S1, S2, L1 and C1 are missing). But these renders show the added GND pour on layers 1 and 4, and the rounded corners of the board outline in this repository.

<table>
  <tr>
    <th>KiCad 9.0: top side</th>
    <th>KiCad 9.0: bottom side</th>
  </tr>
  <tr>
    <td><img width="400" alt="KiCad 9.0 render of the top side of the board, with all 3D models" src="https://github.com/user-attachments/assets/85aaed98-7ecc-47bb-b375-da99e446172a" /></td>
    <td><img width="400" alt="KiCad 9.0 render of the bottom side of the board, with the ICS-43434 microphone, the pins of the LCD pin header and the mounting holes" src="https://github.com/user-attachments/assets/d1b51f1c-da5a-4e81-a80d-72842b435e69" /></td>
  </tr>
  <tr>
    <th>KiCad 10.0: top side</th>
    <th>KiCad 10.0: bottom side</th>
  </tr>
  <tr>
    <td><img width="400" alt="KiCad 10.0 render of the top side of the board, with rounded corners. The 3D models of the crystal (X1), the switches (S1, S2), the inductor (L1) and the tantalum capacitor (C1) are missing." src="https://github.com/user-attachments/assets/2df19aa7-58f1-4832-af66-1122759ae797" /></td>
    <td><img width="400" alt="KiCad 10.0 render of the bottom side of the board, with the ICS-43434 microphone and the pins of the LCD pin header" src="https://github.com/user-attachments/assets/eb0d8f54-d216-4bad-933a-f12b7d15c262" /></td>
  </tr>
</table>

## Related projects

- [STM32_Audio_Spectrum_Analyzer_ST7789_and_INMP441](https://github.com/OscarBazylewich/STM32_Audio_Spectrum_Analyzer_ST7789_and_INMP441): This project is the initial prototype and its firmware. It uses a Nucleo-F767ZI board, an ST7789 display and an INMP441 microphone.

## Credits

- Oscar Bazylewich made this KiCad project.
- The footprints and 3D models for X1, S1, S2, L1 and C1 are from the [JLCPCB KiCad Library](https://github.com/CDFER/JLCPCB-Kicad-Library) of CDFER. The symbols for X1, S1 and S2 are also from this library.
- The symbol and the outline of the Waveshare 2inch LCD Module are from a custom library.
- All other symbols, footprints and 3D models are from the standard KiCad libraries.

## License

The license of this project is the GNU General Public License v3.0. Refer to [LICENSE](LICENSE).
