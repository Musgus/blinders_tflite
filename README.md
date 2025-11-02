# Blinders TFLite – Full Project Guide

This repository contains both the Python pipeline (training + desktop runner) and the Flutter Android app that runs the exported TFLite model in real time.

## Structure

```
AI/
├─ scripts/                 # Python: training + realtime runner
│  ├─ train_tf_recognizer.py
│  ├─ run_tflite_faces_realtime.py
│  ├─ requirements.txt
│  └─ output/               # Model artifacts
│     ├─ model.tflite
│     ├─ labels.json
│     ├─ confusion_matrix.png
│     └─ saved_model/
├─ flutter_face_app/        # Flutter app (Android)
│  ├─ lib/
│  │  ├─ main.dart
│  │  └─ pipeline/
│  │     ├─ image_utils.dart
│  │     ├─ mlkit_service.dart
│  │     └─ tflite_service.dart
│  ├─ assets/
│  │  ├─ model.tflite
│  │  └─ labels.json
│  └─ pubspec.yaml
├─ data/                    # (ignored) your raw images if any
└─ videos/                  # (ignored) your videos if any
```

The `.gitignore` in the root ignores only `data/` and `videos/` as requested.

---

## Python – Training and Desktop Runner

### 1) Environment
```powershell
cd C:\Users\jairo\Downloads\AI
python -m venv .venv
. .\.venv\Scripts\Activate.ps1
pip install -r scripts\requirements.txt
```

### 2) Train the TensorFlow model and export TFLite
```powershell
python scripts\train_tf_recognizer.py
```
Outputs land in `scripts\output\`:
- `model.tflite`
- `labels.json`
- `confusion_matrix.png`
- `saved_model/`

### 3) Realtime TFLite runner (desktop)
```powershell
python scripts\run_tflite_faces_realtime.py
```
This will open your webcam, detect a face, crop/resize to 224×224, and run the TFLite classifier. It includes robustness (dtype casting, contiguous arrays, fresh-interpreter fallback if output zeros).

---

## Flutter Android App – Realtime Face Classification

### 1) Copy model + labels into app assets
```powershell
cp .\scripts\output\model.tflite .\flutter_face_app\assets\model.tflite ; cp .\scripts\output\labels.json .\flutter_face_app\assets\labels.json
```

### 2) Run on device (debug)
```powershell
cd .\flutter_face_app
flutter pub get
flutter run
```

### 3) Build APK (release)
- Universal APK:
```powershell
flutter clean
flutter pub get
flutter build apk --release
```
- Per‑ABI APKs (smaller):
```powershell
flutter build apk --release --split-per-abi
```
Artifacts are under `flutter_face_app\build\app\outputs\flutter-apk\`.

### What the app does
- Uses `camera` for frames, ML Kit to detect faces, crops the largest face from the Y plane, resizes to 224×224.
- Replicates Y (luma) into RGB channels for fast TFLite inference (optionally switch to full YUV→RGB for color‑dependent models).
- Draws a green square that follows the face.
- Shows Top‑4 labels with probabilities in the bottom bar, updated ~every 5 frames.
- Includes a 30s warm‑up hint while probabilities already display.
- Front/back camera toggle.

### Tuning (in `lib/main.dart`)
- `INPUT_SIZE = 224`
- `UPDATE_FRAMES = 5`
- `WARMUP_SECONDS = 30.0`
- `TOP_K = 4`

### Troubleshooting
- Asset not found: Ensure assets exist and are declared in `pubspec.yaml`.
- ML Kit format errors: We feed NV21 bytes; that path is already implemented in `mlkit_service.dart`.
- TFLite shape/dtype errors: The app inspects tensor metadata and builds inputs/outputs accordingly.
- Performance: Consider NNAPI/GPU delegate or a quantized model.

---

## Git – Pushing to GitHub (repo: Musgus/blinders_tflite)

From the workspace root:
```powershell
cd C:\Users\jairo\Downloads\AI
# If not initialized yet
git init
# Only data/ and videos/ are ignored per .gitignore
git add -A
git commit -m "feat: Flutter app + TF/TFLite pipeline and scripts"

git branch -M main
# Set or update origin to the correct repo
git remote remove origin 2>$null
git remote add origin https://github.com/Musgus/blinders_tflite.git

git push -u origin main
```
If the remote has history and push is rejected:
```powershell
git fetch origin
git pull --rebase origin main
git push -u origin main
# or create a new branch
# git checkout -b jairo-inicial
# git push -u origin jairo-inicial
```

---

## Next Steps & Optimizations
- Full YUV→RGB conversion on the face crop for color‑sensitive models
- NNAPI/GPU delegates for TFLite
- Quantization (int8/FP16) for size/speed
- Smoothing filters (box + probabilities) and FPS metrics

---

All set. Clone, train, copy assets, run the app, and you’re live.
