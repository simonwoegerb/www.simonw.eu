---
title: "Race Car Telemetry Transmission"
subtitle: "A concept for Formula Student"
date: 2026-06-09
categories: ["fsa", "formula student"]
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
The microcontroller in question is a STM32H5, which was chosen for the simple reason that it has a cryptographic chip onboard to speed up the signing of the transmitted packets. Signing will be used to ensure the packet actually belongs to my team (A transmission band will be used that I am rather certain many other teams will be using too, see below) and that it was not corrupted.