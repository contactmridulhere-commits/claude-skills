# ESP32 / ESP8266 Pinout & Safe GPIO Guide

## ESP32 GPIO Safety Classification

| Category | GPIOs | Notes |
|----------|-------|-------|
| **Safe** | 4, 5, 16, 17, 18, 19, 21, 22, 23, 25, 26, 27, 32, 33 | Use freely |
| **Input only** | 34, 35, 36 (VP), 39 (VN) | No pull-up, no output |
| **Strapping** | 0, 2, 12, 13, 14, 15 | Affect boot if held wrong |
| **FLASH — DO NOT USE** | 6, 7, 8, 9, 10, 11 | Internal flash SPI |
| **ADC2 (WiFi conflict)** | 0, 2, 4, 12-15, 25-27 | ADC2 unavailable with WiFi |

### Strapping Details
- GPIO 0: HIGH at boot (LOW = download mode). Safe after boot.
- GPIO 2: LOW at boot on some modules. Often onboard LED.
- GPIO 12: Sets flash voltage — HIGH = 1.8V (crashes most). Keep LOW at boot.
- GPIO 15: LOW at boot suppresses UART boot messages.

### Peripheral Mapping
| Function | Default | Remappable? |
|----------|---------|-------------|
| I2C | SDA=21, SCL=22 | Yes — `Wire.begin(sda, scl)` |
| VSPI | MOSI=23, MISO=19, SCK=18, SS=5 | Yes |
| HSPI | MOSI=13, MISO=12, SCK=14, SS=15 | Yes |
| UART0 | TX=1, RX=3 | Yes (but default is USB) |
| UART2 | TX=17, RX=16 | Yes |
| DAC | DAC1=25, DAC2=26 | No |

**Specs**: Dual-core 240MHz, 520KB SRAM, 4MB flash, WiFi+BLE, 3.3V (NOT 5V tolerant), 12mA/pin recommended, deep sleep ~10µA.

### Common ESP32 Mistakes
1. Using GPIO 6-11 (flash crash) 2. ADC2 with WiFi 3. GPIO 12 strapping 4. Powering motors from USB 5. ADC non-linear above 3.1V

## ESP8266 (NodeMCU)

| NodeMCU | GPIO | Safe? | Notes |
|---------|------|-------|-------|
| D1 | GPIO5 | ✅ | SCL |
| D2 | GPIO4 | ✅ | SDA |
| D5 | GPIO14 | ✅ | SPI SCK |
| D6 | GPIO12 | ✅ | SPI MISO |
| D7 | GPIO13 | ✅ | SPI MOSI |
| D0 | GPIO16 | ⚠️ | No PWM/I2C, deep sleep wake |
| D3 | GPIO0 | ⚠️ | Strapping, must be HIGH at boot |
| D4 | GPIO2 | ⚠️ | Strapping, onboard LED |
| D8 | GPIO15 | ⚠️ | Must be LOW at boot |
| A0 | ADC0 | ✅ | Only analog pin, 0-1V |

**Specs**: Single-core 80/160MHz, ~50KB usable SRAM, WiFi only, no BLE, 1 ADC, 3.3V NOT 5V tolerant.
