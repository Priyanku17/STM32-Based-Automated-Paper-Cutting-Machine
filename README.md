#  Automated Paper Cutting Machine (STM32-Based)

A dual-axis automated paper fabrication and shearing system built around the **STM32F103** microcontroller. The machine combines a vertical lead-screw clamping mechanism with a horizontal rack-and-pinion shearing assembly, using **sensorless current-based feedback** to achieve precise, repeatable cuts without manual intervention.

## Overview

Manual paper-cutting processes are prone to uneven cuts and material slippage due to human error. This project automates the process using two coordinated mechanical axes — vertical clamping and horizontal shearing — controlled by an STM32F103 microcontroller. Motor stall detection is achieved **sensorlessly**, using current-signature analysis rather than expensive position/force sensors, making the design both precise and cost-effective.

## Key Features

- **Dual-axis mechatronic design** — vertical lead-screw clamp + horizontal rack-and-pinion shear
- **Sensorless stall detection** via motor current sensing (no limit sensors needed for clamping force)
- **Finite State Machine (FSM) firmware** for safe, sequential operation
- Adjustable sensitivity via a potentiometer-based analog threshold
- Emergency stop via STM32 hardware "Break" input (TIM1_BKIN)
- LED-based diagnostic indicators for power, motor activity, and fault states
- Flash-stored calibration data so the machine "remembers" paper thickness across runs

## Control Core: STM32F103C8T6

| Feature | Specification |
|---|---|
| Processor Core | ARM 32-bit Cortex-M3 RISC |
| Max Clock Frequency | 72 MHz (1.25 DMIPS/MHz) |
| Operating Voltage | 2.0V – 3.6V |
| Flash Memory | 64 KB (64/128 KB options) |
| SRAM | 20 KB |
| ADC | 2× 12-bit, 1µs (up to 16 channels) |
| Timers | 3× 16-bit General Purpose, 1× 16-bit Advanced Control |
| Communication | 2× I2C, 3× USART, 2× SPI, 1× CAN, 1× USB 2.0 |
| GPIO | 37–80 pins |
| Debug Interface | SWD & JTAG |

**Development tools:** STM32CubeIDE + STM32CubeMX (pin/clock configuration), ST-Link V2 debugger (SWD protocol) for real-time monitoring and breakpoint debugging during stall-logic calibration.

## Mechanical Subsystems

### Vertical Clamping Mechanism (Lead Screw)
- Two 1-foot lead screws with a **double-nut, anti-backlash configuration** (nuts welded together to eliminate axial play)
- Self-locking property holds position under load without power
- Precision-welded nut board ensures instantaneous, noise-free press response — critical for clean current-signature readings

### Horizontal Shearing Mechanism (Rack & Pinion)
- Pinion-driven blade traverses a 6-foot channel
- Constant force profile across the full cutting width
- High linear velocity, durable in dusty environments, unlimited travel

| Drive Component | Advantages |
|---|---|
| Lead Screw (Vertical) | High mechanical advantage, self-locking, precision clamping |
| Rack & Pinion (Horizontal) | High translation speed, robust, unlimited travel |

## Electronics & Power Design

| Component | Qty | Function |
|---|---|---|
| Transformer (6-0-6) | 1 | Steps down 220V AC → 12V AC (center-tapped) |
| 6A Diode | 4 | Bridge rectification |
| 7805 IC | 2 | 5V regulation for MCU logic |
| Electrolytic Capacitor (4700 µF) | 2 | DC rail smoothing |
| MOSFET | 6 | High-current bidirectional motor switching |
| Resistor (330Ω, 0.25W) | 24 | LED current limiting / gate protection |
| Red LED | 6 | Diagnostic status indicators |
| Linear Potentiometer | 1 | Adjustable stall-detection sensitivity |

MOSFETs were chosen over BJTs for their low on-resistance (R_DS(on)) and reduced heat generation during high-torque clamping.

## Stall Detection Logic

The system exploits the linear relationship between motor torque and current (τ = K_t · I). As the press compresses the paper, motor current rises detectably:

1. A shunt resistor feeds the voltage drop into the STM32's 12-bit ADC.
2. Current pulses were first characterized on an oscilloscope during development to determine the "full tight" threshold.
3. Firmware applies a **blanking time** to ignore motor inrush current and avoid false positives.
4. A **moving-average filter** smooths commutation noise so only genuine mechanical stalls trigger a stop.
5. Once the threshold is exceeded, PWM to the vertical motor is disabled and the self-locking lead screw holds position.

## Firmware Logic (Finite State Machine)

1. **Initialization** — GPIO, ADC, and TIM1 (PWM) setup; 72 MHz clock via external crystal
2. **Clamping** — Vertical motor engages; current sampled at 1 kHz until stall is detected
3. **Shearing** — Horizontal motor drives the rack-and-pinion blade across the clamped stack
4. **Home** — Both motors reverse to reset position after a limit switch confirms cut completion

## Build Process

**Part 1 — Foundation**
- Component sourcing and verification (transformer, diodes, MOSFETs tested individually)
- STM32 architecture learning: clock config (RCC), ADC continuous-conversion mode, USART serial debug
- Precision welding of the nut board for backlash-free vertical travel

**Part 2 — Integration & Calibration**
- Laser-aligned 6-foot channel with the nut board for a parallel, perpendicular cut
- "Jog" mode and learning-cycle calibration to record and flash-store paper-thickness current signatures
- Safety features: emergency stop (TIM1_BKIN), acceleration/deceleration ramping on the horizontal axis

## Future Enhancements

- I2C LCD display for cut count and paper thickness
- Upgrade horizontal axis to ball screws for higher precision on dense materials (cardstock, thin plastics)
- Additional sensor integration for closed-loop position feedback

## References

1. STMicroelectronics, *STM32F103C8T6 ARM Cortex-M3 Datasheet and Reference*, 2024
2. YouTube/MicroPeta by Nizar Mohideen
3. [stm32-base.org — STM32F103C8T6 Blue Pill](https://stm32-base.org/boards/STM32F103C8T6-Blue-Pill.html)
4. [reversepcb.com/stm32f103](https://reversepcb.com/stm32f103/)

## Author

**Priyanku Buragohain**
3rd Year Student, Department of Electronics and Communication Engineering (ECE)

