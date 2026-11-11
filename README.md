# Relay-Based Cyclic Timer on ARM Cortex-M0+ (Internship Project)

Six-week industrial internship in embedded firmware development (Design & Development – Software), **19 May – 28 June 2025**.

> The source code isn't included because it belongs to the host organisation. This repository describes the design, the approach and what I learned.

---

## Project overview

I designed a relay-based cyclic timer on the **NXP MKE02Z (ARM Cortex-M0+)** microcontroller. It switches two outputs back and forth, but only after the inputs confirm each step. A safety delay comes before every switch, and the total cycle count is kept in non-volatile memory so it survives power loss.

## How it works

1. **Power-up:** initialise clocks (40 MHz), GPIO, PIT timer and flash driver. Read the last saved cycle count from the EEPROM area. Set Output1 = HIGH, Output2 = LOW.
2. **Wait** up to 10 s for the first input condition (Input1 = HIGH, Input2 = LOW).
3. **Validate:** the input must stay stable for 1 s (1 ms sampling, 1000 consecutive samples) so noise and glitches are rejected.
4. **Safety delay:** a 3 s non-blocking delay lets the relays settle and gives a window to abort.
5. **Toggle:** switch to Output1 = LOW, Output2 = HIGH. Then wait for the reverse input condition, validate it and apply the delay again.
6. **Complete the cycle:** switch the outputs back, increment the cycle count, erase the sector and write the 2-byte count to EEPROM.
7. **Repeat.** The cycle count is shown on an OLED display and status is logged over UART.

## Firmware design

- Non-blocking **state machine**, 5 states, called from the main loop
- **PIT timer interrupt** generates periodic time flags. Interrupt handlers stay short and the main loop does the work.
- **Emulated EEPROM in on-chip flash:** the address comes from the flash properties, with erase-then-write and read-back verification
- **Keypad scanning** with debouncing, plus an **OLED display** of the live cycle count
- Modular structure with separate modules for GPIO, timers, EEPROM, UART, display and keypad

## Hardware

- 3 SPDT relays driven through transistor drivers with flyback diodes
- 2 digital inputs
- Separate supplies: 3.3 V for the logic and 5 V/12 V for the relay coils, with a common ground and decoupling

---

## Other work during the internship

- **Microcontrollers:** studied NXP Kinetis parts (MKV20, MKV30, MKE02Z): pin packages (64/48-pin LQFP, 32-pin QFN), ports, JTAG/SWD. Learned to read datasheets and reference manuals (register maps, memory layout, initialisation sequences).
- **ADC:** ADC types (flash, ramp/counting, successive approximation), resolution and sampling theory. Built a working 12-bit polling ADC project with readings printed over UART.
- **UART:** framing and baud-rate calculation. Built a polling echo program and timed transmission using the PIT.
- **Timers and interrupts:** SysTick/PIT configuration, a 1 ms system tick, and interrupt handlers for timer and UART events.
- **Flash/EEPROM:** write, read and verify cycles, plus persistence tests across power cycles.
- **Communication protocols:** introduction to I²C and SPI.
- **Hardware bench work:** soldering, multimeter validation, FTDI USB-to-UART connections, noise filtering and debouncing.

## Tools used

| Tool | Purpose |
|---|---|
| MCUXpresso IDE | Project setup, pin/peripheral configuration, build, debug |
| PE Micro Multilink Universal FX | SWD debugging and flash programming |
| Docklight | Serial terminal for UART logs and testing |
| FTDI cable | USB-to-UART link to the PC |
| Multimeter / oscilloscope | Voltage checks and relay switching transients |

**Language:** Embedded C

## Key learnings

- A 1-second input stability window greatly reduced false triggers from noisy signals.
- Mechanical relays need a proper driver stage and flyback protection to keep the MCU pins safe.
- Writing repeatedly to one flash address wears it out, so a production design would need wear-levelling.
- Keeping the logic supply separate from the relay-coil supply prevented resets caused by coil inrush current.
- Using breakpoints together with UART logs helped me find timing race conditions and edge cases.

**Skills:** Embedded C · ARM Cortex-M · State machines · Timers & interrupts · Flash/EEPROM · UART · ADC · Relay interfacing · Hardware debugging
