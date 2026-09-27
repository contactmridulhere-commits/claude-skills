# Arduino Pinout Reference

## Arduino Uno (ATmega328P)

| Pin | Digital | Analog | PWM | Special Function |
|-----|---------|--------|-----|------------------|
| D0  | Yes     | -      | -   | UART RX (Serial) |
| D1  | Yes     | -      | -   | UART TX (Serial) |
| D2  | Yes     | -      | -   | INT0 (external interrupt) |
| D3  | Yes     | -      | ~   | INT1, PWM (Timer2) |
| D4  | Yes     | -      | -   | - |
| D5  | Yes     | -      | ~   | PWM (Timer0) |
| D6  | Yes     | -      | ~   | PWM (Timer0) |
| D7  | Yes     | -      | -   | - |
| D8  | Yes     | -      | -   | - |
| D9  | Yes     | -      | ~   | PWM (Timer1) |
| D10 | Yes     | -      | ~   | PWM (Timer1), SPI SS |
| D11 | Yes     | -      | ~   | PWM (Timer2), SPI MOSI |
| D12 | Yes     | -      | -   | SPI MISO |
| D13 | Yes     | -      | -   | SPI SCK, Built-in LED |
| A0  | Yes     | ADC0   | -   | - |
| A1  | Yes     | ADC1   | -   | - |
| A2  | Yes     | ADC2   | -   | - |
| A3  | Yes     | ADC3   | -   | - |
| A4  | Yes     | ADC4   | -   | I2C SDA |
| A5  | Yes     | ADC5   | -   | I2C SCL |

- **Operating Voltage**: 5V logic, 7-12V input (Vin)
- **Max current per GPIO**: 20mA (absolute max 40mA)
- **Total GPIO current**: 200mA max across all pins
- **ADC Resolution**: 10-bit (0-1023)
- **Flash**: 32KB (0.5KB bootloader), **SRAM**: 2KB, **EEPROM**: 1KB
- **Clock**: 16MHz

### Uno Gotchas
- D0/D1 shared with USB serial — don't use for peripherals if using Serial Monitor
- Only 6 PWM pins (3, 5, 6, 9, 10, 11)
- Timer0 (pins 5, 6) ~976Hz PWM; Timer1/Timer2 (others) ~490Hz
- A4/A5 are the ONLY I2C pins — no remapping
- Analog pins can be used as digital (D14-D19)

## Arduino Mega (ATmega2560)

- **Digital Pins**: 54 (15 PWM), **Analog**: 16 (A0-A15)
- **Serial Ports**: 4 (Serial, Serial1, Serial2, Serial3)
- **I2C**: SDA=D20, SCL=D21
- **SPI**: MOSI=D51, MISO=D50, SCK=D52, SS=D53
- **Interrupts**: 6 (pins 2, 3, 18, 19, 20, 21)
- **Flash**: 256KB, **SRAM**: 8KB, **EEPROM**: 4KB
- SPI pins are 50-53, NOT 10-13 like Uno

## Arduino Nano (ATmega328P)
Same as Uno but: smaller form factor, A6/A7 analog-only, some clones use CH340 USB.

## Arduino Due (SAM3X8E — ARM Cortex-M3)
- **3.3V logic — 5V on any pin WILL DAMAGE IT**
- 54 digital (12 PWM), 12 analog + 2 DAC, 12-bit ADC
- 512KB flash, 96KB SRAM, 84MHz, CAN bus, native USB host
