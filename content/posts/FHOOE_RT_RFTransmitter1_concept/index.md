---
title: "Car → Pitwall Telemetry Transmission Solution | RF System: Part 1"
subtitle: "A concept for Formula Student"
date: 2026-06-09
tags: ["rf", "formula student", "concept", "devlogs"]
categories: ["Radio Frequency Telemetry System"]
summary: "The intial ideas for a Formula Student ready RF telemetry transmission system based on STM32"
---
# The Reason
Due to my current activity in the [FHOOE Racing Team](https://fhooe-racing.at/), it has come to happen that I am in charge of the RF-based telemetry transmission and analysis now.
Laying aside my lack of experience with anything beyond the word RF, I will attempt my best to document my design choices and suffering in this blog.
# The Concept
In essence, a STM32 based solution was chosen. A custom receiver and transmitter board (ideally the same PCB can be reused) will be designed and tested.

{{< figure src="schem.png" caption="Design Schematic" >}}

The PCB will contain a CAN transceiver, which combined with an STM32H5 will read out all needed data from the CAN bus and use an RA01-SH LoRa module to encode and transmit the data to the pit-wall. The same board with different firmware will then be reused to receive the data, decode it and insert it into a InfluxDB instance. This is then read out via a Grafana or ImGui dashboard (most likely the former for ease of use).
# Choice and Justification of Components 
## The Microcontroller
The microcontroller in question is a STM32H5, which was chosen for the simple reason that it has a cryptographic chip onboard to speed up the signing of the transmitted packets. Signing will be used to ensure the packet actually belongs to this team (A transmission band will be used that I am rather certain many other teams will be using too, see below) and that it was not corrupted.

In terms of clock-speed it is sufficient(ly overkill) and supports proper CAN-FD interfacing with any standard CAN transceiver.
Furthermore, the STM32 ecosystem is well supported in automotive context and the FH OOE Racing Team standard for all MCU use-cases.
## Legal considerations and transmission band choice
The recommended transmission band is EU868, specifically the P subband, at $\pu{869.4-869.65 MHz}$.
This transmission band allows for an airtime/duty cycle of $\pu{10\\%}$, as well as a maximum Effective Radiated Power (ERP) of $\pu{500mW}$. Expected peak TX ERP is around $\pu{60-80 mW}$, not even close to the limit in any case. 
The EU868 band is one of the few bands which can be used without any licensing.
## Modulation method and range considerations 
The RA-01Sh supports several different modulation methods. 
The two interesting ones for the FS usecase are LoRa and FSK/GFSK. 

**Lo**ng **Ra**nge is, as the name implies, optimized for long range transmission, based on the fact that it has very high receiver sensitivity. This is caused by the fact that is uses a "chirp"-based spectrum modulation. 
This allows it to decode signals even with negative Signal-to-Noise ratios down to $\pu{-20dB}$. As a trade-off for this long range, the throughput is rather abysmal at less than $\pu{1 kbps}$.

**G**aussian **F**requency **S**hift **K**eying on the contrary is a digital modulation scheme where data is transmitted by shifting the carrier frequency between discrete steps. A gaussian filter is applied to smoothen the frequency transmission.

| Statistic  | FSK/GFSK   |   LoRa|
|:---:|:---:|:---:|
|  Throughput        | High  | Low  |
|   Range            | Medium  | Very High  |
|   Latency          | Low  | Medium  | 
| Airtime efficiency | High | Low |
| Complexity         | Low | Medium |

Whilst the above table is only a very rough estimate of LoRa and GFSK, it is evident that GFSK is a better fit for the use-case.

The peak ranges of LoRa with around $\pu{10km}$ **L**ine **O**f **S**ight are simply not necessary in a Formula Student context.
Furthermore, the peak throughput is substantially worse with LoRa.
GFSK can effectively be used as "UART-over-air", which seems to be the most user-friendly choice for first experiences with RF.

GFSK allows for around $\pu{4.8-9.6kbps}$, which considering proper bitpacking in the protocol nearly allows for a full CAN dump at a $\pu{10Hz}$ update frequency.
## Antenna Placements and choice
### Pitwall side
An antenna similar to a RAK Wireless 8 dBi Fiberglass 868 Mhz[^rakantenna] is recommended to be used due to its long max. range. The range can only be obtained if the antenna is placed at least $\pu{3-5m}$ above the ground at the pit wall.
### Car side
Any quarter-wave whip antenna mounted on top of the car should be sufficient. No choice has been made yet.

[^rakantenna]:   https://store.rakwireless.com/products/fiber-glass-antenna-1
## Overall systems architecture for reception on pitwall and data analysis
Pitwall architecture will be based on a locally hosted InfluxDB v2 OSS instance. InfluxDB is optimized for time-series measurements and shows high write speeds. It integrates easily with Grafana to visualize live data during the race.
# Future outlook
Now that a basic concept exists, the PCB will be designed. Await part 2 for that.