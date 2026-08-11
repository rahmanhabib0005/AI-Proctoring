# DeepFace Evaluation

This project is an isolated evaluation environment for testing **DeepFace** as a potential AI component for the AI-powered online examination proctoring system.

The goal is to test DeepFace independently before integrating it into the complete proctoring platform.

---

## 1. Environment

### Operating System

* macOS
* Apple Silicon — MacBook Air M4

### Python

```bash
Python 3.11.15
```

### Conda Environment

```bash
deepface-test
```

Activate the environment:

```bash
conda activate deepface-test
```

Verify Python:

```bash
python --version
which python
```

Expected:

```text
Python 3.11.15
/opt/homebrew/Caskroom/miniforge/base/envs/deepface-test/bin/python
```

---

## 2. Installed Packages

The main packages currently being used are:

| Package    | Version |
| ---------- | ------: |
| Python     | 3.11.15 |
| DeepFace   | 0.0.100 |
| TensorFlow |  2.21.0 |
| Keras      |  3.15.1 |
| tf-keras   |  2.21.0 |
| OpenCV     |   5.0.0 |
| NumPy      |   2.4.6 |

Check DeepFace:

```bash
python -m pip show deepface
```

Check TensorFlow:

```bash
python -m pip show tensorflow
```

Check Keras:

```bash
python -m pip show keras
```

Check `tf-keras`:

```bash
python -m pip show tf-keras
```

---

## 3. Installation

Create the Conda environment:

```bash
conda create -n deepface-test python=3.11
```

Activate it:

```bash
conda activate deepface-test
```

Install DeepFace:

```bash
python -m pip install deepface
```

Verify installation:

```bash
python -c "import deepface; print('DeepFace installed')"
```

---

## 4. TensorFlow / Keras Compatibility

During the initial DeepFace test, the following error appeared:

```text
ModuleNotFoundError: No module named 'tf_keras'
```

DeepFace's RetinaFace dependency reported:

```text
You have tensorflow 2.21.0 and this requires tf-keras package.
```

Install the required compatibility package:

```bash
python -m pip install tf-keras
```

Verify:

```bash
python -c "import tf_keras; print('tf-keras OK')"
```

Successful result:

```text
tf-keras OK
```

The final environment therefore uses:

```text
TensorFlow 2.21.0
Keras 3.15.1
tf-keras 2.21.0
```

---

## 5. Verify the Complete Environment

Run:

```bash
python --version
which python
python -m pip show deepface
python -m pip show tensorflow
python -m pip show keras
python -m pip show tf-keras
```

Quick import test:

```bash
python -c "from deepface import DeepFace; print('DeepFace OK')"
```

TensorFlow test:

```bash
python -c "import tensorflow as tf; print('TensorFlow:', tf.__version__)"
```

OpenCV test:

```bash
python -c "import cv2; print('OpenCV:', cv2.__version__)"
```

tf-keras test:

```bash
python -c "import tf_keras; print('tf-keras OK')"
```

---

## 6. Project Structure

The evaluation project was created under:

```text
~/auto-oep-test/deepface-evaluation
```

Current structure:

```text
deepface-evaluation/
├── test_data/
│   └── images/
│       ├── student_1.jpeg
│       └── student_1_2.jpeg
│
├── tests/
│   └── face_verification.py
│
├── results/
│
└── README.md
```

The two images were used to test whether DeepFace recognizes them as belonging to the same person.

---

## 7. Test Images

The following images were prepared:

```text
test_data/images/student_1.jpeg
test_data/images/student_1_2.jpeg
```

These represent two images of the same student/person for the initial face-verification experiment.

---

## 8. Face Verification Test

The first DeepFace experiment tested:

```text
Image A
   +
Image B
   ↓
DeepFace Face Verification
   ↓
Same person?
```

The test was executed using:

```bash
python tests/face_verification.py
```

---

## 9. First Compatibility Error

The first execution produced:

```text
ModuleNotFoundError: No module named 'tf_keras'
```

This was caused by the installed TensorFlow/Keras configuration required by the RetinaFace detector used by DeepFace.

The issue was resolved by installing:

```bash
python -m pip install tf-keras
```

After installation:

```bash
python -c "import tf_keras; print('tf-keras OK')"
```

Result:

```text
tf-keras OK
```

---

## 10. RetinaFace Model Download

During the successful verification test, DeepFace automatically downloaded the RetinaFace model:

```text
retinaface.h5
```

Download size:

```text
119 MB
```

The model was stored in the DeepFace weights directory:

```text
~/.deepface/weights/retinaface.h5
```

This means the model does not need to be downloaded again for subsequent tests unless the cache is removed.

---

## 11. Successful Verification Result

The successful test produced:

```text
## DeepFace Verification Result

Verified: True
Distance: 0.171335
Threshold: 0.3
Model: Facenet512
Detector: retinaface
```

### Interpretation

```text
Distance = 0.171335
Threshold = 0.3
```

Because:

```text
0.171335 < 0.3
```

DeepFace determined that the two images represent the same identity.

Therefore:

```text
Verified: True
```

---

## 12. Current DeepFace Status

### Completed

* [x] Conda environment created
* [x] Python 3.11 configured
* [x] DeepFace installed
* [x] TensorFlow installed
* [x] Keras installed
* [x] `tf-keras` compatibility issue resolved
* [x] OpenCV verified
* [x] Test project created
* [x] Student test images added
* [x] DeepFace imported successfully
* [x] RetinaFace model downloaded
* [x] Face verification executed
* [x] Face verification returned `True`

### Current Result

```text
DeepFace: WORKING
Face Verification: WORKING
FaceNet512: WORKING
RetinaFace: WORKING
```

---

## 13. Why We Are Testing DeepFace Separately

DeepFace is **not yet being integrated into the complete examination system**.

The purpose of this evaluation is to determine which AI technologies are actually useful for the final project.

The planned approach is:

```text
Test AI Model
     ↓
Measure Accuracy
     ↓
Measure FPS / Performance
     ↓
Test False Positives
     ↓
Test False Negatives
     ↓
Evaluate Usefulness for Proctoring
     ↓
Keep or Remove Model
```

This prevents unnecessary AI components from being included in the final system.

---

## 14. Planned DeepFace Tests

The next tests will be performed independently.

### Test 1 — Face Verification

Status:

```text
COMPLETED
```

Purpose:

Determine whether two images belong to the same person.

---

### Test 2 — Face Recognition

Determine whether a live/test image matches a registered student.

Example:

```text
Registered Student
        ↓
Reference Image
        ↓
DeepFace
        ↓
Camera Image
        ↓
Identity Match
```

---

### Test 3 — Face Detection

Determine whether a face is present in the camera frame.

---

### Test 4 — Multiple Face Detection

Test situations such as:

```text
One student
    ↓
Normal

Two people
    ↓
Potential Proctoring Violation
```

---

### Test 5 — Real-Time Webcam

Test DeepFace against a live camera stream.

---

### Test 6 — Identity Change

Example:

```text
Student A starts examination
        ↓
Camera continuously monitored
        ↓
Student B appears
        ↓
Identity mismatch
        ↓
Generate event
```

---

### Test 7 — Performance Testing

Measure:

* FPS
* Processing time
* CPU usage
* Memory usage
* Detection latency
* Recognition latency

This is especially important for running the final system on a MacBook Air M4.

---

## 15. Planned Proctoring Integration

Eventually DeepFace may become one component of the larger AI proctoring pipeline:

```text
                 Camera Stream
                       │
                       ▼
              ┌─────────────────┐
              │ Frame Processing │
              └────────┬────────┘
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
      DeepFace        YOLO      MediaPipe
          │            │            │
          ▼            ▼            ▼
    Face Identity   Objects     Landmarks
          │            │            │
          └────────────┼────────────┘
                       ▼
               Behavior Analysis
                       │
                       ▼
                 Event Detection
                       │
                       ▼
                Risk Assessment
                       │
                       ▼
                Alert Generation
```

DeepFace would primarily contribute to:

* Face verification
* Face recognition
* Identity monitoring
* Identity-change detection

It should **not** be responsible for every proctoring decision.

---

## 16. Important Evaluation Principle

A detection result is not automatically a proctoring violation.

For example:

```text
YOLO detects:
cell phone
```

does not necessarily mean:

```text
Student is cheating
```

Similarly:

```text
DeepFace identity mismatch
```

should not immediately mean:

```text
Exam violation confirmed
```

The final system should combine multiple signals and use a risk/event model.

Example:

```text
Face mismatch
      +
Unknown person
      +
Persistent mismatch
      ↓
High-confidence identity alert
```

This will be evaluated later.

---

## 17. Next Step

The next DeepFace experiment should be:

```text
Face Recognition
```

using the existing student images.

After that:

```text
Face Detection
        ↓
Multiple Face Detection
        ↓
Live Webcam
        ↓
Performance Testing
        ↓
Proctoring Simulation
```

The goal is to determine whether DeepFace is actually necessary for the final AI examination proctoring system.
