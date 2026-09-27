# 3D Modelling & Parametric CAD Reference

## Table of Contents
1. CadQuery (Python — Primary Tool)
2. OpenSCAD
3. FreeCAD (GUI + Python Scripting)
4. Export Workflows (STL, STEP, OBJ, glTF)
5. Assembly Modelling
6. Design Patterns for Hardware Projects
7. Mechanical Components Library

---

## 1. CadQuery (Python — Recommended)

CadQuery is a Python parametric CAD library built on OCCT (Open CASCADE).
It produces BREP solids — real engineering geometry with fillets, chamfers,
and STEP export for manufacturing. Claude can generate CadQuery scripts directly.

### Installation
```bash
pip install cadquery
# Optional: Jupyter visualization
pip install jupyter-cadquery
# Optional: CLI viewer
pip install cq-editor
```

### Core Operations

#### Primitives
```python
import cadquery as cq

box = cq.Workplane("XY").box(50, 30, 10)
cylinder = cq.Workplane("XY").cylinder(height=20, radius=10)
sphere = cq.Workplane("XY").sphere(15)
```

#### Boolean Operations
```python
# Difference (cut)
result = box.cut(cylinder)

# Union (join)
result = box.union(cylinder)

# Intersection
result = box.intersect(cylinder)
```

#### Extrude from 2D Sketch
```python
# L-shaped bracket
bracket = (
    cq.Workplane("XY")
    .moveTo(0, 0)
    .lineTo(40, 0)
    .lineTo(40, 5)
    .lineTo(5, 5)
    .lineTo(5, 30)
    .lineTo(0, 30)
    .close()
    .extrude(3)  # 3mm thick
)
```

#### Fillets and Chamfers
```python
# Fillet all edges
box = cq.Workplane("XY").box(40, 20, 10).edges().fillet(2)

# Fillet specific edges (e.g., top edges only)
box = cq.Workplane("XY").box(40, 20, 10).edges("|Z").fillet(2)

# Chamfer
box = cq.Workplane("XY").box(40, 20, 10).edges(">Z").chamfer(1)
```

#### Holes
```python
# Through hole
plate = cq.Workplane("XY").box(50, 30, 5).faces(">Z").workplane().hole(6)

# Counterbore (M3 socket head cap screw)
plate = (cq.Workplane("XY").box(50, 30, 5)
    .faces(">Z").workplane()
    .cboreHole(3.4, 6.5, 3.5)  # through, cbore_dia, cbore_depth
)

# Countersink
plate = (cq.Workplane("XY").box(50, 30, 5)
    .faces(">Z").workplane()
    .cskHole(3.4, 6.5, 82)  # through, csk_dia, csk_angle
)

# Pattern of holes
plate = (cq.Workplane("XY").box(50, 30, 5)
    .faces(">Z").workplane()
    .pushPoints([(10, 5), (-10, 5), (10, -5), (-10, -5)])
    .hole(3.2)  # M3 clearance holes
)
```

#### Revolve (for round parts)
```python
# Lens housing (tube)
tube = (
    cq.Workplane("XZ")
    .moveTo(10, 0)   # Inner radius
    .lineTo(12, 0)   # Outer radius
    .lineTo(12, 20)  # Height
    .lineTo(10, 20)
    .close()
    .revolve(360)
)
```

#### Loft (variable cross-section)
```python
# Transition from square to circle
transition = (
    cq.Workplane("XY")
    .rect(20, 20)
    .workplane(offset=30)
    .circle(12)
    .loft()
)
```

#### Shell (hollow out)
```python
# Hollow box, open on top
box = cq.Workplane("XY").box(50, 30, 20).faces(">Z").shell(-2)
# -2 = 2mm wall thickness, negative = shell inward
```

#### Text
```python
# Embossed text
labeled = (
    cq.Workplane("XY").box(50, 20, 5)
    .faces(">Z").workplane()
    .text("V1.0", fontsize=8, distance=0.5)  # Raised 0.5mm
)

# Engraved text
labeled = (
    cq.Workplane("XY").box(50, 20, 5)
    .faces(">Z").workplane()
    .text("V1.0", fontsize=8, distance=-0.3, cut=True)
)
```

### Export
```python
# STL (3D printing)
cq.exporters.export(result, "part.stl")

# STEP (manufacturing, CAD exchange)
cq.exporters.export(result, "part.step")

# SVG (2D drawing from 3D)
cq.exporters.export(result, "part.svg")

# DXF (laser cutting profile)
sketch = cq.Workplane("XY").rect(50, 30).circle(10)
cq.exporters.export(sketch, "profile.dxf", exportType="DXF")
```

## 2. OpenSCAD

Declarative, CSG-based. Good for simple geometric parts.

### Core Syntax
```openscad
// Primitives
cube([50, 30, 10]);
cylinder(h=20, r=10, $fn=64);
sphere(r=15, $fn=64);

// Boolean operations
difference() {
    cube([50, 30, 10]);
    translate([25, 15, -1])
        cylinder(h=12, d=6, $fn=32);  // Through hole
}

union() {
    cube([50, 30, 10]);
    translate([25, 15, 10])
        cylinder(h=5, d=8, $fn=32);  // Boss
}

// Modules (reusable components)
module mounting_hole(d=3.2, h=10) {
    cylinder(d=d, h=h, $fn=24);
}

module mounting_pattern(spacing_x, spacing_y, d=3.2, h=10) {
    for (x = [-spacing_x/2, spacing_x/2])
        for (y = [-spacing_y/2, spacing_y/2])
            translate([x, y, 0])
                mounting_hole(d, h);
}

// Parametric design
wall = 2;
board_l = 85;
board_w = 56;
clearance = 0.4;

difference() {
    // Outer box with rounded corners
    minkowski() {
        cube([board_l + 2*(wall+clearance), board_w + 2*(wall+clearance), 20]);
        sphere(r=1, $fn=16);
    }
    // Inner cavity
    translate([wall, wall, wall])
        cube([board_l + 2*clearance, board_w + 2*clearance, 25]);
}
```

### Export from CLI
```bash
openscad -o output.stl input.scad
openscad -o output.png input.scad --camera=0,0,0,45,0,30,200  # Render preview
```

## 3. FreeCAD (GUI + Python Scripting)

For complex assemblies and when a GUI is needed alongside scripting.

### Python Scripting in FreeCAD
```python
import FreeCAD
import Part

# Create a box
box = Part.makeBox(50, 30, 10)

# Create a cylinder
cyl = Part.makeCylinder(5, 15)
cyl.translate(FreeCAD.Vector(25, 15, 0))

# Boolean difference
result = box.cut(cyl)

# Add fillets
result = result.makeFillet(2, result.Edges)

# Export
result.exportStep("part.step")
result.exportStl("part.stl")
```

### FreeCAD → glTF (for Three.js)
```
FreeCAD → Export as STEP → Open in Blender → Export as glTF/GLB
OR
FreeCAD → Export as OBJ → Load in Three.js with OBJLoader
```

## 4. Export Workflows

### Format Selection
| Format | Use Case | Quality | Notes |
|--------|----------|---------|-------|
| STL | 3D printing | Mesh (triangles) | No color, no units metadata |
| STEP | Manufacturing, CAD exchange | BREP (exact) | Industry standard, preserves geometry |
| OBJ | Rendering, Three.js | Mesh + materials | Supports UV maps, normals |
| glTF/GLB | Web 3D (Three.js) | Mesh + materials + animation | Modern web standard, compact |
| 3MF | Advanced 3D printing | Mesh + color + materials | Successor to STL |
| IGES | Legacy CAD exchange | BREP | Older, use STEP instead |
| DXF | Laser cutting (2D) | Vectors | 2D profiles only |

### CadQuery → Three.js Pipeline
```python
# Step 1: CadQuery → STEP
import cadquery as cq
result = cq.Workplane("XY").box(50, 30, 10).edges().fillet(2)
cq.exporters.export(result, "part.step")

# Step 2: STEP → glTF (use FreeCAD headless or Blender CLI)
# Blender CLI:
# blender --background --python convert.py
```

```python
# convert.py (Blender script)
import bpy
bpy.ops.wm.read_factory_settings(use_empty=True)
bpy.ops.import_scene.x3d(filepath="part.x3d")  # or OBJ
bpy.ops.export_scene.gltf(filepath="part.glb", export_format='GLB')
```

### CadQuery → STL Quality Settings
```python
# Higher tessellation for smoother curves
cq.exporters.export(result, "part.stl",
    tolerance=0.01,      # Linear tolerance (mm)
    angularTolerance=0.1  # Angular tolerance (radians)
)
```

## 5. Assembly Modelling

### CadQuery Assembly
```python
import cadquery as cq

# Create parts
base = cq.Workplane("XY").box(100, 60, 5)
pillar = cq.Workplane("XY").cylinder(height=30, radius=5)
top = cq.Workplane("XY").box(80, 40, 3)

# Assemble
assy = cq.Assembly()
assy.add(base, name="base", color=cq.Color("gray"))
assy.add(pillar, name="pillar_1",
    loc=cq.Location(cq.Vector(-30, -15, 5)),
    color=cq.Color("blue"))
assy.add(pillar, name="pillar_2",
    loc=cq.Location(cq.Vector(30, 15, 5)),
    color=cq.Color("blue"))
assy.add(top, name="top",
    loc=cq.Location(cq.Vector(0, 0, 35)),
    color=cq.Color("green"))

# Export entire assembly
assy.save("assembly.step")
```

## 6. Design Patterns for Hardware Projects

### Parametric Board Mount
```python
def board_mount(board_l, board_w, mount_holes, screw_d=3.2,
                wall=2, clearance=0.4, standoff_h=3):
    """Generate a universal board mounting base plate."""
    plate_l = board_l + 2*(wall + clearance)
    plate_w = board_w + 2*(wall + clearance)

    base = cq.Workplane("XY").box(plate_l, plate_w, wall)

    # Lip to hold board
    lip_h = standoff_h + 1.6  # PCB thickness
    base = (base.faces(">Z").workplane()
        .rect(plate_l, plate_w).rect(board_l + 2*clearance, board_w + 2*clearance)
        .extrude(lip_h)
    )

    # Standoffs
    for (x, y) in mount_holes:
        sx = x - board_l/2
        sy = y - board_w/2
        base = (base.faces(">Z").workplane(offset=-lip_h)
            .moveTo(sx, sy).circle(screw_d/2 + 1.5).extrude(standoff_h)
            .faces(">Z").workplane(offset=-lip_h-standoff_h)
            .moveTo(sx, sy).hole(screw_d - 0.4, standoff_h)
        )
    return base
```

### Hinge/Living Hinge (print-in-place)
```python
# Living hinge: thin section connecting two halves
# Works in PETG/TPU, minimal in PLA
# Design: 0.4mm thick × 3-5mm wide × full part width
# Print with 3 perimeters, 0° layer alignment
```

### Snap Fit Cantilever
```python
# Design rules:
# - Beam length: 10-15mm for good flex
# - Beam thickness: 1-2mm
# - Hook overhang: 0.5-1mm
# - Taper: 30-45° entry ramp
# - Material: PETG works well, PLA too stiff
```

## 7. Mechanical Components Library

### Standard Fastener Dimensions (for designing holes/pockets)

| Fastener | Shaft Ø | Head Ø | Head H | Clearance Hole | Close Fit |
|----------|---------|--------|--------|---------------|-----------|
| M2 | 2.0 | 3.8 | 1.5 | 2.4 | 2.2 |
| M2.5 | 2.5 | 4.5 | 1.7 | 2.9 | 2.7 |
| M3 | 3.0 | 5.5 | 2.0 | 3.4 | 3.2 |
| M4 | 4.0 | 7.0 | 2.6 | 4.5 | 4.3 |
| M5 | 5.0 | 8.5 | 3.35 | 5.5 | 5.3 |

### Heat-Set Insert Dimensions (for strong 3D printed threads)
| Insert | Shaft Ø | Hole Ø (print) | Depth | Tip Temp |
|--------|---------|---------------|-------|----------|
| M2 × 3mm | 2.0 | 3.0 | 3.5 | 220-250°C |
| M2.5 × 4mm | 2.5 | 3.5 | 4.5 | 220-250°C |
| M3 × 5mm | 3.0 | 4.0 | 5.5 | 220-250°C |
| M4 × 6mm | 4.0 | 5.2 | 6.5 | 220-250°C |

### Bearing Pockets
| Bearing | OD | ID | Width | Pocket Ø |
|---------|----|----|-------|----------|
| 608 (skateboard) | 22mm | 8mm | 7mm | 22.2mm |
| 625 | 16mm | 5mm | 5mm | 16.2mm |
| 623 | 10mm | 3mm | 4mm | 10.2mm |
| MR105 | 10mm | 5mm | 4mm | 10.2mm |
