Flutter TFLite Face Recognizer (Android)

Overview
- Live camera preview with face detection (ML Kit)
- Tracks the largest face and draws a green square that follows it
- Crops and resizes the face to 224×224 and runs a TFLite classifier
- Bottom bar shows Top‑4 labels with probabilities (updates ~every 5 frames)
- Warm‑up hint: shows “Preparando… Ns” for 30s while still displaying probabilities
- Front/back camera toggle via floating button

Project layout (this app)
- `lib/main.dart`: UI + orchestration, drawing overlay, camera switching
- `lib/pipeline/mlkit_service.dart`: ML Kit face detection wrapper (NV21 bytes input)
- `lib/pipeline/image_utils.dart`: YUV_420_888 → NV21 converter; crop+resize from Y plane to RGB (Y replicated)
- `lib/pipeline/tflite_service.dart`: interpreter wrapper; shapes input/output and returns probabilities
- `assets/`: `model.tflite`, `labels.json`

Requirements
- Flutter SDK installed (stable channel)
- Android device/emulator
- This repo already declares camera permission; make sure your device grants it at runtime

Copy model + labels (from Python output)
Run in the workspace root (PowerShell):

```powershell
cp .\scripts\output\model.tflite .\flutter_face_app\assets\model.tflite ; cp .\scripts\output\labels.json .\flutter_face_app\assets\labels.json
```

Run the app (debug)
```powershell
cd c:\Users\jairo\Downloads\AI\flutter_face_app
flutter pub get
flutter run
```

Build a release APK
```powershell
cd c:\Users\jairo\Downloads\AI\flutter_face_app
flutter clean
flutter pub get
flutter build apk --release
```
APK result: `build\app\outputs\flutter-apk\app-release.apk`

Smaller APKs per ABI (recommended)
```powershell
flutter build apk --release --split-per-abi
```
Outputs:
- app-armeabi-v7a-release.apk
- app-arm64-v8a-release.apk
- app-x86_64-release.apk

Tuning knobs (in `main.dart`)
- `INPUT_SIZE = 224` — model input size
- `UPDATE_FRAMES = 5` — update probabilities every N frames
- `WARMUP_SECONDS = 30.0` — warm‑up countdown (while still showing probs)
- `TOP_K = 4` — number of labels shown in overlay

Troubleshooting
- Unable to load asset: Ensure `assets/model.tflite` and `assets/labels.json` exist and are declared in `pubspec.yaml`.
- ImageFormat not supported: We convert YUV_420_888 → NV21 internally; this resolves ML Kit format issues.
- Interpreter shape mismatch: The app inspects tensor shapes and builds inputs/outputs accordingly; if your model shape changes, it should adapt.
- Low FPS: try increasing `UPDATE_FRAMES`, and consider NNAPI/GPU delegates or model quantization (int8/FP16).

Notes
- We replicate Y (luma) into RGB for inference to keep it fast and consistent. If your model needs full color, we can switch to a proper YUV→RGB conversion on the face crop.
- Front camera overlay is mirrored to match the preview.

License
This app code is provided as-is for your project. Add a license file if you intend to distribute.

