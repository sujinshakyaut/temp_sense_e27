# Temperature Sensor Board

[![KiCad](https://img.shields.io/badge/KiCad-7.x-blue.svg)](https://www.kicad.org/)
[![License: MIT](https://img.shields.io/badge/license-MIT-yellow.svg)](LICENSE)
[![MCU](https://img.shields.io/badge/MCU-Texas%20Instruments-red.svg)](https://www.ti.com/)
[![BMS](https://img.shields.io/badge/BMS-Orion%20BMS-brightgreen.svg)](https://www.orionbms.com/)
[![Hardware](https://img.shields.io/badge/type-hardware-orange.svg)]()

A compact multi-channel temperature sensing PCB built around the TI MSPM0C1104 microcontroller with CAN FD communication. Designed to be later integrated on a voltage tap board that mounts directly onto the Orion BMS module, providing up to 24 analog temperature inputs per tapboard or breakout board multiplexed through a 3 ADC channel with robust power protection for automotive and industrial environments.

## Table of Contents

- [Background](#background)
- [Hardware](#hardware)
- [NTC Thermistors & Steinhart-Hart](#ntc-thermistors--steinhart-hart)
- [Validation & Troubleshooting](#validation--troubleshooting)
- [Features](#features)
- [Contributing](#contributing)
- [License](#license)

## Background

This is a redesigned temperature sensing board intended to be integrated with a voltage tap board on the Orion BMS module, providing cell temperature monitoring across the battery pack. The Orion BMS is highly capable, but routing dozens of analog thermistor wires directly back to the main unit introduces noise vulnerability and wiring harness complexity. This decentralized satellite board digitizes localized analog sensor data and broadcasts it over a noise-immune CAN FD bus.

<img src="https://api.memegen.link/images/fine/60V_transient_spikes/in_the_accumulator.jpg" alt="This is fine" width="200"/>

The board consolidates power regulation, input protection, analog multiplexing, and CAN FD communication onto a single compact PCB. A three-stage power supply has been implemented due to potential 60V voltage spikes from the battery module. Power input is protected by an LM74500 ideal diode controller and an SMF24CA TVS diode, with the AP63205 buck converter stepping down to 5 V and a TPS7A03 LDO providing a clean 3.3 V rail. A fuse is included for extra protection.

The CLV4051A 8:1 analog multiplexer allows a single ADC input on the MSPM0C to scan up to 8 temperature sensor channels sequentially, reducing pin count. The MCP2517FD provides CAN FD connectivity over SPI.

## Hardware

<img src="https://api.memegen.link/images/drake/Standard_Schottky_diodes/LM74500_ideal_diode_controller.jpg" alt="Drake Meme" width="200"/>

| Component | Part | Description |
|-----------|------|-------------|
| MCU | MSPM0C1104SDGS20R | TI MSPM0C Arm Cortex-M0+ microcontroller |
| CAN FD | MCP2517FD-H/SL | Microchip CAN FD controller (SPI) |
| Analog Mux | CLV4051ATDWRG4Q1 | TI 8:1 analog multiplexer |
| Buck Converter | AP63205 | 2 A synchronous buck, input to 5 V |
| LDO | TPS7A03 | 3.3 V LDO for MCU and analog rails |
| Ideal Diode | LM74500QDDFRQ1 | Reverse polarity and ORing protection |
| MOSFET | SUD20N10-66L-GE3 | N-channel power switch |
| TVS | Littelfuse SMF24CA | 24 V bidirectional transient protection |
| Crystal | HC49/4HSMX | MCU clock source |
| Fuse | Keystone 3568 | Blade mini fuse holder |

## NTC Thermistors & Steinhart-Hart

This board interfaces with Negative Temperature Coefficient (NTC) thermistors (typically 10 kΩ at 25°C). The thermistors are placed in a voltage divider circuit with a known 10 kΩ bias resistor. Because the resistance curve of an NTC is highly non-linear, the firmware utilizes the Steinhart-Hart equation for accurate conversion from resistance to temperature across the accumulator's operating range:

$$\frac{1}{T} = A + B \ln(R) + C (\ln(R))^3$$

Where:
* T = Temperature in Kelvin
* R = Measured resistance of the thermistor in Ohms
* A, B, C = Steinhart-Hart coefficients provided by the thermistor manufacturer

The MCU reads the 12-bit ADC value, calculates the resistance R via the voltage divider equation, applies the Steinhart-Hart conversion to find Kelvin, and then broadcasts the Celsius equivalent over CAN FD.

## Validation & Troubleshooting

Rigorous system-level troubleshooting and component validation are required for reliable operation in high-performance environments.

* **Power Rail Validation:** Prior to MCU initialization, verify the buck converter output at exactly 5.0 V (+/- 1%) and the LDO output at 3.3 V. 
* **Protection Circuitry:** The SMF24CA and LM74500 are selected to clamp and isolate reverse transients. Verify the blade fuse continuity and the SUD20N10 MOSFET gate drive state if the board remains unpowered while connected to an active accumulator rail.
* **CAN & SPI Diagnostics:** If the Orion BMS fails to acknowledge the node, use an oscilloscope to validate the MCP2517FD SPI clock logic and verify that the 120 Ω CAN termination resistor aligns with the overall bus topology.

## Features

- 8-channel analog temperature input via CLV4051A multiplexer
- CAN FD communication (MCP2517FD over SPI) to Orion BMS
- Wide input voltage range with reverse polarity and TVS protection
- On-board buck + LDO three-stage power supply with 24V voltage regulation
- High-accuracy NTC thermistor reading using Steinhart-Hart conversion
- Screw terminal connectors
- M3 mounting holes for panel or DIN mount

## Contributing

Open an issue or pull request on [GitHub](https://github.com/sujinshakyaut/temp_sense_e27).
