# Raspberry Pi (Full SBC) Reference — Pi 3 / 4 / 5 / Zero

## Model Comparison

| Spec | Pi 5 | Pi 4B | Pi 3B+ | Pi Zero 2 W |
|------|------|-------|--------|-------------|
| CPU | Cortex-A76 × 4 | Cortex-A72 × 4 | Cortex-A53 × 4 | Cortex-A53 × 4 |
| Clock | 2.4GHz | 1.8GHz | 1.4GHz | 1.0GHz |
| RAM | 4/8GB | 1/2/4/8GB | 1GB | 512MB |
| USB | 2×USB3 + 2×USB2 | 2×USB3 + 2×USB2 | 4×USB2 | 1×Micro-USB OTG |
| Ethernet | Gigabit | Gigabit | 300Mbps | None (WiFi only) |
| WiFi | 802.11ac dual-band | 802.11ac dual-band | 802.11ac | 802.11n 2.4GHz |
| Bluetooth | BLE 5.0 | BLE 5.0 | BLE 4.2 | BLE 4.2 |
| Video Out | 2×micro-HDMI 4Kp60 | 2×micro-HDMI 4Kp60 | 1×HDMI 4Kp30 | Mini-HDMI |
| Camera | 2×CSI (4-lane) | 1×CSI (2-lane) | 1×CSI (2-lane) | 1×CSI (mini) |
| PCIe | 1×PCIe 2.0 x1 | None | None | None |
| GPIO I/O chip | RP1 (dedicated) | SoC direct | SoC direct | SoC direct |
| Power | USB-C 5V/5A (PD) | USB-C 5V/3A | Micro-USB 5V/2.5A | Micro-USB 5V/1.2A |
| Idle power | ~3W | ~2.7W | ~1.8W | ~0.5W |
| Peak power | ~12W | ~7.5W | ~5W | ~2W |
| Price | $60-80 | $35-75 | $35 | $15 |

### When to Choose Each
- **Pi 5**: Edge AI, LLMs, multi-camera CV, heavy compute, PCIe peripherals (NVMe, AI accelerators)
- **Pi 4**: General projects, single camera CV, lightweight ML, home server
- **Pi 3B+**: Legacy projects, simple IoT, learning
- **Pi Zero 2 W**: Space-constrained IoT, battery projects, headless sensors

## GPIO (40-pin header — same layout on all models)

All GPIOs are **3.3V logic — NOT 5V tolerant.** 5V on any GPIO damages the SoC.

### Pin Layout (physical pin numbers)
```
3V3  [1]  [2]  5V
GP2  [3]  [4]  5V
GP3  [5]  [6]  GND
GP4  [7]  [8]  GP14 (UART TX)
GND  [9]  [10] GP15 (UART RX)
GP17 [11] [12] GP18 (PCM CLK / PWM0)
GP27 [13] [14] GND
GP22 [15] [16] GP23
3V3  [17] [18] GP24
GP10 [19] [20] GND        (SPI0 MOSI)
GP9  [21] [22] GP25       (SPI0 MISO)
GP11 [23] [24] GP8        (SPI0 SCLK / SPI0 CE0)
GND  [25] [26] GP7        (SPI0 CE1)
GP0  [27] [28] GP1        (I2C0 — EEPROM, avoid)
GP5  [29] [30] GND
GP6  [31] [32] GP12 (PWM0)
GP13 [33] [34] GND        (PWM1)
GP19 [35] [36] GP16
GP26 [37] [38] GP20
GND  [39] [40] GP21
```

### Default Peripheral Pins
| Function | Pins | Notes |
|----------|------|-------|
| I2C1 | SDA=GP2, SCL=GP3 | Default user I2C bus |
| I2C0 | SDA=GP0, SCL=GP1 | Reserved for HAT EEPROM — avoid |
| SPI0 | MOSI=GP10, MISO=GP9, SCLK=GP11, CE0=GP8, CE1=GP7 | |
| SPI1 | MOSI=GP20, MISO=GP19, SCLK=GP21, CE0=GP18 | |
| UART0 | TX=GP14, RX=GP15 | Enable in raspi-config |
| PWM0 | GP12, GP18 | Hardware PWM channel 0 |
| PWM1 | GP13, GP19 | Hardware PWM channel 1 |

### GPIO Libraries (Python)
- **`gpiod`** (recommended for Pi 5+): Modern Linux GPIO interface
- **`lgpio`**: Direct library, works on Pi 5
- **`RPi.GPIO`**: Legacy — DOES NOT WORK on Pi 5 (no /dev/gpiomem)
- **`pigpio`**: Daemon-based, precise timing, works on Pi 4 and earlier
- **`gpiozero`**: High-level (LED, Button, Servo classes), uses lgpio backend

```python
# Pi 5 compatible GPIO (gpiozero + lgpio backend)
from gpiozero import LED, Button
led = LED(17)
button = Button(22)
button.when_pressed = led.on
button.when_released = led.off
```

### Pi 5 Specific Notes
- GPIO managed by RP1 chip (not SoC) — different /dev interface
- `RPi.GPIO` library does NOT work — use `gpiozero`, `lgpio`, or `gpiod`
- PCIe x1 slot: NVMe SSD (via HAT), Coral TPU, Hailo-8 AI accelerator
- Active cooling recommended (official Active Cooler or fan HAT)
- Power: Requires USB-C PD 5V/5A for full performance — underpowered = throttling
- Real-time clock (RTC) built-in (needs coin cell battery for backup)

## Camera Setup

### Hardware
- Pi 5: 2× CSI connectors (4-lane MIPI), use 15-pin to 22-pin FPC adapter
- Pi 4/3: 1× CSI connector (2-lane MIPI), standard 15-pin FPC cable
- Camera Module 3: 12MP, autofocus, HDR, ~$25
- Camera Module v2: 8MP, fixed focus, ~$25

### Software (libcamera stack — Pi 4/5)
```bash
# Test camera
libcamera-hello --timeout 5000

# Capture still
libcamera-still -o test.jpg

# Record video
libcamera-vid -t 10000 -o test.h264

# Python (picamera2)
pip install picamera2
```

```python
from picamera2 import Picamera2
import cv2

picam = Picamera2()
picam.configure(picam.create_preview_configuration(main={"size": (640, 480)}))
picam.start()

while True:
    frame = picam.capture_array()
    # frame is a numpy array — use with OpenCV
    cv2.imshow("Camera", frame)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break
```

### Legacy Camera (Pi 3 with raspistill)
```bash
# Only if using legacy camera stack
sudo raspi-config  # Enable Legacy Camera under Interface Options
raspistill -o test.jpg
```

## OS Setup Essentials

### Recommended OS
- **Raspberry Pi OS (64-bit, Bookworm)** — default choice
- **Ubuntu Server 24.04** — for Docker/server workloads
- **DietPi** — minimal, optimized for headless/IoT

### First Boot Checklist
```bash
sudo apt update && sudo apt upgrade -y
sudo raspi-config  # Set locale, enable I2C/SPI/UART/Camera
# For Python projects:
python3 -m venv ~/myenv && source ~/myenv/bin/activate
pip install gpiozero lgpio numpy opencv-python-headless
```

### Headless Setup (no monitor)
1. Flash OS with Raspberry Pi Imager — set WiFi + SSH + username in Advanced Options
2. Boot, find IP: `nmap -sn 192.168.1.0/24` or check router DHCP leases
3. SSH in: `ssh pi@<ip-address>`

### Auto-start Scripts on Boot
```bash
# Using systemd service (recommended)
sudo nano /etc/systemd/system/myproject.service
```
```ini
[Unit]
Description=My Project
After=network.target

[Service]
ExecStart=/home/pi/myenv/bin/python /home/pi/project/main.py
WorkingDirectory=/home/pi/project
Restart=always
User=pi

[Install]
WantedBy=multi-user.target
```
```bash
sudo systemctl enable myproject
sudo systemctl start myproject
```

## Power & Cooling

### Pi 5 Power Requirements
- Official 27W USB-C PD supply recommended
- Underpowered → yellow lightning bolt icon → CPU throttling
- HAT power: Each USB port provides up to 600mA (1.6A total with PD supply)
- Power over Ethernet (PoE): Use PoE+ HAT

### Cooling
- Pi 5: Active cooling mandatory under sustained load (AI inference, compilation)
- Pi 4: Heatsink sufficient for most tasks, fan for CV/ML workloads
- Pi 3/Zero: Passive heatsink usually sufficient

## Common Mistakes
1. Using `RPi.GPIO` on Pi 5 (doesn't work — use gpiozero/lgpio)
2. Underpowering Pi 5 (needs 5V/5A PD, not a random USB-C cable)
3. Forgetting to enable I2C/SPI in raspi-config
4. Running heavy AI workloads without active cooling
5. Not using a virtual environment for Python projects
6. Using micro-SD for write-heavy workloads (use NVMe on Pi 5 or USB SSD on Pi 4)
