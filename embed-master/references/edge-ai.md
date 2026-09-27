# Edge AI & On-Device Machine Learning Reference

## Table of Contents
1. TensorFlow Lite on Pi
2. ONNX Runtime on Pi
3. TFLite Micro on ESP32/Arduino
4. Offline LLMs (llama.cpp, Ollama)
5. Speech Recognition (Whisper.cpp)
6. RAG on Edge
7. Agentic AI on Edge
8. AI Accelerators
9. Edge Impulse (AutoML for MCUs)

---

## 1. TensorFlow Lite on Raspberry Pi

### Installation
```bash
pip install tflite-runtime  # Lightweight, inference only
# OR full TensorFlow (larger, includes training):
pip install tensorflow
```

### Image Classification Example
```python
import numpy as np
from tflite_runtime.interpreter import Interpreter
from PIL import Image

# Load model
interpreter = Interpreter(model_path="mobilenet_v2.tflite")
interpreter.allocate_tensors()
input_details = interpreter.get_input_details()
output_details = interpreter.get_output_details()

# Preprocess image
img = Image.open("photo.jpg").resize((224, 224))
input_data = np.expand_dims(np.array(img, dtype=np.float32) / 255.0, axis=0)

# Inference
interpreter.set_tensor(input_details[0]['index'], input_data)
interpreter.invoke()
output = interpreter.get_tensor(output_details[0]['index'])
predicted_class = np.argmax(output)
```

### Object Detection (SSD MobileNet)
```python
interpreter = Interpreter(model_path="ssd_mobilenet_v2.tflite")
interpreter.allocate_tensors()
# ... (same pattern — preprocess, invoke, parse bounding boxes)
# Output tensors: boxes, classes, scores, count
```

### Performance Tips
- Use quantized models (INT8) — 2-4× faster than FP32
- Resize input to smallest acceptable resolution
- Use `interpreter.set_num_threads(4)` for multi-core
- Delegate to GPU: `Interpreter(model_path=..., experimental_delegates=[load_delegate('libedgetpu.so.1')])`

## 2. ONNX Runtime on Raspberry Pi

```bash
pip install onnxruntime  # CPU
# For models: download from ONNX Model Zoo or export from PyTorch
```

```python
import onnxruntime as ort
import numpy as np

session = ort.InferenceSession("model.onnx")
input_name = session.get_inputs()[0].name
result = session.run(None, {input_name: input_data})
```

ONNX vs TFLite: ONNX better for PyTorch-origin models, TFLite better for
TensorFlow-origin and has broader MCU support.

## 3. TFLite Micro on ESP32 / Arduino

For running ML on microcontrollers (keyword spotting, anomaly detection, gesture recognition):

```cpp
#include <TensorFlowLite_ESP32.h>
// Or use the Arduino_TensorFlowLite library

#include "model_data.h"  // Your .tflite model as C array

// Setup
tflite::MicroInterpreter* interpreter;
const int kTensorArenaSize = 10 * 1024;  // Adjust per model
uint8_t tensor_arena[kTensorArenaSize];

void setup() {
  static tflite::MicroMutableOpResolver<5> resolver;
  resolver.AddFullyConnected();
  resolver.AddSoftmax();
  // Add only the ops your model needs

  static tflite::MicroInterpreter static_interpreter(
      model, resolver, tensor_arena, kTensorArenaSize);
  interpreter = &static_interpreter;
  interpreter->AllocateTensors();
}

void loop() {
  float* input = interpreter->input(0)->data.f;
  // Fill input buffer with sensor data
  interpreter->Invoke();
  float* output = interpreter->output(0)->data.f;
  // Use output for decision
}
```

### Model Size Limits on MCUs
| Platform | RAM | Practical Model Size |
|----------|-----|---------------------|
| Arduino Uno | 2KB | Not viable |
| Arduino Mega | 8KB | Very tiny models (<5KB) |
| ESP32 | 520KB | Up to ~300KB models |
| Pico (RP2040) | 264KB | Up to ~150KB models |

## 4. Offline LLMs on Raspberry Pi

### llama.cpp (Recommended — Best Performance)

```bash
# Install on Pi 5
sudo apt install cmake g++ git
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp && mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
cmake --build . -j4

# Download a model (GGUF format)
# Example: Phi-3-mini-4k-instruct Q4_K_M (~2.3GB)
wget https://huggingface.co/microsoft/Phi-3-mini-4k-instruct-gguf/resolve/main/Phi-3-mini-4k-instruct-q4.gguf

# Run
./bin/llama-cli -m Phi-3-mini-4k-instruct-q4.gguf \
  -p "Explain how a capacitor works in simple terms:" \
  -n 256 -t 4 --temp 0.7
```

### llama.cpp as HTTP Server (for programmatic access)
```bash
./bin/llama-server -m model.gguf -c 2048 -t 4 --port 8080
# Then POST to http://localhost:8080/completion
```

```python
import requests
response = requests.post("http://localhost:8080/completion", json={
    "prompt": "What is a resistor?",
    "n_predict": 128, "temperature": 0.7
})
print(response.json()["content"])
```

### Ollama (Easiest Setup)

```bash
# Install
curl -fsSL https://ollama.ai/install.sh | sh

# Run a model
ollama run phi3          # Phi-3-mini (~2.3GB)
ollama run tinyllama     # TinyLlama 1.1B (~637MB)
ollama run gemma:2b      # Gemma 2B (~1.4GB)

# API access (runs on port 11434)
curl http://localhost:11434/api/generate -d '{
  "model": "phi3",
  "prompt": "Explain PWM in one paragraph"
}'
```

```python
# Python with Ollama
import ollama
response = ollama.chat(model='phi3', messages=[
    {'role': 'user', 'content': 'What is I2C?'}
])
print(response['message']['content'])
```

### Model Selection Guide for Pi

| Model | Params | GGUF Size (Q4) | Pi 5 8GB Speed | Pi 4 8GB Speed | Quality |
|-------|--------|----------------|---------------|----------------|---------|
| TinyLlama 1.1B | 1.1B | ~637MB | ~8 tok/s | ~2 tok/s | Basic |
| Phi-3-mini | 3.8B | ~2.3GB | ~4 tok/s | ~0.8 tok/s | Good |
| Gemma 2B | 2B | ~1.4GB | ~6 tok/s | ~1.5 tok/s | Good |
| Llama 3.2 1B | 1B | ~700MB | ~10 tok/s | ~3 tok/s | Good |
| Llama 3.2 3B | 3B | ~2GB | ~4 tok/s | Too slow | Better |
| Mistral 7B | 7B | ~4.1GB | ~1.5 tok/s | OOM | Best |

**Rule of thumb**: On Pi 5 8GB, models up to ~4GB GGUF run usably. Pi 4 caps at ~1.5GB models.
Pi 4 4GB or less: skip LLMs, use API calls instead.

### Quantization Levels
- Q8_0: Best quality, 2× model size
- Q5_K_M: Great quality, moderate size
- **Q4_K_M**: Best balance for Pi (recommended)
- Q3_K_S: Smaller but noticeable quality loss
- Q2_K: Emergency only — significant degradation

## 5. Speech Recognition (Whisper.cpp)

```bash
git clone https://github.com/ggerganov/whisper.cpp
cd whisper.cpp && make -j4

# Download model
bash ./models/download-ggml-model.sh tiny.en  # 75MB, fastest
# Also available: base.en (142MB), small.en (466MB)

# Transcribe
./main -m models/ggml-tiny.en.bin -f audio.wav
```

| Model | Size | Pi 5 Speed | Pi 4 Speed | Quality |
|-------|------|-----------|-----------|---------|
| tiny.en | 75MB | ~4× realtime | ~1.5× realtime | Decent |
| base.en | 142MB | ~2× realtime | ~0.7× realtime | Good |
| small.en | 466MB | ~0.5× realtime | Too slow | Great |

For real-time voice commands, use `tiny.en` with voice activity detection (VAD).

## 6. RAG on Edge (Raspberry Pi 5)

### Architecture
```
Documents → Chunking → Embedding → Vector Store (ChromaDB)
                                        ↓
User Query → Embedding → Similarity Search → Top-K Chunks
                                        ↓
                        Prompt Template + Chunks → Local LLM → Response
```

### Implementation
```bash
pip install chromadb sentence-transformers langchain
```

```python
from langchain.embeddings import HuggingFaceEmbeddings
from langchain.vectorstores import Chroma
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.llms import Ollama
from langchain.chains import RetrievalQA

# 1. Embeddings (runs locally, ~80MB model)
embeddings = HuggingFaceEmbeddings(
    model_name="all-MiniLM-L6-v2",
    model_kwargs={'device': 'cpu'}
)

# 2. Ingest documents
splitter = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=50)
chunks = splitter.split_documents(documents)
vectorstore = Chroma.from_documents(chunks, embeddings, persist_directory="./chroma_db")

# 3. Local LLM
llm = Ollama(model="phi3", temperature=0.3)

# 4. RAG chain
qa = RetrievalQA.from_chain_type(
    llm=llm, retriever=vectorstore.as_retriever(search_kwargs={"k": 3})
)
result = qa.run("What does the datasheet say about max current?")
```

### Performance Tips for Edge RAG
- Use `all-MiniLM-L6-v2` for embeddings (~80MB, fast on Pi)
- ChromaDB over FAISS for simplicity (FAISS faster for >100K documents)
- Chunk size 300-500 tokens for small context LLMs
- Keep retrieval k=3-5 to fit in LLM context window
- Pre-embed documents — don't re-embed on every query

## 7. Agentic AI on Edge

### Simple Agent Loop
```python
import ollama
import json

tools = {
    "read_sensor": lambda sensor: read_gpio_sensor(sensor),
    "control_pin": lambda pin, state: set_gpio(pin, state),
    "capture_image": lambda: camera_capture(),
}

def agent_loop(task, max_steps=5):
    history = [{"role": "user", "content": f"""You are a hardware agent.
    Available tools: {list(tools.keys())}
    Respond with JSON: {{"thought": "...", "tool": "...", "args": {{}}}}
    Or {{"thought": "...", "answer": "..."}} when done.
    Task: {task}"""}]

    for step in range(max_steps):
        response = ollama.chat(model='phi3', messages=history)
        content = response['message']['content']
        try:
            action = json.loads(content)
            if 'answer' in action:
                return action['answer']
            result = tools[action['tool']](**action['args'])
            history.append({"role": "assistant", "content": content})
            history.append({"role": "user", "content": f"Tool result: {result}"})
        except Exception as e:
            history.append({"role": "user", "content": f"Error: {e}. Try again."})

    return "Max steps reached"
```

### Agent Safety Rules
- **Watchdog timer**: Kill stuck loops after 60 seconds
- **Pin whitelist**: Only allow control of designated GPIO pins
- **Action logging**: Log every tool call with timestamp
- **Rate limiting**: Max 1 GPIO toggle per 100ms to prevent oscillation damage
- **Confirmation**: Require human approval for destructive actions (motor control)

## 8. AI Accelerators (Hardware Add-ons)

| Accelerator | Interface | TOPS | Price | Best For |
|-------------|-----------|------|-------|----------|
| Google Coral USB | USB 3.0 | 4 | ~$60 | TFLite INT8 models |
| Coral M.2/Mini PCIe | M.2/PCIe | 4 | ~$30 | Pi 5 PCIe slot |
| Hailo-8L (Pi AI HAT) | PCIe | 13 | ~$70 | Pi 5 dedicated, YOLO |
| Intel NCS2 | USB 3.0 | ~1 | ~$70 | OpenVINO models |

### Coral USB on Pi
```bash
# Install
echo "deb https://packages.cloud.google.com/apt coral-edgetpu-stable main" | \
  sudo tee /etc/apt/sources.list.d/coral-edgetpu.list
sudo apt update && sudo apt install libedgetpu1-std python3-pycoral

# Use in TFLite
from tflite_runtime.interpreter import Interpreter, load_delegate
interpreter = Interpreter(
    model_path="model_edgetpu.tflite",
    experimental_delegates=[load_delegate('libedgetpu.so.1')]
)
```

## 9. Edge Impulse (AutoML for Microcontrollers)

For training and deploying custom ML models on ESP32/Arduino/Pico:
1. Collect sensor data via Edge Impulse CLI or web dashboard
2. Design signal processing + neural network pipeline in browser
3. Export as Arduino library or TFLite model
4. Flash to device

Best for: vibration anomaly detection, keyword spotting, gesture classification.
```bash
npm install -g edge-impulse-cli
edge-impulse-daemon  # Connects your board to Edge Impulse cloud
```
