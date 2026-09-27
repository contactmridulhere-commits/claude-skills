# Waveguide Optics & AR Display Engineering Reference

## Table of Contents
1. Optical Fundamentals for AR
2. Waveguide Types & Operating Principles
3. Diffractive Waveguide Design
4. Holographic Optical Elements (HOE)
5. Micro-Projector / Light Engines
6. Beam Splitter & Birdbath Combiners
7. Optical Component Sourcing
8. Prototyping AR Displays
9. Ray Tracing & Simulation
10. Common Specifications & Targets

---

## 1. Optical Fundamentals for AR

### Key Parameters

| Parameter | Definition | Typical AR Target |
|-----------|-----------|-------------------|
| Field of View (FoV) | Angular extent of visible image | 30-50° diagonal |
| Eye box | Area where full image is visible | 8-15mm × 8-15mm |
| Eye relief | Distance from optic to eye | 15-20mm |
| Angular resolution | Pixels per degree | 30-60 PPD (human acuity ~60) |
| Luminance | Brightness of virtual image | >500 nits outdoor, >2000 ideal |
| Transparency | How clear the real world appears | >70% for usability |
| MTF (Modulation Transfer Function) | Image sharpness metric | >0.3 at Nyquist |
| Uniformity | Brightness consistency across FoV | >70% center-to-edge |
| Color gamut | Range of displayable colors | >sRGB for quality |

### Total Internal Reflection (TIR)
The fundamental principle behind waveguides. Light enters a glass slab and
bounces internally when hitting the surface at angles beyond the critical angle:

```
θ_critical = arcsin(n₂ / n₁)

For glass (n=1.5) in air (n=1.0):
θ_c = arcsin(1.0 / 1.5) = 41.8°

Light hitting the glass-air boundary at >41.8° from normal reflects totally.
```

This means light can travel through a thin glass plate by bouncing back and
forth — the waveguide principle.

### Snell's Law
```
n₁ sin(θ₁) = n₂ sin(θ₂)

Used to calculate refraction at every surface transition.
```

### Diffraction Grating Equation
```
d(sin θ_m - sin θ_i) = mλ

d = grating period (nm)
θ_i = incident angle
θ_m = diffracted angle (order m)
m = diffraction order (usually m=1)
λ = wavelength (nm)

For AR: choose d so that the m=1 diffracted beam enters TIR in the waveguide.
```

## 2. Waveguide Types

### Surface Relief Grating (SRG) Waveguide
- **How**: Nanoscale ridges etched into glass surface diffract light in/out
- **Used by**: Microsoft HoloLens 2, Magic Leap 2
- **Pros**: Mass-producible (nanoimprint lithography), durable
- **Cons**: Rainbow artifacts (chromatic dispersion), efficiency ~1-5% per bounce
- **Typical specs**: 0.5-1.5mm thick glass, 300-600nm grating pitch
- **FoV**: Up to ~50° with exit pupil expansion (EPE)

### Volume Holographic Waveguide
- **How**: Volume holograms (Bragg gratings) recorded in photopolymer film
- **Used by**: Sony, Digilens, Vuzix
- **Pros**: Wavelength-selective (less rainbow), higher efficiency per interaction
- **Cons**: Angular bandwidth limited, humidity/temp sensitivity
- **Recording**: Requires laser interference setup (coherent light)
- **Materials**: Photopolymer films (Covestro Bayfol HX, Digilens proprietary)

### Geometric Waveguide (Partially Reflective Mirrors)
- **How**: Array of semi-reflective surfaces (mirrors) embedded in glass at angle
- **Used by**: Lumus, some military HUDs
- **Pros**: No diffractive artifacts, good color uniformity
- **Cons**: Thicker, visible mirror edges, complex manufacturing
- **FoV**: 30-40° typical

### Pin-Mirror / Cascaded Mirror Array
- **How**: Tiny reflective elements arranged to expand pupil
- **Pros**: Very thin possible, no chromatic artifacts
- **Cons**: Complex fabrication, limited FoV

## 3. Diffractive Waveguide Design

### Architecture
```
Light Engine → Collimating Lens → In-Coupler Grating
    → TIR propagation through waveguide →
    → Exit Pupil Expander (optional, 1D or 2D) →
    → Out-Coupler Grating → User's Eye
```

### In-Coupler Design
- Purpose: Couple collimated light into waveguide at TIR angles
- Grating pitch: Calculated to diffract m=1 into TIR range
- Size: Matches light engine output aperture (~3-5mm)
- Efficiency: Higher = brighter image, but limits transparency

### Exit Pupil Expansion (EPE)
- **1D EPE**: Grating that partially diffracts light at each bounce, creating
  multiple copies along one axis → expands eye box horizontally
- **2D EPE**: Two-stage expansion (horizontal then vertical) or 2D grating
  for simultaneous expansion → fills full eye box
- Tradeoff: More expansion = dimmer image (light split across more copies)

### Grating Pitch Calculation Example
```python
import numpy as np

# Design parameters
wavelength = 532e-9  # Green laser, 532nm
n_glass = 1.7        # High-index glass (e.g., Schott SF6)
n_air = 1.0

# Critical angle for TIR
theta_c = np.arcsin(n_air / n_glass)  # ~36°

# We want first-order diffraction to enter TIR
# For normal incidence (θ_i = 0):
# d * sin(θ_m) = λ  →  d = λ / sin(θ_m)
# θ_m must be > θ_c for TIR

# Choose θ_m = 50° (well into TIR)
theta_m = np.radians(50)
d = wavelength / (n_glass * np.sin(theta_m))
pitch_nm = d * 1e9

print(f"Grating pitch: {pitch_nm:.0f} nm")
print(f"Lines/mm: {1e6/pitch_nm:.0f}")
# ~408nm pitch, ~2450 lines/mm
```

### Wavelength Multiplexing (Color)
For full-color AR, three approaches:
1. **Three stacked waveguides** (one per R/G/B) — HoloLens approach, thick but clean
2. **Single waveguide, time-sequential color** — flash R, G, B rapidly from projector
3. **Single waveguide, broadband grating** — challenging, rainbow artifacts worse

## 4. Holographic Optical Elements (HOE)

### Recording a Transmission HOE
```
Laser (coherent) → Beam Splitter →
    Reference beam → Mirror → Photopolymer Film ← Object beam → Lens/Mirror setup
```

The interference pattern is recorded in the photopolymer. When replayed with
similar light, it reconstructs the object wavefront.

### Key HOE Parameters
| Parameter | Typical Value | Notes |
|-----------|--------------|-------|
| Diffraction efficiency | 80-95% (peak) | At design wavelength and angle |
| Angular bandwidth | 5-15° | Limits FoV and eye box |
| Spectral bandwidth | 20-50nm | Color-selective |
| Thickness | 10-30µm (film) | Thicker = narrower bandwidth but higher efficiency |
| Multiplexing | 2-3 holograms per layer | Multiple wavelengths or angles |

### DIY HOE Recording (Simplified)
1. **Laser**: Diode laser (cheap) or HeNe (better coherence), single wavelength
2. **Beam splitter**: Cube or plate, 50/50 split
3. **Spatial filter**: Microscope objective + pinhole → clean beam
4. **Photopolymer**: Covestro Bayfol HX (commercially available) or dichromated gelatin
5. **Vibration isolation**: Optical table or heavy granite slab (critical!)
6. **Exposure**: Typically 10-100 mJ/cm², expose 5-60 seconds
7. **Post-processing**: UV cure + bake (depends on material)

## 5. Micro-Projector / Light Engines

### Technology Comparison

| Technology | Resolution | Brightness (lm) | Power | Size | Coherence |
|-----------|-----------|-----------------|-------|------|-----------|
| LBS (MEMS mirror) | 720p-1080p | 20-50 | 200-500mW | 5×5×3mm | Laser (high) |
| LCoS | Up to 4K | 5-30 | 300-800mW | 10×15mm | LED/Laser |
| DLP (DMD) | 720p-1080p | 20-100 | 500mW-1W | 15×20mm | LED |
| Micro-LED | 640×480 to 1080p | 50-200 | 100-400mW | 5×10mm | Incoherent |

### LBS (Laser Beam Scanning) — Best for Waveguides
- MEMS mirror scans a laser beam across the field
- Always in focus (no focal plane) — projects at infinity
- Compact: STMicro laser projector modules, MicroVision/Bosch MEMS
- Requires laser safety consideration (Class 1 at output)
- **Key components**: RGB laser diodes + MEMS scanner + driver IC + collimation optics

### Collimation
The projector output must be collimated (parallel light) before entering the
waveguide. The light appears to come from infinity → user's eye focuses it.

```
Point source → Collimating lens (f = working distance)
Focal length selection: f determines the virtual image distance and magnification.
Shorter f = wider FoV but harder to achieve uniform quality.
```

## 6. Beam Splitter & Birdbath Combiners

### Birdbath Architecture
```
Display (facing up) → Beam Splitter (45°, partially reflective)
    → Curved combiner (concave mirror with partial reflectivity)
    → User's eye (sees virtual image + real world through combiner)
```

- **Simplest AR combiner to prototype**
- Curved mirror focuses the display image at a virtual distance
- 50/50 beam splitter costs ~50% of display brightness
- Combiner curvature determines FoV and focal distance
- Used in: Nreal/Xreal Light, some HUD displays

### Beam Splitter Specs for Prototyping
| Type | Size | R/T Ratio | Price |
|------|------|-----------|-------|
| Plate beam splitter | 25×25mm | 50/50 | ~$20-50 |
| Cube beam splitter | 20mm | 50/50 | ~$30-80 |
| Pellicle beam splitter | 50mm | 50/50 | ~$50-100 |
| Hot mirror (IR reflect) | 25×25mm | Transmit visible | ~$30 |

## 7. Optical Component Sourcing

| Component | Supplier | Notes |
|-----------|----------|-------|
| Beam splitters, lenses, mirrors | Thorlabs, Edmund Optics | Professional, well-documented |
| Budget optics (lenses, prisms) | AliExpress, Surplus Shed | Cheap prototyping |
| Diffraction gratings (educational) | Rainbow Symphony, AliExpress | For learning, low quality |
| Photopolymer (HOE recording) | Covestro Bayfol HX | Professional holographic film |
| High-index glass (n>1.7) | Schott, Ohara | For waveguides |
| MEMS mirrors | STMicro, Mirrorcle | LBS projectors |
| Micro-displays (LCoS) | Himax, Syndiant | OEM modules |
| Laser diodes (RGB) | Osram, Nichia | For LBS light engines |
| Pi Camera / USB cameras | Raspberry Pi, ArduCam | See-through AR camera passthrough |

## 8. Prototyping AR Displays

### Beginner: Pepper's Ghost (No Waveguide)
```
Phone/OLED display (face down) → 45° angled clear plastic/glass → User sees reflection
overlaid on real world behind the glass
```
Not true AR (fixed viewing angle, no depth) but great for demo/proof-of-concept.

### Intermediate: Beam Splitter HUD
1. Small OLED/LCD display (1.3" SPI OLED or 2" TFT)
2. Convex lens (magnifies display, sets virtual distance)
3. 50/50 beam splitter at 45°
4. Mount everything in a 3D-printed housing
5. Connect display to Pi/ESP32 for rendering

### Advanced: DIY Waveguide Prototype
1. High-index glass plate (1-2mm, n>1.5)
2. Transmission diffraction grating film (1000+ lines/mm)
3. Laser pointer or laser diode module
4. Optical bench / 3D-printed alignment jig
5. Procedure: Attach grating as in-coupler on one end, observe TIR propagation,
   attach second grating as out-coupler, align to extract light toward eye position

### Software for AR Display Prototyping
- **Python + OpenCV**: Camera passthrough AR on Pi (capture → overlay → display)
- **Three.js + WebXR**: Browser-based AR rendering
- **Unity + MRTK**: If targeting existing AR headsets
- **Custom renderer on Pi**: OpenGL ES / Vulkan for direct display rendering

## 9. Ray Tracing & Simulation

### Python Ray Tracing (Simple 2D)
```python
import numpy as np
import matplotlib.pyplot as plt

def snell(theta_i, n1, n2):
    """Calculate refracted angle using Snell's law."""
    sin_t = (n1 / n2) * np.sin(theta_i)
    if abs(sin_t) > 1:
        return None  # Total internal reflection
    return np.arcsin(sin_t)

def trace_waveguide(theta_in, n_glass, thickness, length):
    """Trace a ray through a flat waveguide, return bounce positions."""
    theta_glass = snell(theta_in, 1.0, n_glass)
    if theta_glass is None:
        return []

    # Check TIR condition
    theta_critical = np.arcsin(1.0 / n_glass)
    if theta_glass < theta_critical:
        return []  # Escapes at first surface

    bounces = []
    x = 0
    z = 0  # Start at bottom surface
    going_up = True
    dz = thickness
    dx = dz / np.tan(np.pi/2 - theta_glass)

    while x < length:
        x += dx
        z = thickness if going_up else 0
        bounces.append((x, z))
        going_up = not going_up

    return bounces, theta_glass

# Example: trace ray in 1.7 index glass
bounces, angle = trace_waveguide(
    theta_in=np.radians(30),  # 30° input
    n_glass=1.7,
    thickness=1.5,  # mm
    length=30  # mm
)
```

### Professional Simulation Tools
| Tool | Type | Price | Notes |
|------|------|-------|-------|
| Zemax OpticStudio | Ray tracing | $$$$ (academic license available) | Industry standard |
| Code V | Ray tracing | $$$$ | Synopsys, AR-specific features |
| LightTools | Illumination | $$$$ | Luminit/Synopsys |
| TracePro | Ray tracing | $$$ | Lambda Research |
| Ansys Lumerical | FDTD / wave optics | $$$$ | Nanophotonics simulation |
| Python + raytracing | Basic 2D/3D | Free | Good for learning, limited |

## 10. Common Specifications & Targets

### Minimum Viable AR Display (DIY)
| Spec | Target | Notes |
|------|--------|-------|
| FoV | 15-20° diagonal | Achievable with simple optics |
| Resolution | 640×480 | Small OLED/LCoS display |
| Brightness | >200 nits | Usable indoors |
| Eye relief | 15-25mm | Must accommodate glasses |
| Weight (optics only) | <30g | Comfort threshold |
| Transparency | >50% | Must see real world |

### Consumer AR Glasses Target (Ambitious)
| Spec | Target | Notes |
|------|--------|-------|
| FoV | 40-50° diagonal | Requires waveguide |
| Resolution | 1080p per eye | LCoS or Micro-LED |
| Brightness | >2000 nits | Outdoor visibility |
| Eye box | 12×12mm | Comfortable viewing |
| Weight (total) | <80g | All-day wearable |
| Battery life | >3 hours | 2000-3000mAh, efficient optics |
