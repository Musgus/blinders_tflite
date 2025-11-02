Flutter TFLite Face Recognizer - Starter

What this provides
- A minimal Flutter app scaffold at `flutter_face_app/` that:
  - uses the `camera` plugin for camera frames
  - suggests `google_mlkit_face_detection` for face detection
  - uses `tflite_flutter` and `tflite_flutter_helper` to run the provided `model.tflite`
  - updates the displayed prediction every 10 seconds (configurable)

Important: copy model and labels
1. From your Python project, copy the TFLite model and labels into the Flutter project assets:

   Windows PowerShell (from workspace root):

   cp .\scripts\output\model.tflite .\flutter_face_app\assets\model.tflite ; cp .\scripts\output\labels.json .\flutter_face_app\assets\labels.json

2. Run in the flutter app folder:

   cd flutter_face_app
   flutter pub get
   flutter run -d <your_android_device>

Notes and TODOs
- The scaffold `lib/main.dart` contains TODOs for converting `CameraImage` (YUV) to RGB `TensorImage` which is required for both ML Kit and TFLite. There are multiple community tested snippets for that conversion; search for "camera image to inputimage flutter nv21" or use `camera` plugin sample code plus `tflite_flutter_helper` utilities.
- On Android you must add camera permission to `android/app/src/main/AndroidManifest.xml`:

  <uses-permission android:name="android.permission.CAMERA" />

- ML Kit: the plugin `google_mlkit_face_detection` may require AndroidX and configuration in Gradle. Follow the plugin README.
- Performance: consider using NNAPI or GPU delegate for `tflite_flutter` on-device. You can also quantize the model (FP16 or int8) for faster inference.

Next steps I can help with
- Provide a tested `CameraImage` -> `TensorImage` conversion snippet that works on Android (I can add it into `main.dart`).
- Wire ML Kit detection properly using `InputImage.fromBytes` and plane metadata.
- Add Android Gradle settings or example `build.gradle` changes, if you want me to scaffold them.

