# Raspberry Pi Pico / Pico W — RP2040 Reference

## Key Specs
- **CPU**: Dual-core ARM Cortex-M0+ @ 133MHz
- **SRAM**: 264KB, **Flash**: 2MB
- **ADC**: 4ch 12-bit (noisy — average readings), **PWM**: 16ch (8 slices × 2)
- **PIO**: 2 blocks × 4 state machines (custom protocols at up to 133MHz)
- **USB**: 1.1 Host/Device, **Voltage**: 3.3V (NOT 5V tolerant)
- **Pico W**: Adds CYW43439 WiFi 2.4GHz + BLE 5.2

## GPIO Quick Map (all 30 GPIOs)
All GPIOs support digital I/O. Peripherals are flexibly muxed:
- **I2C**: I2C0 or I2C1 on almost any pin (alternating pattern)
- **SPI**: SPI0 or SPI1 on most pins
- **UART**: UART0 or UART1 on many pins
- **PWM**: Every GPIO has a PWM slice assignment
- **ADC**: GP26 (ADC0), GP27 (ADC1), GP28 (ADC2), GP29 (ADC3/VSYS sense)

## Special Pins (don't use for general I/O)
- **GP23**: Voltage regulator control (Pico board)
- **GP24**: VBUS detect
- **GP25**: Onboard LED (standard Pico) — on Pico W, LED is on WiFi chip
- **GP29**: VSYS/3 voltage sense

## Pico W LED
```python
# MicroPython — Pico W LED is NOT on GP25
import machine
led = machine.Pin("LED", machine.Pin.OUT)  # Use "LED" string
led.on()
```

## PIO (Programmable I/O)
8 independent state machines, 32-instruction programs each, up to system clock speed.
Use for: WS2812B NeoPixels, DPI video, quadrature encoders, custom serial protocols.

## Common Mistakes
1. GP25 for LED on Pico W (it's on WiFi chip now)
2. Not averaging ADC readings (inherently noisy)
3. No EEPROM — use flash storage or external chip
4. GP23/24/29 are board power management pins
