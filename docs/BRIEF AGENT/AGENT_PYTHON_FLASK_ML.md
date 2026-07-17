# PANDUAN PENULISAN KODE — Python Flask + ML/AI API

Dokumen ini berisi standar dan konvensi penulisan kode yang wajib dipatuhi AI saat mengerjakan proyek API berbasis **Python Flask** untuk serving model **Machine Learning / AI** (TFLite, PyTorch, ONNX, dll).

---

## TECH STACK

| Layer | Teknologi |
|---|---|
| Web Framework | Flask (Python 3.10+) |
| ML Runtime | TensorFlow Lite / PyTorch / ONNX Runtime |
| Image Processing | OpenCV (cv2) + NumPy |
| WSGI Server | Gunicorn (production) |
| Hosting Target | Render.com / Railway / VPS / Docker |

---

## 1. STRUKTUR FOLDER PROJECT

```
project-root/
├── api_flask.py            # Aplikasi Flask utama (endpoint)
├── model.tflite            # Model ML hasil training (atau .onnx / .pt)
├── requirements.txt        # Dependensi Python
├── start.sh                # Script start untuk deployment (gunicorn)
├── Dockerfile              # (Opsional) Container deployment
├── .env                    # Environment variables
├── utils/
│   ├── preprocessing.py    # Fungsi preprocessing gambar/data
│   ├── postprocessing.py   # Fungsi postprocessing hasil prediksi
│   └── config.py           # Konstanta & konfigurasi
├── tests/
│   └── test_predict.py     # Unit test endpoint
└── referensi/              # Notebook/script referensi (tidak dijalankan)
```

---

## 2. ENDPOINT FLASK

### 2.1 Standar Endpoint
| Endpoint | Method | Fungsi |
|---|---|---|
| `/health` | GET | Health check, cek model aktif |
| `/predict` | POST | Terima input → kembalikan JSON prediksi |

### 2.2 Contoh Implementasi
```python
from flask import Flask, request, jsonify
import numpy as np
import cv2

app = Flask(__name__)

# Load model saat startup (BUKAN di setiap request)
interpreter = load_model('model.tflite')

@app.route('/health', methods=['GET'])
def health():
    return jsonify({"status": "ok", "model_loaded": interpreter is not None})

@app.route('/predict', methods=['POST'])
def predict():
    try:
        if 'image' not in request.files:
            return jsonify({"error": "No image provided"}), 400

        file = request.files['image']
        result = run_prediction(file, interpreter)
        return jsonify(result)

    except Exception as e:
        return jsonify({"error": str(e)}), 500
```

### 2.3 Format Response `/predict`
```json
{
  "label": "ClassA",
  "confidence": 0.9423,
  "probs": {
    "ClassA": 0.9423,
    "ClassB": 0.0312,
    "ClassC": 0.0198,
    "ClassD": 0.0046
  }
}
```

---

## 3. PREPROCESSING (KRITIS)

### 3.1 Konversi Warna (BGR → RGB)
**WAJIB** konversi BGR → RGB sebelum prediksi. OpenCV membaca BGR, model dilatih dengan RGB:
```python
img = cv2.imdecode(file_bytes, cv2.IMREAD_COLOR)  # BGR
img = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)         # → RGB ← WAJIB!
img = cv2.resize(img, (IMAGE_SIZE, IMAGE_SIZE))
img = img.astype("float32") / 255.0
img = np.expand_dims(img, axis=0)
```
**Jangan pernah hapus baris `cv2.cvtColor`** — ini penyebab utama prediksi salah.

### 3.2 Urutan Kelas Model
Urutan harus KONSISTEN antara training dan inference:
```python
CLASS_NAMES = [
    "ClassA",    # index 0
    "ClassB",    # index 1
    "ClassC",    # index 2
    "ClassD",    # index 3
]
```

### 3.3 Konstanta Preprocessing
```python
# utils/config.py
IMAGE_SIZE = 128          # Sesuaikan dengan training
CONFIDENCE_THRESHOLD = 0.5
MAX_FILE_SIZE = 5 * 1024 * 1024  # 5 MB
ALLOWED_EXTENSIONS = {'jpg', 'jpeg', 'png'}
```

---

## 4. MODEL FORMAT & DEPLOYMENT

### 4.1 Pilihan Format Model
| Format | Ukuran | Cocok Untuk |
|---|---|---|
| `.tflite` (FP16) | ~30 MB | Render.com, Railway, edge device |
| `.onnx` | ~50 MB | General purpose, cross-platform |
| `.pt` (PyTorch) | ~100 MB | VPS dengan GPU |
| `.h5` (Keras) | ~100+ MB | Terlalu besar untuk shared hosting |

### 4.2 Paksa CPU-Only
```python
import os
os.environ["CUDA_VISIBLE_DEVICES"] = "-1"  # Paksa CPU
```

### 4.3 Gunicorn Production
```bash
# start.sh
gunicorn api_flask:app --bind 0.0.0.0:$PORT --workers 2 --timeout 120
```

---

## 5. KONEKSI DENGAN APLIKASI LAIN

### 5.1 Panggilan dari PHP (cURL)
```php
$ch = curl_init(API_URL);
curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, ['image' => new CURLFile($filePath)]);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_TIMEOUT, 30);
$response = curl_exec($ch);
$result = json_decode($response, true);
curl_close($ch);
```

### 5.2 Panggilan dari JavaScript (Fetch)
```javascript
const formData = new FormData();
formData.append('image', fileInput.files[0]);

const response = await fetch('https://api-url.com/predict', {
  method: 'POST',
  body: formData,
});
const result = await response.json();
```

---

## 6. ERROR HANDLING

```python
@app.errorhandler(413)
def file_too_large(e):
    return jsonify({"error": "File terlalu besar. Maksimal 5MB."}), 413

@app.errorhandler(500)
def internal_error(e):
    return jsonify({"error": "Internal server error"}), 500
```

- Selalu return JSON, jangan return HTML error page.
- Log error ke file/stdout untuk debugging.
- Set timeout yang wajar (30 detik) untuk cold start di free tier hosting.

---

## 7. TESTING

```python
# tests/test_predict.py
import pytest
from api_flask import app

@pytest.fixture
def client():
    app.config['TESTING'] = True
    with app.test_client() as client:
        yield client

def test_health(client):
    response = client.get('/health')
    assert response.status_code == 200
    assert response.json['status'] == 'ok'

def test_predict_no_image(client):
    response = client.post('/predict')
    assert response.status_code == 400
```

---

## 8. ENVIRONMENT VARIABLES

```env
FLASK_ENV=production
PORT=5000
MODEL_PATH=model.tflite
MAX_CONTENT_LENGTH=5242880
```
- Jangan hardcode URL, port, atau path model di dalam kode.
- Gunakan `os.getenv()` untuk membaca environment variable.
