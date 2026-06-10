---
title: "First Steps for the PCB | RF System: Part 2"
subtitle: "Component Selection and Power Stage Development"
date: 2026-06-10
tags: ["rf", "formula student", "power"]
categories: ["FHOOE Racing Team Devlogs"]
summary: "Essential components are selected and justified and the power stage developed"
draft: true
---
# Components
The design of the PCB calls for several dedicated circuitry for CAN interfacing, RF transmission, reverse polarity protection and the microcontroller.
## CAN
Several different CAN ICs were considered, before settling on the TCAN1057A[^TCAN1057A] by Texas Instruments.

[^TCAN1057A]: https://www.ti.com/lit/ds/symlink/tcan1057a-q1.pdf