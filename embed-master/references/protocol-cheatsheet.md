# Communication Protocol Cheatsheet

## I2C
- 2 wires: SDA + SCL, plus GND. Pull-ups required.
- Standard 100kHz: 4.7kΩ pull-ups. Fast 400kHz: 2.2kΩ. Fast+ 1MHz: 1kΩ.
- 7-bit addresses, max ~1m bus length. Run I2C scanner to detect devices.

### I2C Scanner (Arduino)
```cpp
#include <Wire.h>
void setup() {
  Wire.begin(); Serial.begin(115200);
  for (byte a=1; a<127; a++) { Wire.beginTransmission(a); if(!Wire.endTransmission()) { Serial.print("0x"); Serial.println(a,HEX); } }
}
void loop() {}
```

### Common I2C Addresses
MPU6050: 0x68/0x69 | BME280: 0x76/0x77 | SSD1306: 0x3C/0x3D | PCA9685: 0x40+ | INA219: 0x40+

## SPI
- 4 wires: MOSI, MISO, SCK, CS (per device) + GND
- Full-duplex, up to 80MHz on ESP32. Each device needs own CS pin.
- Mode 0 (CPOL=0/CPHA=0) most common — verify in datasheet.

## UART
- 2 wires: TX→RX cross-connected + GND. **TX of A → RX of B.**
- Common baud: 9600 (GPS), 115200 (default modern), 921600 (high-speed)
- Level shifting (MAX3232) if voltages differ

## 1-Wire (Dallas)
- Single DQ wire + GND + 4.7kΩ pull-up to VCC
- Each device has 64-bit unique address — multiple on one wire
- Libraries: `OneWire` + `DallasTemperature`

## PWM Quick Reference
| Application | Frequency | Notes |
|-------------|-----------|-------|
| LED dimming | 1-5kHz | 8-bit resolution |
| Servo | 50Hz | 500-2400µs pulse width |
| DC motor | 5-25kHz | 8-10 bit |
| Buzzer | 20Hz-20kHz | tone() function |
| WS2812B | 800kHz | Use NeoPixel/FastLED library |

## Voltage Divider (read voltages above ADC range)
```
Vout = Vin × R2 / (R1 + R2)
Example: 12V→3.3V ADC: R1=100kΩ, R2=33kΩ → 2.98V
```
