# Hybrid Proctoring — MediaPipe + DeepFace Identity Verification

## Overview

This test evaluates a hybrid face-recognition pipeline for the **AI-powered Smart Examination / Proctoring Platform**.

The system combines:

* **MediaPipe** — real-time face detection and face-count monitoring
* **DeepFace** — face verification
* **RetinaFace** — face detection/alignment used internally by DeepFace during verification
* **FaceNet512** — face embedding/recognition model
* **OpenCV** — webcam capture and visualization

The objective of this phase is to verify whether the person appearing in the webcam matches the registered student's reference image.

---

## System Architecture

```text
                    Webcam
                       │
                       ▼
              OpenCV Video Capture
                       │
                       ▼
              MediaPipe Face Detection
                       │
             ┌─────────┼─────────┐
             │         │         │
             ▼         ▼         ▼
          0 Faces    1 Face    2+ Faces
             │         │         │
             ▼         │         ▼
          NO FACE      │    MULTIPLE FACES
                       │
                       ▼
              DeepFace Verification
                       │
                       ▼
                  RetinaFace
                       │
                       ▼
                   FaceNet512
                       │
                       ▼
              Cosine Distance
                       │
              ┌────────┴────────┐
              ▼                 ▼
          Distance ≤ 0.30   Distance > 0.30
              │                 │
              ▼                 ▼
           VERIFIED       IDENTITY MISMATCH
```

---

## Environment

The successful test was performed using:

| Component  |    Version |
| ---------- | ---------: |
| Python     |       3.11 |
| TensorFlow |     2.21.0 |
| Keras      |     3.15.1 |
| tf-keras   |     2.21.0 |
| DeepFace   |    0.0.100 |
| RetinaFace |     0.0.18 |
| MediaPipe  |      1.0.0 |
| Face Model | FaceNet512 |
| Detector   | RetinaFace |
| OS         |      macOS |
| CPU        |   Apple M4 |

---

## Important Keras Configuration

TensorFlow 2.21 and modern Keras require legacy Keras mode for the RetinaFace implementation used in this environment.

Therefore, the following configuration must appear **before importing DeepFace**:

```python
import os

os.environ["TF_USE_LEGACY_KERAS"] = "1"

import cv2
import time
import traceback
import mediapipe as mp
from deepface import DeepFace
```

Without this configuration, RetinaFace produced:

```text
A KerasTensor cannot be used as input to a TensorFlow function
```

After enabling legacy Keras mode, RetinaFace and DeepFace verification worked correctly.

---

# Project Structure

The relevant project structure is:

```text
hybrid-proctoring/
│
├── tests/
│   ├── models/
│   │   └── blaze_face_short_range.tflite
│   │
│   ├── test_mediapipe_image.py
│   ├── mediapipe_deepface.py
│   └── live_hybrid_verify.py
│
├── test_data/
│   ├── database/
│   │   ├── student_1/
│   │   │   └── student_1.jpg
│   │   │
│   │   ├── student_2/
│   │   │   └── student_2.jpg
│   │   │
│   │   └── student_3/
│   │       └── student_3.jpg
│   │
│   └── queries/
│       └── student_1_2.jpeg
│
└── results/
    └── debug_frame.jpg
```

---

# 1. MediaPipe Face Detection Test

MediaPipe was first tested independently to verify that it could detect faces in images and webcam frames.

The BlazeFace model used was:

```text
tests/models/blaze_face_short_range.tflite
```

### Image Test

Example:

```bash
python tests/test_mediapipe_image.py
```

Successful result:

```text
Image: test_data/database/student_1/student_1.jpg
Image size: (2200, 2295, 3)
Faces detected: 1
Face 1: score=0.988
```

The system was also tested with:

* 0 faces
* 1 face
* 2 faces

The live detector achieved approximately:

```text
23–25 FPS
```

during testing.

---

# 2. DeepFace Multi-Student Recognition Test

Three registered students were placed into the database:

```text
student_1
student_2
student_3
```

Database:

```text
test_data/database/
```

The test queries were:

```text
test_data/queries/
```

The system successfully recognized all three students.

### Student 1

```text
Matched identity: student_1
Distance: 0.126603
Threshold: 0.300000
Result: RECOGNIZED
```

### Student 2

```text
Matched identity: student_2
Distance: 0.186721
Threshold: 0.300000
Result: RECOGNIZED
```

### Student 3

```text
Matched identity: student_3
Distance: 0.238716
Threshold: 0.300000
Result: RECOGNIZED
```

An unknown person was also tested:

```text
Testing: Unknown
Query: test_data/queries/unknown_test.jpg
Result: NOT RECOGNIZED
```

This demonstrated that the recognition system could distinguish registered students from an unknown person in the prepared test data.

---

# 3. DeepFace Live Verification Test

A webcam-based DeepFace verification test was performed before integrating MediaPipe.

The reference image was:

```text
test_data/database/student_1/student_1.jpg
```

The webcam frame was compared against this reference image using:

```python
DeepFace.verify(
    img1_path=REFERENCE_IMAGE,
    img2_path=frame,
    model_name="Facenet512",
    detector_backend="retinaface",
    enforce_detection=False
)
```

The test demonstrated that the system could recognize the registered student and reject another person.

---

# 4. Hybrid MediaPipe + DeepFace Test

The final test combines both systems.

### MediaPipe responsibility

MediaPipe performs the fast real-time face detection.

It determines:

```text
0 faces
1 face
2+ faces
```

The system handles these states as:

| Face Count | Status                |
| ---------: | --------------------- |
|          0 | `NO FACE`             |
|          1 | Continue verification |
|         2+ | `MULTIPLE FACES`      |

DeepFace verification is performed only when exactly one face is detected.

---

## DeepFace Configuration

The identity verification uses:

```text
Model: FaceNet512
Detector: RetinaFace
Metric: Cosine
Threshold: 0.30
```

Verification is performed every:

```text
3 seconds
```

rather than on every webcam frame.

This prevents the computationally expensive DeepFace verification process from running continuously.

---

# 5. Live Hybrid Verification

Run:

```bash
python tests/live_hybrid_verify.py
```

The webcam window displays:

```text
Faces: 1
Status: VERIFIED
Distance: 0.215
```

or:

```text
Faces: 1
Status: IDENTITY MISMATCH
Distance: 0.350
```

Possible states include:

```text
WAITING
NO FACE
MULTIPLE FACES
VERIFIED
IDENTITY MISMATCH
VERIFICATION ERROR
```

Press:

```text
Q
```

to exit the webcam window.

---

# 6. Verification Decision

The current DeepFace threshold is:

```text
0.300000
```

The decision is:

```text
Distance <= 0.30
        ↓
    VERIFIED
```

and:

```text
Distance > 0.30
        ↓
IDENTITY MISMATCH
```

Example successful verification:

```text
Verification: True
Distance: 0.215410
Threshold: 0.300000
```

Example mismatch:

```text
Verification: False
Distance: 0.350051
Threshold: 0.300000
```

---

# 7. Final Live Test Results

During the final hybrid test, the following results were observed:

```text
Verification: False | Distance: 1.004767 | Threshold: 0.300000
Verification: False | Distance: 0.335600 | Threshold: 0.300000
Verification: False | Distance: 0.317943 | Threshold: 0.300000
Verification: False | Distance: 0.334567 | Threshold: 0.300000

Verification: True | Distance: 0.287649 | Threshold: 0.300000
Verification: True | Distance: 0.283795 | Threshold: 0.300000
Verification: True | Distance: 0.262638 | Threshold: 0.300000
Verification: True | Distance: 0.277837 | Threshold: 0.300000
Verification: True | Distance: 0.215410 | Threshold: 0.300000

Verification: False | Distance: 0.350051 | Threshold: 0.300000
```

This demonstrates that the hybrid pipeline is operational.

---

# 8. Performance Considerations

MediaPipe is used as the real-time detection layer because it is considerably faster than repeatedly running DeepFace.

The current approach is:

```text
MediaPipe
    ↓
Fast continuous detection
    ↓
Check face count
    ↓
DeepFace every 3 seconds
    ↓
Identity verification
```

This avoids:

```text
Webcam frame
    ↓
DeepFace
    ↓
RetinaFace
    ↓
FaceNet512
```

being executed on every frame.

---

# 9. Debugging and Compatibility

During development, the following RetinaFace error occurred:

```text
ValueError:
A KerasTensor cannot be used as input to a TensorFlow function
```

The installed versions were:

```text
TensorFlow 2.21.0
Keras 3.15.1
DeepFace 0.0.100
RetinaFace 0.0.18
```

`tf-keras` was installed:

```text
tf-keras 2.21.0
```

The issue was resolved by enabling:

```bash
TF_USE_LEGACY_KERAS=1
```

or equivalently inside Python:

```python
import os
os.environ["TF_USE_LEGACY_KERAS"] = "1"
```

This must happen before importing DeepFace/TensorFlow-dependent components.

---

# 10. Current Limitations

This test is a **prototype evaluation**, not yet a production-ready examination proctoring system.

Current limitations include:

1. Only a small number of student identities have been tested.
2. The current threshold `0.30` has not been statistically optimized.
3. Lighting conditions have not been systematically evaluated.
4. Different camera angles have not been systematically evaluated.
5. Significant changes in distance from the camera have not been evaluated.
6. Multiple genuine images per student have not yet been used for threshold evaluation.
7. False Acceptance Rate (FAR) and False Rejection Rate (FRR) have not yet been calculated.
8. Identity verification currently runs every 3 seconds.
9. The system has not yet implemented temporal identity consistency.
10. No complete exam-session violation scoring system has been implemented yet.

---

# 11. Next Evaluation Phase

The next recommended experiment is **Face Recognition Accuracy Evaluation**.

Instead of testing only a few examples, create multiple images for each student:

```text
student_1/
    image_01.jpg
    image_02.jpg
    image_03.jpg
    ...

student_2/
    image_01.jpg
    image_02.jpg
    image_03.jpg
    ...

student_3/
    image_01.jpg
    image_02.jpg
    image_03.jpg
    ...
```

Then generate:

### Genuine pairs

```text
Student 1 ↔ Student 1
Student 2 ↔ Student 2
Student 3 ↔ Student 3
```

### Impostor pairs

```text
Student 1 ↔ Student 2
Student 1 ↔ Student 3
Student 2 ↔ Student 3
```

From these results, calculate:

```text
Genuine distance distribution
Impostor distance distribution
FAR
FRR
EER
Optimal threshold
Accuracy
Precision
Recall
F1-score
```

This will provide a much stronger experimental foundation for the university project than using the default `0.30` threshold without statistical evaluation.

---

## Conclusion

The current prototype successfully demonstrates the core identity-verification pipeline:

```text
MediaPipe
   +
DeepFace
   +
RetinaFace
   +
FaceNet512
   +
OpenCV
```

The final hybrid system can:

* Detect whether a face is present.
* Detect multiple faces.
* Verify a single person's identity.
* Recognize the registered student.
* Reject an identity mismatch.
* Perform real-time face detection at approximately 23–25 FPS.
* Perform DeepFace verification periodically rather than on every frame.
* Operate successfully with the current TensorFlow 2.21 / Keras 3 environment after enabling legacy Keras compatibility.

**Current phase status: `COMPLETED — Hybrid Face Detection + Identity Verification Prototype`**.
