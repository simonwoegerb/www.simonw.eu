---
title: "First Steps for the PCB | RF System: Part 2"
subtitle: "Component Selection and Power Stage Development"
date: 2026-06-10
tags: ["rf", "formula student", "power"]
categories: ["Radio Frequency Telemetry System"]
summary: "Essential components are selected and justified and the power stage developed"
draft: true
---
# Components
The design of the PCB calls for several dedicated circuitry for CAN interfacing, RF transmission, reverse polarity protection and the microcontroller.
## CAN
Several different CAN ICs were considered, before settling on the TCAN1057A[^TCAN1057A] by Texas Instruments. Reasons are plenty, yet the IC has been used by other projects in the team and as such was chosen as the standard CAN transceiver.
## Power Stage
The power stage needs to step down $\pu{12VDC}$ from the cars LV battery to $\pu{5VDC}$ and $\pu{3.3VDC}$ for the electronics. 
A peak current of around $\pu{350 mA}$ is expected on the $\pu{3.3VDC}$ rail, while the $\pu{5VDC}$ should be negligible, as it only drives the CAN transceiver. 
The highest load will be the RA-01SH at a peak current draw of $\pu{200 mA}$, followed by the STM32H533 with a maximum current draw of $\pu{70-95 mA}$.

The entire power stage will be reverse polarity protected to ensure that any mishaps in assembly do not destroy any electronics.
For further details regarding such a system, see onsemi application note AND90146/D[^AND90146]. 
The document outlines different options for reverse polarity protection include a schottky diode, a PMOS and an NMOS. 

Due to higher $I^2 R$ losses, the diode was not chosen. For higher implementation complexity, the PMOS was not selected. 
The PMOS is now the protection circuit of choice for this project.

A PMOS, connected as seen in Figure 4 of the reference document, will protect the entire circuit with very low $I^2 R$ losses. The Gate is clamped with a Zener diode to a maximum voltage of $\pu{15 V} to protect the MOSFET with a series resistor to limit the current. 

The exact selected PMOS is a UMW BSP170[^BSP170], with a maximum $V_{DS} = \pu{-60V}$ it lies comfortably in the $\pu{12V}$ range of the battery. Whilst the $R_{ds(on)} = \pu{300 m \Omega}$ is certainly not as low as it could be, with a maximum power draw of around $\pu{150 mA}$ on the $\pu{12V}$ line, the maximum power loss should be as follows:
$$P = \pu{150^2 mA}* \pu{300 m\Omega} = \pu{6.75 mW}$$

This is acceptable certainly acceptable and substantially better than using a 1N4007 with $P = \pu{0.7 V} * \pu{150 m\Omega} = \pu{0.105 W}$, whilst not being substantially more expensive nor does it take too much more PCB space.

[^TCAN1057A]: https://www.ti.com/lit/ds/symlink/tcan1057a-q1.pdf
[^AND90146]: https://www.onsemi.com/download/application-notes/pdf/and90146-d.pdf
[^BSP170]: https://www.umw-ic.com/static/pdf/3fc93cb5bd77497ae067183cb403213f.pdf