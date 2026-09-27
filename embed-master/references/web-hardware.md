# Web-Based Hardware Control Reference

## Table of Contents
1. ESP32 as Standalone Web Server
2. Raspberry Pi Web Server (Flask / FastAPI)
3. Node.js + JavaScript for Hardware
4. WebSocket Real-Time Communication
5. MQTT for IoT
6. Frontend Dashboard Patterns
7. Multi-Device Architecture

---

## 1. ESP32 as Standalone Web Server

### AsyncWebServer + WebSocket (Recommended)
```cpp
#include <WiFi.h>
#include <ESPAsyncWebServer.h>
#include <AsyncWebSocket.h>
#include <SPIFFS.h>  // Or LittleFS

const char* ssid = "YOUR_SSID";
const char* password = "YOUR_PASS";

AsyncWebServer server(80);
AsyncWebSocket ws("/ws");

// Sensor data
float temperature = 0;
bool ledState = false;

void onWsEvent(AsyncWebSocket *server, AsyncWebSocketClient *client,
               AwsEventType type, void *arg, uint8_t *data, size_t len) {
    if (type == WS_EVT_DATA) {
        String msg = String((char*)data).substring(0, len);
        if (msg == "toggle_led") {
            ledState = !ledState;
            digitalWrite(2, ledState);
            // Broadcast state to all clients
            ws.textAll("{\"led\":" + String(ledState) + "}");
        }
    }
}

void setup() {
    pinMode(2, OUTPUT);
    WiFi.begin(ssid, password);
    while (WiFi.status() != WL_CONNECTED) delay(500);

    SPIFFS.begin(true);  // Serve HTML from SPIFFS

    ws.onEvent(onWsEvent);
    server.addHandler(&ws);

    // Serve frontend
    server.on("/", HTTP_GET, [](AsyncWebServerRequest *req) {
        req->send(SPIFFS, "/index.html", "text/html");
    });

    // REST API endpoint
    server.on("/api/sensor", HTTP_GET, [](AsyncWebServerRequest *req) {
        req->send(200, "application/json",
            "{\"temperature\":" + String(temperature) + "}");
    });

    server.begin();
}

void loop() {
    temperature = analogRead(34) * 0.1;  // Example sensor read
    // Send sensor data every 2 seconds
    static unsigned long lastSend = 0;
    if (millis() - lastSend > 2000) {
        ws.textAll("{\"temp\":" + String(temperature) + "}");
        lastSend = millis();
    }
    ws.cleanupClients();
}
```

### Upload HTML to SPIFFS/LittleFS
```bash
# In PlatformIO: place files in data/ folder, then:
pio run --target uploadfs
# In Arduino IDE: use ESP32 Sketch Data Upload plugin
```

### ESP32 Frontend (index.html served from SPIFFS)
```html
<!DOCTYPE html>
<html>
<head>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>ESP32 Dashboard</title>
    <style>
        body { font-family: sans-serif; max-width: 600px; margin: 0 auto; padding: 20px; }
        .card { background: #1a1a2e; color: #eee; padding: 20px; border-radius: 12px; margin: 10px 0; }
        .value { font-size: 2em; font-weight: bold; color: #0ff; }
        button { background: #e94560; color: white; border: none; padding: 12px 24px;
                 border-radius: 8px; font-size: 1.1em; cursor: pointer; }
    </style>
</head>
<body>
    <h1>ESP32 Dashboard</h1>
    <div class="card">
        <p>Temperature</p>
        <p class="value" id="temp">--</p>
    </div>
    <div class="card">
        <button onclick="send('toggle_led')">Toggle LED</button>
        <p>LED: <span id="led-state">OFF</span></p>
    </div>

    <script>
        const ws = new WebSocket(`ws://${location.host}/ws`);

        ws.onmessage = (event) => {
            const data = JSON.parse(event.data);
            if (data.temp !== undefined)
                document.getElementById('temp').textContent = data.temp.toFixed(1) + '°C';
            if (data.led !== undefined)
                document.getElementById('led-state').textContent = data.led ? 'ON' : 'OFF';
        };

        ws.onclose = () => setTimeout(() => location.reload(), 3000);

        function send(msg) { ws.send(msg); }
    </script>
</body>
</html>
```

## 2. Raspberry Pi Web Server

### Flask (Simple, Python)
```python
from flask import Flask, jsonify, render_template, request
from flask_socketio import SocketIO, emit
import gpiozero
import threading, time

app = Flask(__name__)
socketio = SocketIO(app, cors_allowed_origins="*")

led = gpiozero.LED(17)
sensor_data = {"temperature": 0, "humidity": 0}

# Background sensor reader
def read_sensors():
    while True:
        sensor_data["temperature"] = read_temp_sensor()  # Your sensor function
        socketio.emit('sensor_update', sensor_data)
        time.sleep(2)

threading.Thread(target=read_sensors, daemon=True).start()

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/api/led', methods=['POST'])
def toggle_led():
    led.toggle()
    return jsonify({"led": led.is_lit})

@socketio.on('command')
def handle_command(data):
    if data.get('action') == 'set_pin':
        pin = gpiozero.LED(data['pin'])
        pin.on() if data['state'] else pin.off()
        emit('pin_update', {'pin': data['pin'], 'state': data['state']})

if __name__ == '__main__':
    socketio.run(app, host='0.0.0.0', port=5000)
```

### FastAPI (Modern, Async, Type-Safe)
```python
from fastapi import FastAPI, WebSocket
from fastapi.staticfiles import StaticFiles
import asyncio, json

app = FastAPI()
app.mount("/static", StaticFiles(directory="static"), name="static")

clients = set()

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    clients.add(websocket)
    try:
        while True:
            data = await websocket.receive_text()
            cmd = json.loads(data)
            # Handle command, broadcast response
            for client in clients:
                await client.send_json({"status": "ok"})
    except:
        clients.discard(websocket)

# Run: uvicorn main:app --host 0.0.0.0 --port 8000
```

## 3. Node.js + JavaScript for Hardware

### GPIO Control (onoff library)
```javascript
const { Gpio } = require('onoff');

const led = new Gpio(17, 'out');
const button = new Gpio(22, 'in', 'both');

button.watch((err, value) => {
    if (err) throw err;
    led.writeSync(value);
});

// Cleanup on exit
process.on('SIGINT', () => {
    led.unexport();
    button.unexport();
    process.exit();
});
```

### PWM / Servo (pigpio library)
```javascript
const Gpio = require('pigpio').Gpio;

const servo = new Gpio(18, { mode: Gpio.OUTPUT });
servo.servoWrite(1500);  // Center position (500-2500µs)

const motor = new Gpio(12, { mode: Gpio.OUTPUT });
motor.pwmWrite(128);  // 0-255 duty cycle
```

### I2C (i2c-bus)
```javascript
const i2c = require('i2c-bus');

async function readSensor() {
    const bus = await i2c.openPromisified(1);  // I2C bus 1
    const data = Buffer.alloc(2);
    await bus.readI2cBlock(0x48, 0x00, 2, data);  // addr, register, length
    const temp = (data[0] << 8 | data[1]) / 256;
    await bus.close();
    return temp;
}
```

### Serial Port (serialport)
```javascript
const { SerialPort } = require('serialport');
const { ReadlineParser } = require('@serialport/parser-readline');

const port = new SerialPort({ path: '/dev/ttyUSB0', baudRate: 115200 });
const parser = port.pipe(new ReadlineParser({ delimiter: '\r\n' }));

parser.on('data', (line) => {
    console.log('Arduino says:', line);
});

port.write('LED_ON\n');
```

### Express + Socket.IO Server
```javascript
const express = require('express');
const http = require('http');
const { Server } = require('socket.io');
const { Gpio } = require('onoff');

const app = express();
const server = http.createServer(app);
const io = new Server(server);

const led = new Gpio(17, 'out');
let sensorData = { temp: 0, humidity: 0 };

app.use(express.static('public'));

io.on('connection', (socket) => {
    console.log('Client connected');
    socket.emit('sensor_update', sensorData);

    socket.on('toggle_led', () => {
        const state = led.readSync() ^ 1;
        led.writeSync(state);
        io.emit('led_state', { on: state === 1 });
    });

    socket.on('disconnect', () => console.log('Client disconnected'));
});

// Sensor broadcast loop
setInterval(() => {
    sensorData.temp = readTemperature();  // Your sensor function
    io.emit('sensor_update', sensorData);
}, 2000);

server.listen(3000, '0.0.0.0', () => console.log('Server on :3000'));
```

### package.json
```json
{
  "name": "pi-hardware-server",
  "dependencies": {
    "express": "^4.18",
    "socket.io": "^4.7",
    "onoff": "^6.0",
    "pigpio": "^3.3",
    "i2c-bus": "^5.2",
    "serialport": "^12.0",
    "mqtt": "^5.3"
  }
}
```

## 4. WebSocket Patterns

### Browser Client (vanilla JS)
```javascript
class HardwareSocket {
    constructor(url) {
        this.url = url;
        this.connect();
    }

    connect() {
        this.ws = new WebSocket(this.url);
        this.ws.onopen = () => console.log('Connected');
        this.ws.onclose = () => {
            console.log('Disconnected, reconnecting...');
            setTimeout(() => this.connect(), 3000);
        };
        this.ws.onmessage = (event) => {
            const data = JSON.parse(event.data);
            this.onData(data);
        };
    }

    send(command, payload = {}) {
        this.ws.send(JSON.stringify({ command, ...payload }));
    }

    onData(data) {
        // Override this for your dashboard
    }
}
```

## 5. MQTT for IoT

### Mosquitto Broker on Pi
```bash
sudo apt install mosquitto mosquitto-clients
sudo systemctl enable mosquitto
# Test: mosquitto_sub -t "test" & mosquitto_pub -t "test" -m "hello"
```

### ESP32 MQTT Client (PubSubClient)
```cpp
#include <WiFi.h>
#include <PubSubClient.h>

WiFiClient espClient;
PubSubClient mqtt(espClient);

void callback(char* topic, byte* payload, unsigned int length) {
    String msg = String((char*)payload).substring(0, length);
    if (String(topic) == "home/led") {
        digitalWrite(2, msg == "on" ? HIGH : LOW);
    }
}

void setup() {
    WiFi.begin(ssid, password);
    mqtt.setServer("192.168.1.100", 1883);  // Pi's IP
    mqtt.setCallback(callback);
}

void reconnect() {
    while (!mqtt.connected()) {
        mqtt.connect("esp32_sensor");
        mqtt.subscribe("home/led");
    }
}

void loop() {
    if (!mqtt.connected()) reconnect();
    mqtt.loop();

    // Publish sensor data every 5s
    static unsigned long lastPub = 0;
    if (millis() - lastPub > 5000) {
        mqtt.publish("home/temperature", String(readTemp()).c_str());
        lastPub = millis();
    }
}
```

### Python MQTT Client (Pi-side)
```python
import paho.mqtt.client as mqtt

def on_message(client, userdata, msg):
    print(f"{msg.topic}: {msg.payload.decode()}")

client = mqtt.Client()
client.on_message = on_message
client.connect("localhost", 1883)
client.subscribe("home/#")  # All home topics
client.loop_forever()
```

### Node.js MQTT Client
```javascript
const mqtt = require('mqtt');
const client = mqtt.connect('mqtt://localhost');

client.on('connect', () => {
    client.subscribe('home/#');
});

client.on('message', (topic, message) => {
    console.log(`${topic}: ${message.toString()}`);
});

client.publish('home/led', 'on');
```

## 6. Frontend Dashboard Patterns

### Live Chart (Chart.js + WebSocket)
```html
<canvas id="tempChart" width="600" height="300"></canvas>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script>
const ctx = document.getElementById('tempChart').getContext('2d');
const chart = new Chart(ctx, {
    type: 'line',
    data: {
        labels: [],
        datasets: [{
            label: 'Temperature',
            data: [],
            borderColor: '#0ff',
            tension: 0.3,
            fill: false
        }]
    },
    options: { scales: { x: { display: true }, y: { min: 15, max: 45 } } }
});

const ws = new WebSocket('ws://192.168.1.100/ws');
ws.onmessage = (event) => {
    const data = JSON.parse(event.data);
    const now = new Date().toLocaleTimeString();
    chart.data.labels.push(now);
    chart.data.datasets[0].data.push(data.temp);
    if (chart.data.labels.length > 30) {
        chart.data.labels.shift();
        chart.data.datasets[0].data.shift();
    }
    chart.update('none');
};
</script>
```

## 7. Multi-Device Architecture

### Hub-and-Spoke (Pi as central hub)
```
[ESP32 #1] --MQTT--> [Pi (Mosquitto + Flask)] <--WebSocket--> [Browser Dashboard]
[ESP32 #2] --MQTT-->                          <--REST API----> [Mobile App]
[ESP32-CAM] --HTTP-->
[Pi Camera] --local->
```

### Topic Naming Convention
```
home/{room}/{device}/{property}
home/kitchen/esp32_01/temperature
home/garage/door/state
home/living/light/brightness
```
