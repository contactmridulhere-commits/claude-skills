# 3D Printing for Electronics — Enclosures & Mechanical Parts

## Table of Contents
1. Design-for-Print Rules
2. CadQuery Patterns (Python)
3. OpenSCAD Patterns
4. Board Dimensions Reference
5. Slicer Settings
6. Material Selection
7. Common Enclosure Features

---

## 1. Design-for-Print Rules

### Tolerances
| Feature | Clearance to Add |
|---------|-----------------|
| Board slot/pocket | +0.3 to 0.5mm per side |
| Screw hole (M3) | 3.2-3.4mm diameter |
| Screw hole (M2.5) | 2.7-2.9mm diameter |
| Press-fit post (M3) | 2.8mm diameter (screw taps into plastic) |
| Snap-fit clip | +0.2mm gap |
| USB/HDMI port cutout | +0.5mm per side |
| Lid-to-body fit | +0.2mm per side |
| Cable grommet hole | +0.3mm diameter |

### Printability Rules
- **Minimum wall thickness**: 1.2mm (3 perimeters at 0.4mm nozzle)
- **Minimum feature size**: 0.8mm
- **Overhangs**: Up to 45° without supports, 60° with good cooling
- **Bridges**: Up to 20mm reliable, 50mm possible with slow speed + cooling
- **Avoid supports** when possible — design with chamfers and self-supporting angles
- **Orientation matters**: Print the largest flat surface down
- **Screw bosses**: 6mm outer diameter for M3, 5mm for M2.5, with 2mm min wall

### Electronics-Specific Rules
- **Ventilation slots**: 1.5mm wide, spaced 3mm apart, for heat-generating boards
- **Cable routing**: Include channels (5×5mm minimum) for wire management
- **Antenna clearance**: No plastic within 5mm of WiFi antennas (ESP32, Pi)
- **Display windows**: Use transparent filament or leave open cutout
- **LED light pipes**: Design 3mm diameter holes, print in clear PLA/PETG
- **Board standoffs**: 3mm height minimum (keeps solder joints off the floor)
- **Snap-fit retention**: Slight taper on clips (1-2°) for easier assembly

## 2. CadQuery Patterns (Python, Parametric)

CadQuery is the recommended tool — parametric, scriptable, Claude can generate
enclosure code directly.

### Installation
```bash
pip install cadquery
# For visualization (optional):
pip install jupyter-cadquery
```

### Basic Enclosure Template
```python
import cadquery as cq

# === PARAMETERS (edit these) ===
BOARD_L = 68.6    # Board length (mm)
BOARD_W = 53.4    # Board width (mm)
BOARD_H = 15.0    # Component height above board (mm)
WALL = 2.0        # Wall thickness
CLEARANCE = 0.4   # Board clearance per side
STANDOFF_H = 3.0  # Board standoff height
SCREW_D = 3.2     # Screw hole diameter (M3)

# Internal dimensions
INT_L = BOARD_L + 2 * CLEARANCE
INT_W = BOARD_W + 2 * CLEARANCE
INT_H = BOARD_H + STANDOFF_H + 2.0  # Extra headroom

# === BOTTOM CASE ===
bottom = (
    cq.Workplane("XY")
    .box(INT_L + 2*WALL, INT_W + 2*WALL, INT_H + WALL)
    .edges("|Z").fillet(2.0)  # Rounded corners
    # Hollow out from top
    .faces(">Z").workplane()
    .rect(INT_L, INT_W)
    .cutBlind(-INT_H)
)

# Add mounting standoffs
standoff_positions = [
    (3.0, 3.0), (3.0, BOARD_W - 3.0),
    (BOARD_L - 3.0, 3.0), (BOARD_L - 3.0, BOARD_W - 3.0)
]
# Adjust positions relative to center
for pos in standoff_positions:
    x = pos[0] - BOARD_L/2 + CLEARANCE
    y = pos[1] - BOARD_W/2 + CLEARANCE
    bottom = (
        bottom.faces("<Z").workplane(offset=WALL)
        .moveTo(x, y)
        .circle(SCREW_D/2 + 1.5)  # Boss outer diameter
        .extrude(STANDOFF_H)
        .faces("<Z").workplane(offset=WALL)
        .moveTo(x, y)
        .hole(SCREW_D - 0.4, STANDOFF_H)  # Press-fit hole
    )

# Export
cq.exporters.export(bottom, "enclosure_bottom.stl")
```

### Adding Port Cutouts
```python
# USB-C port cutout on front face
USB_W = 9.5    # USB-C width
USB_H = 3.5    # USB-C height
USB_Y_OFFSET = STANDOFF_H + 1.5  # Height from internal bottom

bottom = (
    bottom.faces(">X").workplane()  # Front face
    .moveTo(0, -INT_H/2 + WALL + USB_Y_OFFSET + USB_H/2)
    .rect(USB_W + 1.0, USB_H + 1.0)  # +1mm clearance
    .cutThruAll()
)

# Circular hole (e.g., barrel jack or button)
bottom = (
    bottom.faces(">X").workplane()
    .moveTo(15, -INT_H/2 + WALL + 8)
    .hole(6.5)  # 6.5mm for standard barrel jack
)
```

### Snap-Fit Lid
```python
LID_OVERLAP = 3.0  # How deep lid sits into case
LIP = 1.2          # Lip width for snap fit

# Lid
lid = (
    cq.Workplane("XY")
    .box(INT_L + 2*WALL, INT_W + 2*WALL, WALL)
    .edges("|Z").fillet(2.0)
)

# Add inner lip that slides into the case
lid = (
    lid.faces("<Z").workplane()
    .rect(INT_L - 0.4, INT_W - 0.4)  # 0.2mm gap per side
    .extrude(LID_OVERLAP)
)

# Add ventilation slots on top
lid = (
    lid.faces(">Z").workplane()
    .rarray(4, 1, 8, 1)  # 8 slots, 4mm spacing
    .slot2D(15, 1.5)     # 15mm long, 1.5mm wide
    .cutThruAll()
)

cq.exporters.export(lid, "enclosure_lid.stl")
```

### Raspberry Pi 4/5 Enclosure
```python
# Pi 4/5 specific dimensions
PI_L = 85.0
PI_W = 56.0
PI_MOUNT_HOLES = [  # Relative to bottom-left corner
    (3.5, 3.5), (3.5, 52.5),
    (61.5, 3.5), (61.5, 52.5)
]
PI_SCREW = 2.7  # M2.5

# Port cutouts (measured from board edge):
# USB-A × 2: right side, 29mm and 47mm from front, 16mm wide × 16mm tall
# Ethernet: right side, 10mm from front, 16mm wide × 14mm tall
# USB-C power: front, 3.5mm from left, 9mm wide × 3.5mm tall
# micro-HDMI × 2: front, 26mm and 39.5mm from left
# SD card: back, centered, slot opening
# GPIO header: top, 40-pin cutout if needed
```

## 3. OpenSCAD Patterns

For those who prefer OpenSCAD's declarative style:

```openscad
// Basic enclosure
board_l = 68.6;
board_w = 53.4;
wall = 2;
clearance = 0.4;

module enclosure_bottom() {
    difference() {
        // Outer shell with rounded corners
        minkowski() {
            cube([board_l + 2*(wall+clearance) - 4,
                  board_w + 2*(wall+clearance) - 4, 20]);
            cylinder(r=2, h=0.01, $fn=32);
        }
        // Inner cavity
        translate([wall, wall, wall])
            cube([board_l + 2*clearance,
                  board_w + 2*clearance, 20]);
    }
    // Standoffs
    for (pos = [[5,5], [5,48], [63,5], [63,48]])
        translate([pos[0]+wall+clearance, pos[1]+wall+clearance, wall])
            difference() {
                cylinder(d=6, h=3, $fn=24);
                cylinder(d=2.5, h=3, $fn=24);
            }
}

enclosure_bottom();
```

```bash
# Export from command line
openscad -o enclosure.stl enclosure.scad
```

## 4. Board Dimensions Reference

| Board | L × W (mm) | Mount Holes | Screw | Hole Spacing (mm) |
|-------|-----------|-------------|-------|-------------------|
| Arduino Uno | 68.6 × 53.4 | 4 × M3 | 3.2mm | Various (non-uniform) |
| Arduino Nano | 45 × 18 | None | — | Breadboard-mounted |
| Arduino Mega | 101.5 × 53.4 | 4 × M3 | 3.2mm | Various |
| ESP32 DevKit V1 | 51 × 28 | None | — | Breadboard-mounted |
| ESP32-CAM | 40 × 27 | None | — | |
| ESP8266 NodeMCU | 58 × 31 | None | — | Breadboard-mounted |
| Raspberry Pi 4/5 | 85 × 56 | 4 × M2.5 | 2.7mm | 58 × 49 |
| Raspberry Pi 3B+ | 85 × 56 | 4 × M2.5 | 2.7mm | 58 × 49 |
| Raspberry Pi Pico | 51 × 21 | 4 × M2 | 2.2mm | 47 × 11.4 |
| Raspberry Pi Zero | 65 × 30 | 4 × M2.5 | 2.7mm | 58 × 23 |

### Common Port Dimensions (for cutouts)
| Port | Width × Height (mm) | Clearance |
|------|---------------------|-----------|
| USB-A | 13 × 6 | +0.5mm each side |
| USB-C | 9 × 3.5 | +0.5mm each side |
| Micro-USB | 8 × 3 | +0.5mm each side |
| Micro-HDMI | 7 × 3.5 | +0.5mm each side |
| Full HDMI | 15 × 5 | +0.5mm each side |
| Ethernet (RJ45) | 16 × 14 | +0.5mm each side |
| Barrel jack (5.5mm) | 6.5mm hole | +0.3mm |
| 3.5mm audio | 3.8mm hole | +0.2mm |
| SD card slot | 12 × 2.5 | +0.5mm |

## 5. Slicer Settings for Electronics Enclosures

### Recommended Defaults (PLA)
| Setting | Value | Reason |
|---------|-------|--------|
| Layer height | 0.2mm | Balance of quality and speed |
| First layer | 0.28mm | Better adhesion |
| Perimeters | 3 | Structural integrity |
| Top/bottom layers | 4 | Solid top surface |
| Infill | 20-30% | Grid or Gyroid |
| Print speed | 50mm/s | Reliable quality |
| Temperature | 205-215°C | PLA range |
| Bed temp | 60°C | PLA adhesion |
| Cooling fan | 100% after first layer | Better overhangs |
| Supports | Only if needed | Design to avoid |
| Brim | 3-5mm for large flat parts | Prevent warping |

### PETG Adjustments
- Temp: 230-245°C, Bed: 70-80°C
- Reduce cooling fan to 50-70%
- Increase retraction slightly
- PETG strings more — increase travel speed

## 6. Material Selection

| Material | Temp Resistance | Strength | Flexibility | Use Case |
|----------|----------------|----------|-------------|----------|
| PLA | ~60°C | Good | Brittle | Prototypes, indoor projects |
| PETG | ~80°C | Better | Slight flex | Near heat sources, outdoor |
| ABS | ~100°C | Good | Some flex | High temp (motor drivers, PSUs) |
| ASA | ~100°C | Good | Some flex | Outdoor, UV-resistant |
| TPU | ~80°C | Flexible | Very flexible | Grommets, bumpers, gaskets |
| Nylon | ~100°C+ | Excellent | Tough | Gears, structural parts |

**Default choice**: PLA for prototypes, PETG for anything permanent or near heat.

## 7. Common Enclosure Features

### Feature Library (CadQuery snippets)

**Ventilation grid**:
```python
# Add to top face
lid = lid.faces(">Z").workplane()
for i in range(-20, 21, 4):
    lid = lid.moveTo(i, 0).slot2D(15, 1.5).cutThruAll()
```

**Cable strain relief**:
```python
# Add zip-tie anchor slot inside case
bottom = (bottom.faces("<Z").workplane(offset=WALL)
    .moveTo(-INT_L/2 + 10, 0)
    .rect(5, 3).extrude(8)  # Post
    .faces("<Z").workplane(offset=WALL + 4)
    .moveTo(-INT_L/2 + 10, 0)
    .rect(5, 1.5).cutBlind(-3)  # Zip-tie slot
)
```

**Camera mount (Pi Camera)**:
```python
# Camera module v2: 25 × 23.86mm, lens at center
CAM_W = 25.0
CAM_H = 23.86
CAM_MOUNT_HOLES = [(10.5, 0), (-10.5, 0)]  # 21mm spacing, M2

cam_mount = (
    cq.Workplane("XY")
    .box(CAM_W + 6, CAM_H + 6, 3)  # Mounting plate
    .faces(">Z").workplane()
    .pushPoints(CAM_MOUNT_HOLES)
    .hole(2.2, 3)  # M2 holes
    .faces(">Z").workplane()
    .moveTo(0, 0)
    .hole(8.0, 3)  # Lens opening
)
```

**Magnetic lid attachment**:
- Design 6mm diameter × 2.5mm deep pockets for 6×2mm neodymium magnets
- Press-fit, add tiny dot of superglue
- Place in matching positions on lid and body
