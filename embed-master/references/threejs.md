# Three.js / WebGL 3D Visualization Reference

## Table of Contents
1. Scene Setup (Boilerplate)
2. Geometries & Materials
3. Lighting
4. Loading 3D Models (GLTF/OBJ)
5. Animation & Interaction
6. Real-Time Sensor Data Visualization
7. Digital Twin Patterns
8. Hardware Dashboard Integration
9. Performance Optimization
10. WebXR (AR/VR in Browser)

---

## 1. Scene Setup (Boilerplate)

### Minimal Three.js Scene
```html
<!DOCTYPE html>
<html>
<head>
    <style>
        body { margin: 0; overflow: hidden; background: #0a0a0a; }
        canvas { display: block; }
    </style>
</head>
<body>
<script type="importmap">
{
    "imports": {
        "three": "https://cdn.jsdelivr.net/npm/three@0.160/build/three.module.js",
        "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.160/examples/jsm/"
    }
}
</script>
<script type="module">
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

// Scene
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x1a1a2e);

// Camera
const camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 1000);
camera.position.set(5, 4, 6);

// Renderer
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setPixelRatio(window.devicePixelRatio);
renderer.shadowMap.enabled = true;
document.body.appendChild(renderer.domElement);

// Controls
const controls = new OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;
controls.dampingFactor = 0.05;

// Lighting
const ambientLight = new THREE.AmbientLight(0x404040, 2);
scene.add(ambientLight);

const dirLight = new THREE.DirectionalLight(0xffffff, 3);
dirLight.position.set(5, 10, 7);
dirLight.castShadow = true;
scene.add(dirLight);

// Grid helper
scene.add(new THREE.GridHelper(10, 10, 0x444444, 0x222222));

// Animation loop
function animate() {
    requestAnimationFrame(animate);
    controls.update();
    renderer.render(scene, camera);
}
animate();

// Handle resize
window.addEventListener('resize', () => {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
});
</script>
</body>
</html>
```

### React / JSX Setup
```jsx
import { useEffect, useRef } from 'react';
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';

export default function Scene3D() {
    const containerRef = useRef(null);

    useEffect(() => {
        const container = containerRef.current;
        const scene = new THREE.Scene();
        const camera = new THREE.PerspectiveCamera(60, container.clientWidth / container.clientHeight, 0.1, 1000);
        camera.position.set(5, 4, 6);

        const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
        renderer.setSize(container.clientWidth, container.clientHeight);
        container.appendChild(renderer.domElement);

        const controls = new OrbitControls(camera, renderer.domElement);
        controls.enableDamping = true;

        // Add content...
        const animate = () => {
            requestAnimationFrame(animate);
            controls.update();
            renderer.render(scene, camera);
        };
        animate();

        return () => {
            renderer.dispose();
            container.removeChild(renderer.domElement);
        };
    }, []);

    return <div ref={containerRef} style={{ width: '100%', height: '500px' }} />;
}
```

## 2. Geometries & Materials

### Common Geometries for Hardware Visualization
```javascript
// PCB board
const pcb = new THREE.Mesh(
    new THREE.BoxGeometry(8.5, 0.16, 5.6),  // Pi 4 dimensions in cm
    new THREE.MeshStandardMaterial({ color: 0x006600, roughness: 0.8 })
);

// Cylindrical component (capacitor, motor)
const capacitor = new THREE.Mesh(
    new THREE.CylinderGeometry(0.3, 0.3, 0.8, 16),
    new THREE.MeshStandardMaterial({ color: 0x222222, metalness: 0.5 })
);

// IC chip
const chip = new THREE.Mesh(
    new THREE.BoxGeometry(1, 0.2, 1),
    new THREE.MeshStandardMaterial({ color: 0x111111, metalness: 0.3 })
);

// Wire (use TubeGeometry along a path)
const wirePath = new THREE.CatmullRomCurve3([
    new THREE.Vector3(0, 0, 0),
    new THREE.Vector3(1, 0.5, 0),
    new THREE.Vector3(2, 0.3, 1),
    new THREE.Vector3(3, 0, 1),
]);
const wire = new THREE.Mesh(
    new THREE.TubeGeometry(wirePath, 20, 0.03, 8, false),
    new THREE.MeshStandardMaterial({ color: 0xff0000 })  // Red wire
);

// LED (glowing sphere)
const ledMat = new THREE.MeshStandardMaterial({
    color: 0x00ff00, emissive: 0x00ff00, emissiveIntensity: 2
});
const led = new THREE.Mesh(new THREE.SphereGeometry(0.15, 16, 16), ledMat);
```

### Material Types
| Material | Use Case | Performance |
|----------|----------|-------------|
| MeshBasicMaterial | Unlit, solid color, wireframe | Fastest |
| MeshStandardMaterial | PBR, realistic, metalness/roughness | Good |
| MeshPhysicalMaterial | Glass, clearcoat, transmission | Slower |
| MeshNormalMaterial | Debug, shows face normals | Fast |
| MeshToonMaterial | Cartoon/stylized look | Good |

## 3. Lighting

```javascript
// Ambient (global fill)
scene.add(new THREE.AmbientLight(0x404040, 1.5));

// Directional (sun-like, parallel rays, casts shadows)
const sun = new THREE.DirectionalLight(0xffffff, 2);
sun.position.set(5, 10, 7);
sun.castShadow = true;
sun.shadow.mapSize.set(1024, 1024);
scene.add(sun);

// Point light (LED indicator glow)
const ledGlow = new THREE.PointLight(0x00ff00, 1, 3);
ledGlow.position.set(0, 0.5, 0);
scene.add(ledGlow);

// Spot light (focused beam, like a flashlight)
const spot = new THREE.SpotLight(0xffffff, 2, 10, Math.PI / 6);
spot.position.set(0, 5, 0);
spot.castShadow = true;
scene.add(spot);

// Hemisphere light (sky + ground colors)
scene.add(new THREE.HemisphereLight(0x8888ff, 0x444422, 1));
```

## 4. Loading 3D Models

### GLTF/GLB (Recommended — Industry Standard)
```javascript
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';

const loader = new GLTFLoader();
loader.load('model.glb', (gltf) => {
    const model = gltf.scene;
    model.scale.set(0.01, 0.01, 0.01);  // Scale if needed
    model.traverse((child) => {
        if (child.isMesh) {
            child.castShadow = true;
            child.receiveShadow = true;
        }
    });
    scene.add(model);
}, undefined, (error) => {
    console.error('Model load error:', error);
});
```

### OBJ + MTL
```javascript
import { OBJLoader } from 'three/addons/loaders/OBJLoader.js';
import { MTLLoader } from 'three/addons/loaders/MTLLoader.js';

const mtlLoader = new MTLLoader();
mtlLoader.load('model.mtl', (materials) => {
    materials.preload();
    const objLoader = new OBJLoader();
    objLoader.setMaterials(materials);
    objLoader.load('model.obj', (obj) => scene.add(obj));
});
```

### STL (3D prints — no color/material info)
```javascript
import { STLLoader } from 'three/addons/loaders/STLLoader.js';

const stlLoader = new STLLoader();
stlLoader.load('enclosure.stl', (geometry) => {
    const material = new THREE.MeshStandardMaterial({
        color: 0x888888, metalness: 0.2, roughness: 0.7
    });
    const mesh = new THREE.Mesh(geometry, material);
    mesh.rotation.x = -Math.PI / 2;  // STL often oriented differently
    scene.add(mesh);
});
```

## 5. Animation & Interaction

### Rotate / Oscillate (e.g., servo arm)
```javascript
let servoAngle = 0;
const servoArm = new THREE.Mesh(
    new THREE.BoxGeometry(0.2, 2, 0.2),
    new THREE.MeshStandardMaterial({ color: 0xcccccc })
);
servoArm.geometry.translate(0, 1, 0);  // Pivot at bottom

function animate() {
    requestAnimationFrame(animate);

    // Oscillate between -90° and +90°
    servoAngle = Math.sin(Date.now() * 0.002) * Math.PI / 2;
    servoArm.rotation.z = servoAngle;

    renderer.render(scene, camera);
}
```

### Click to Select / Highlight Parts
```javascript
const raycaster = new THREE.Raycaster();
const mouse = new THREE.Vector2();

renderer.domElement.addEventListener('click', (event) => {
    mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
    mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;

    raycaster.setFromCamera(mouse, camera);
    const intersects = raycaster.intersectObjects(scene.children, true);

    if (intersects.length > 0) {
        const obj = intersects[0].object;
        // Highlight selected object
        obj.material.emissive = new THREE.Color(0x333333);
        console.log('Selected:', obj.name);
    }
});
```

### Exploded View (for assemblies)
```javascript
const parts = [
    { mesh: lid, offset: new THREE.Vector3(0, 5, 0) },
    { mesh: pcb, offset: new THREE.Vector3(0, 2, 0) },
    { mesh: base, offset: new THREE.Vector3(0, 0, 0) },
];

let exploded = false;
function toggleExplode() {
    exploded = !exploded;
    parts.forEach(({ mesh, offset }) => {
        const target = exploded ? offset : new THREE.Vector3(0, 0, 0);
        // Animate with tween or lerp
        mesh.position.copy(target);
    });
}
```

## 6. Real-Time Sensor Data Visualization

### WebSocket → Three.js (live hardware data)
```javascript
// Connect to Pi/ESP32 WebSocket
const ws = new WebSocket('ws://192.168.1.100:8080/ws');

// IMU data → 3D object rotation
const board3D = new THREE.Mesh(
    new THREE.BoxGeometry(5, 0.2, 3),
    new THREE.MeshStandardMaterial({ color: 0x006600 })
);
scene.add(board3D);

ws.onmessage = (event) => {
    const data = JSON.parse(event.data);

    // Apply IMU quaternion to 3D board
    if (data.quaternion) {
        board3D.quaternion.set(data.quaternion.x, data.quaternion.y,
                               data.quaternion.z, data.quaternion.w);
    }

    // Or Euler angles
    if (data.euler) {
        board3D.rotation.set(
            THREE.MathUtils.degToRad(data.euler.roll),
            THREE.MathUtils.degToRad(data.euler.yaw),
            THREE.MathUtils.degToRad(data.euler.pitch)
        );
    }

    // Temperature → color mapping
    if (data.temperature !== undefined) {
        const t = THREE.MathUtils.clamp((data.temperature - 20) / 30, 0, 1);
        const color = new THREE.Color().setHSL(0.7 - t * 0.7, 1, 0.5);
        board3D.material.color = color;
    }

    // LED state → emissive glow
    if (data.led !== undefined) {
        ledMat.emissiveIntensity = data.led ? 2 : 0;
    }
};
```

### Sensor Data as 3D Chart
```javascript
// Live 3D bar chart (e.g., multi-sensor values)
const bars = [];
const sensorNames = ['Temp', 'Humidity', 'Pressure', 'Light'];

sensorNames.forEach((name, i) => {
    const bar = new THREE.Mesh(
        new THREE.BoxGeometry(0.8, 1, 0.8),
        new THREE.MeshStandardMaterial({ color: 0x0088ff })
    );
    bar.position.x = i * 1.5 - 2.25;
    bar.geometry.translate(0, 0.5, 0);  // Pivot at bottom
    scene.add(bar);
    bars.push(bar);
});

// Update bar heights from sensor data
function updateBars(values) {
    values.forEach((val, i) => {
        const normalized = val / 100;  // Normalize to 0-1
        bars[i].scale.y = Math.max(normalized, 0.01);
        // Color: green (low) → red (high)
        const hue = 0.33 - normalized * 0.33;
        bars[i].material.color.setHSL(hue, 1, 0.5);
    });
}
```

## 7. Digital Twin Patterns

### Robot Arm Digital Twin
```javascript
// Create kinematic chain
const base = createJoint(0x888888);
const shoulder = createJoint(0x666666);
const elbow = createJoint(0x444444);
const wrist = createJoint(0x222222);

// Hierarchy (child transforms relative to parent)
scene.add(base);
base.add(shoulder);
shoulder.position.y = 2;
shoulder.add(elbow);
elbow.position.y = 3;
elbow.add(wrist);
wrist.position.y = 2;

// Update from servo angles (received via WebSocket)
function updateArmPose(angles) {
    base.rotation.y = THREE.MathUtils.degToRad(angles[0]);
    shoulder.rotation.x = THREE.MathUtils.degToRad(angles[1]);
    elbow.rotation.x = THREE.MathUtils.degToRad(angles[2]);
    wrist.rotation.x = THREE.MathUtils.degToRad(angles[3]);
}

function createJoint(color) {
    const group = new THREE.Group();
    // Joint sphere
    group.add(new THREE.Mesh(
        new THREE.SphereGeometry(0.3), new THREE.MeshStandardMaterial({ color })
    ));
    // Link arm
    const arm = new THREE.Mesh(
        new THREE.BoxGeometry(0.2, 2, 0.2),
        new THREE.MeshStandardMaterial({ color: 0xaaaaaa })
    );
    arm.position.y = 1;
    group.add(arm);
    return group;
}
```

### Drone Orientation Viewer
```javascript
// Drone body
const drone = new THREE.Group();
const body = new THREE.Mesh(
    new THREE.BoxGeometry(2, 0.3, 2),
    new THREE.MeshStandardMaterial({ color: 0x333333 })
);
drone.add(body);

// 4 arms + rotors
for (let i = 0; i < 4; i++) {
    const angle = (i * Math.PI / 2) + Math.PI / 4;
    const arm = new THREE.Mesh(
        new THREE.BoxGeometry(0.15, 0.1, 1.5),
        new THREE.MeshStandardMaterial({ color: 0x555555 })
    );
    arm.position.set(Math.cos(angle) * 1.2, 0, Math.sin(angle) * 1.2);
    arm.rotation.y = -angle;
    drone.add(arm);

    const rotor = new THREE.Mesh(
        new THREE.CylinderGeometry(0.5, 0.5, 0.02, 16),
        new THREE.MeshStandardMaterial({ color: 0x0088ff, transparent: true, opacity: 0.5 })
    );
    rotor.position.set(Math.cos(angle) * 1.8, 0.1, Math.sin(angle) * 1.8);
    drone.add(rotor);
}
scene.add(drone);

// Update from IMU
ws.onmessage = (e) => {
    const d = JSON.parse(e.data);
    drone.rotation.set(d.pitch, d.yaw, d.roll);
};
```

## 8. Hardware Dashboard Integration

### Split Screen: Three.js + Control Panel
```html
<div style="display: flex; height: 100vh;">
    <!-- 3D viewport -->
    <div id="viewport" style="flex: 2;"></div>

    <!-- Control panel -->
    <div id="controls" style="flex: 1; padding: 20px; background: #1a1a2e; color: #eee;">
        <h2>Hardware Controls</h2>
        <div>
            <label>Servo 1: <span id="servo1-val">90</span>°</label>
            <input type="range" id="servo1" min="0" max="180" value="90"
                   oninput="sendServo(1, this.value)">
        </div>
        <div>
            <label>LED</label>
            <button onclick="sendCommand('toggle_led')">Toggle</button>
        </div>
        <div id="sensor-readout">
            <p>Temperature: <span id="temp">--</span>°C</p>
            <p>Humidity: <span id="humidity">--</span>%</p>
        </div>
    </div>
</div>

<script type="module">
// Three.js scene in #viewport
// WebSocket for bidirectional control + telemetry
const ws = new WebSocket('ws://192.168.1.100/ws');

window.sendServo = (id, angle) => {
    ws.send(JSON.stringify({ cmd: 'servo', id, angle: parseInt(angle) }));
    document.getElementById(`servo${id}-val`).textContent = angle;
    // Update 3D model too
    updateArmPose([parseInt(angle)]);
};

window.sendCommand = (cmd) => ws.send(JSON.stringify({ cmd }));

ws.onmessage = (e) => {
    const data = JSON.parse(e.data);
    if (data.temp) document.getElementById('temp').textContent = data.temp.toFixed(1);
    if (data.humidity) document.getElementById('humidity').textContent = data.humidity.toFixed(0);
};
</script>
```

## 9. Performance Optimization

| Technique | Impact | When |
|-----------|--------|------|
| Lower renderer.setPixelRatio | Big | Mobile/Pi browser |
| Reduce shadow map size | Medium | Multiple lights |
| Use BufferGeometry (default in modern Three.js) | Built-in | Always |
| InstancedMesh for repeated objects | Big | >100 similar objects |
| LOD (Level of Detail) | Medium | Large scenes |
| Frustum culling (automatic) | Built-in | Always on |
| Merge static geometries | Medium | Many static objects |
| Dispose unused geometries/textures | Memory | Dynamic scenes |

```javascript
// InstancedMesh example (1000 identical components)
const geometry = new THREE.BoxGeometry(0.1, 0.1, 0.1);
const material = new THREE.MeshStandardMaterial({ color: 0x888888 });
const instances = new THREE.InstancedMesh(geometry, material, 1000);

const dummy = new THREE.Object3D();
for (let i = 0; i < 1000; i++) {
    dummy.position.set(Math.random()*10, Math.random()*10, Math.random()*10);
    dummy.updateMatrix();
    instances.setMatrixAt(i, dummy.matrix);
}
scene.add(instances);
```

## 10. WebXR (AR/VR in Browser)

### Basic AR Session
```javascript
import { ARButton } from 'three/addons/webxr/ARButton.js';

renderer.xr.enabled = true;
document.body.appendChild(ARButton.createButton(renderer));

// Place objects in AR
const controller = renderer.xr.getController(0);
controller.addEventListener('select', () => {
    const mesh = new THREE.Mesh(
        new THREE.BoxGeometry(0.1, 0.1, 0.1),
        new THREE.MeshStandardMaterial({ color: 0x00ff88 })
    );
    mesh.position.copy(controller.position);
    scene.add(mesh);
});
scene.add(controller);

// Modified render loop for XR
renderer.setAnimationLoop(() => {
    renderer.render(scene, camera);
});
```

### Hit Testing (place objects on real surfaces)
```javascript
let hitTestSource = null;

renderer.xr.addEventListener('sessionstart', async () => {
    const session = renderer.xr.getSession();
    const viewerSpace = await session.requestReferenceSpace('viewer');
    hitTestSource = await session.requestHitTestSource({ space: viewerSpace });
});

// In render loop:
if (hitTestSource) {
    const hitResults = frame.getHitTestResults(hitTestSource);
    if (hitResults.length > 0) {
        const hit = hitResults[0];
        const pose = hit.getPose(referenceSpace);
        reticle.visible = true;
        reticle.matrix.fromArray(pose.transform.matrix);
    }
}
```
