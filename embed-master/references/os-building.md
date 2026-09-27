# Custom Embedded Linux / OS Building Reference

## Table of Contents
1. Buildroot (Recommended for Embedded)
2. Yocto / OpenEmbedded
3. Kernel Configuration
4. Device Trees
5. Root Filesystem Design
6. Boot Optimization
7. Read-Only Rootfs + Overlay
8. Custom Pi OS Images

---

## 1. Buildroot

Buildroot generates a complete embedded Linux system: cross-compiler, kernel,
bootloader, and root filesystem — all from source.

### Quick Start
```bash
git clone https://github.com/buildroot/buildroot.git
cd buildroot

# Use a default config for your board
make raspberrypi4_64_defconfig    # Pi 4
# Or: make raspberrypi3_64_defconfig, raspberrypi0w_defconfig
# Or: make stm32mp157c_dk2_defconfig (STM32MP1)

# Customize
make menuconfig    # Main config (packages, toolchain, kernel)
make linux-menuconfig    # Kernel config specifically

# Build (takes 30-60 min first time)
make -j$(nproc)

# Output: output/images/
#   sdcard.img — flash directly to SD card
#   rootfs.tar — root filesystem
#   zImage/Image — kernel
#   *.dtb — device tree blobs
```

### Key Menuconfig Options
```
Target options → Target Architecture: AArch64 (Pi 4/5), ARM (Pi 3/Zero)
Toolchain → C library: musl (smaller) or glibc (more compatible)
System configuration →
    Init system: BusyBox (tiny) or systemd (full)
    Root password: set it
    /dev management: devtmpfs + mdev (BusyBox) or eudev (systemd)
    Enable root login with password: yes
Kernel → Custom version: pick latest stable (6.x)
Target packages →
    Networking: dropbear (tiny SSH) or openssh
    Interpreters: python3, micropython
    Libraries: libgpiod, i2c-tools
    Hardware: can-utils, spi-tools
```

### Adding Custom Packages
Create `package/myapp/`:
```makefile
# package/myapp/myapp.mk
MYAPP_VERSION = 1.0
MYAPP_SITE = $(TOPDIR)/../myapp-src
MYAPP_SITE_METHOD = local

define MYAPP_BUILD_CMDS
    $(MAKE) CC="$(TARGET_CC)" -C $(@D)
endef

define MYAPP_INSTALL_TARGET_CMDS
    $(INSTALL) -D -m 0755 $(@D)/myapp $(TARGET_DIR)/usr/bin/myapp
endef

$(eval $(generic-package))
```

### Adding Custom Scripts / Files (Overlay)
```bash
# Create an overlay directory
mkdir -p board/myproject/rootfs-overlay/etc
echo "myproject" > board/myproject/rootfs-overlay/etc/hostname

# In menuconfig:
# System configuration → Root filesystem overlay: board/myproject/rootfs-overlay
```

## 2. Yocto / OpenEmbedded

Yocto is more complex but produces production-grade, maintainable distributions
with proper package management (opkg, dpkg, rpm).

### Quick Start
```bash
git clone -b scarthgap git://git.yoctoproject.org/poky
cd poky
source oe-init-build-env

# Edit conf/local.conf:
# MACHINE = "raspberrypi4-64"  (needs meta-raspberrypi layer)

# Build minimal image
bitbake core-image-minimal

# Output: tmp/deploy/images/<machine>/
```

### Adding BSP Layers
```bash
# For Raspberry Pi
git clone -b scarthgap https://github.com/agherzan/meta-raspberrypi.git
bitbake-layers add-layer ../meta-raspberrypi

# For STM32
git clone -b scarthgap https://github.com/STMicroelectronics/meta-st-stm32mp.git
```

### Custom Layer (for your project)
```bash
bitbake-layers create-layer meta-myproject
bitbake-layers add-layer meta-myproject

# Add recipes in: meta-myproject/recipes-myapp/myapp/myapp_1.0.bb
```

### When Buildroot vs Yocto
| Factor | Buildroot | Yocto |
|--------|-----------|-------|
| Learning curve | 1-2 days | 1-2 weeks |
| Build time | 30-60 min | 2-6 hours |
| Package management | No (static image) | Yes (opkg/dpkg/rpm) |
| OTA updates | Manual | swupdate, RAUC, Mender |
| Commercial support | Community | Many vendors |
| Reproducibility | Good | Excellent |
| Best for | Prototype → small production | Mass production |

## 3. Kernel Configuration

### Essential Kernel Options for Embedded
```bash
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- menuconfig
```

Key options to enable/disable:

**GPIO / I2C / SPI:**
```
Device Drivers → GPIO Support → [*] /sys/class/gpio (sysfs — legacy)
Device Drivers → I2C support → [*] I2C device interface (/dev/i2c-*)
Device Drivers → SPI support → [*] User mode SPI device driver (/dev/spidev*)
```

**PWM:**
```
Device Drivers → Pulse-Width Modulation (PWM) support → [*]
  → [*] BCM2835 PWM (Pi)
```

**Camera (Pi):**
```
Device Drivers → Multimedia support → [*] Media USB Adapters
  → [*] V4L2 sub-device userspace API
Device Drivers → Staging → [*] BCM2835 camera
```

**USB gadget (turn Pi into USB device):**
```
Device Drivers → USB support → USB Gadget Support → [*]
  → USB functions configurable through configfs
```

**Reduce kernel size:**
```
# Disable everything you don't need:
File systems → disable ext2, NTFS, CIFS, NFS if not used
Networking → disable IPv6, Bluetooth, NFC if not needed
Sound → disable ALSA if headless
```

### Building the Kernel Standalone
```bash
# For Raspberry Pi 4/5
git clone --depth 1 https://github.com/raspberrypi/linux.git
cd linux
KERNEL=kernel8
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- bcm2711_defconfig
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- menuconfig  # Customize
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j$(nproc) Image modules dtbs
```

## 4. Device Trees

Device trees describe the hardware to the Linux kernel. Essential for
custom boards or when enabling specific peripherals.

### Device Tree Overlay (Pi — easiest method)
```dts
// Enable I2C sensor at address 0x76 on I2C1
/dts-v1/;
/plugin/;

/ {
    compatible = "brcm,bcm2711";  // Pi 4

    fragment@0 {
        target = <&i2c1>;
        __overlay__ {
            status = "okay";
            #address-cells = <1>;
            #size-cells = <0>;

            bme280@76 {
                compatible = "bosch,bme280";
                reg = <0x76>;
            };
        };
    };
};
```

### Compile & Load Overlay
```bash
# Compile
dtc -@ -I dts -O dtb -o my-sensor.dtbo my-sensor.dts

# Load at boot (Pi): add to /boot/config.txt
dtoverlay=my-sensor

# Load at runtime
sudo dtoverlay my-sensor
```

### Common Pi Overlays (already available)
```ini
# /boot/config.txt
dtparam=i2c_arm=on       # Enable I2C1
dtparam=spi=on            # Enable SPI0
dtoverlay=pwm-2chan       # Enable hardware PWM on GPIO12/13
dtoverlay=uart2           # Enable UART2
dtoverlay=w1-gpio         # 1-Wire on GPIO4
dtoverlay=spi1-3cs        # Enable SPI1 with 3 chip selects
```

## 5. Root Filesystem Design

### Minimal Rootfs Structure
```
/
├── bin/         → BusyBox symlinks (sh, ls, cat, etc.)
├── dev/         → Device nodes (managed by devtmpfs)
├── etc/
│   ├── init.d/  → Startup scripts
│   ├── inittab  → BusyBox init config
│   ├── network/ → Network config
│   └── passwd   → User accounts
├── lib/         → Shared libraries (libc, libpthread)
├── proc/        → Procfs mount
├── sbin/        → System binaries (init, reboot)
├── sys/         → Sysfs mount
├── tmp/         → Temp (tmpfs)
├── usr/
│   ├── bin/     → User binaries (your app goes here)
│   └── lib/     → Python, etc.
└── var/         → Logs, runtime data
```

### Minimal Init Script (/sbin/init alternative)
```bash
#!/bin/sh
mount -t proc proc /proc
mount -t sysfs sysfs /sys
mount -t devtmpfs devtmpfs /dev

# Start your application
/usr/bin/myapp &

# Drop to shell (debug) or loop forever (production)
exec /bin/sh
```

## 6. Boot Optimization

| Technique | Time Saved | Difficulty |
|-----------|-----------|------------|
| Kernel: disable unused drivers | 2-5s | Easy |
| Kernel: built-in vs modules | 0.5-1s | Easy |
| BusyBox init instead of systemd | 3-5s | Easy |
| Quiet boot (no console output) | 0.5-1s | Easy |
| Compress rootfs (squashfs) | 1-2s | Medium |
| Kernel: deferred initcalls | 1-3s | Medium |
| U-Boot: Falcon mode (skip U-Boot) | 1-2s | Hard |
| Pre-built kernel (skip decompression) | 0.5s | Medium |

### Quick Wins
```ini
# /boot/cmdline.txt (Pi)
quiet loglevel=0 logo.nologo vt.global_cursor_default=0

# Disable unnecessary services
sudo systemctl disable bluetooth hciuart avahi-daemon triggerhappy
```

## 7. Read-Only Rootfs + Overlay

For embedded systems that must survive power loss without SD card corruption:

```bash
# In Buildroot menuconfig:
# Filesystem images → [*] read-only rootfs

# Or manually:
# /etc/fstab
/dev/mmcblk0p2  /        ext4  ro,noatime      0  1
tmpfs           /tmp     tmpfs defaults         0  0
tmpfs           /var/log tmpfs defaults         0  0
tmpfs           /var/run tmpfs defaults         0  0

# For writable config: use overlayfs
# Mount a writable partition over /etc or specific dirs
mount -t overlay overlay -o lowerdir=/etc,upperdir=/data/etc-upper,workdir=/data/etc-work /etc
```

## 8. Custom Pi OS Images

### Modify Existing Raspberry Pi OS
```bash
# On a Linux host:
# 1. Download Pi OS image
# 2. Mount it
sudo losetup -fP raspios.img
sudo mount /dev/loop0p2 /mnt/rootfs
sudo mount /dev/loop0p1 /mnt/rootfs/boot

# 3. Chroot into it (with QEMU for ARM emulation)
sudo cp /usr/bin/qemu-aarch64-static /mnt/rootfs/usr/bin/
sudo chroot /mnt/rootfs /bin/bash

# 4. Install packages, configure, add your app
apt install python3-opencv python3-flask
pip install your-project

# 5. Exit, unmount, flash
exit
sudo umount -R /mnt/rootfs
sudo losetup -d /dev/loop0
# Flash modified image to SD card
```

### pi-gen (build Pi OS from scratch)
```bash
git clone https://github.com/RPi-Distro/pi-gen.git
cd pi-gen
cat > config <<EOF
IMG_NAME=myproject-os
FIRST_USER_NAME=pi
FIRST_USER_PASS=mypassword
ENABLE_SSH=1
LOCALE_DEFAULT=en_US.UTF-8
TARGET_HOSTNAME=mydevice
EOF

# Customize stage2 (minimal) or stage3+ (desktop)
# Add packages to stage2/01-sys-tweaks/00-packages
echo "python3-opencv" >> stage2/01-sys-tweaks/00-packages

# Build
./build-docker.sh  # Needs Docker
# Output: deploy/image_*.zip
```
