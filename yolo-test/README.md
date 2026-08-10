# AI Exam Proctoring — Model Evaluation

This project is an independent evaluation environment for testing AI/ML models that may be useful for an AI-powered online examination proctoring system.

The purpose is **not to copy the complete AutoOEP implementation immediately**. Instead, each model is being installed, tested, evaluated, and compared independently.

The final system will only use models that provide sufficient accuracy and practical value.

---

# Current Evaluation Progress

| Model / Technology | Purpose                              | Installation | Basic Test | Status  |
| ------------------ | ------------------------------------ | -----------: | ---------: | ------- |
| MediaPipe          | Face, hand, pose, landmark detection |            ✅ |          ✅ | Testing |
| YOLO               | Object detection                     |            ✅ |          ✅ | Testing |
| DeepFace           | Face recognition / verification      |            ⬜ |          ⬜ | Pending |
| LSTM               | Sequential behavior analysis         |            ⬜ |          ⬜ | Pending |
| LightGBM           | Static behavior classification       |            ⬜ |          ⬜ | Pending |
| PyTorch            | Deep learning framework              |            ⬜ |          ⬜ | Pending |

---

# 1. Development Environment

Primary development machine:

* MacBook Air M4
* Apple Silicon (`arm64`)
* macOS 26
* Homebrew
* Miniforge / Conda
* Python 3.11 for the current AI testing environments

We initially experimented with system Python and Homebrew Python versions, but encountered compatibility problems with Python 3.9, 3.12, and 3.14.

The most reliable environment for the current AI experiments is:

```text
Conda
    ↓
Python 3.11
    ↓
Individual environment per model
```

---

# 2. Why Separate Environments?

Each AI model may require different:

* Python versions
* NumPy versions
* OpenCV versions
* TensorFlow versions
* PyTorch versions
* Native libraries
* Hardware acceleration support

Therefore, models are being tested independently.

Example:

```text
auto-oep-test/
│
├── mediapipe-evaluation/
│
├── yolo-test/
│
├── deepface-test/
│
├── lstm-test/
│
└── lightgbm-test/
```

This prevents one model's dependencies from breaking another model.

---

# 3. MediaPipe Evaluation

## 3.1 Environment

Conda environment:

```bash
conda activate mediapipe-test
```

Python version:

```bash
python --version
```

Current environment:

```text
Python 3.11.15
```

Check Python:

```bash
which python
```

Expected:

```text
/opt/homebrew/Caskroom/miniforge/base/envs/mediapipe-test/bin/python
```

---

# 4. MediaPipe Installation

MediaPipe was successfully installed in the Conda Python 3.11 environment.

Check installation:

```bash
python -m pip install mediapipe
```

Verify:

```bash
python -c "import mediapipe as mp; print(mp.__version__)"
```

Successful result:

```text
1.0.0
```

Therefore:

```text
MediaPipe
    ↓
Installed successfully
    ↓
Python 3.11
    ↓
Import successful
```

---

# 5. MediaPipe Evaluation Project

Project structure created:

```text
mediapipe-evaluation/
│
├── tests/
│   ├── face_detection/
│   ├── face_landmarks/
│   ├── hand_tracking/
│   ├── pose_estimation/
│   └── holistic/
│
├── test_data/
│   ├── images/
│   └── videos/
│
├── results/
│   ├── screenshots/
│   └── videos/
│
└── requirements.txt
```

Initial requirements:

```text
mediapipe==1.0.0
```

Installation:

```bash
python -m pip install -r requirements.txt
```

Result:

```text
Requirement already satisfied:
mediapipe==1.0.0
```

---

# 6. MediaPipe Webcam Testing

A webcam-based MediaPipe test was attempted.

Initial result:

```text
OpenCV: not authorized to capture video
OpenCV: camera failed to properly initialize!
Error: Could not open webcam.
```

This was identified as a macOS camera-permission issue rather than a MediaPipe installation issue.

A later MediaPipe model test also required the appropriate MediaPipe model asset, such as:

```text
blaze_face_short_range
```

This demonstrated an important point:

> Installing the MediaPipe Python package does not automatically provide every model asset required by the Tasks API.

The package installation itself is successful.

---

# 7. MediaPipe Current Status

### Completed

* [x] Conda environment created
* [x] Python 3.11 configured
* [x] MediaPipe installed
* [x] MediaPipe imported successfully
* [x] Version verified
* [x] Evaluation project structure created
* [x] Webcam testing attempted
* [x] Model-asset requirement identified

### Pending

* [ ] Face detection evaluation
* [ ] Face landmark evaluation
* [ ] Hand tracking evaluation
* [ ] Pose estimation evaluation
* [ ] Holistic evaluation
* [ ] Video testing
* [ ] Accuracy measurements
* [ ] FPS/performance measurements

---

# 8. YOLO Evaluation

YOLO is being tested separately from MediaPipe.

Conda environment:

```bash
conda activate yolo-test
```

Python:

```text
Python 3.11.15
```

Check:

```bash
python --version
which python
```

Expected:

```text
Python 3.11.15

/opt/homebrew/Caskroom/miniforge/base/envs/yolo-test/bin/python
```

---

# 9. YOLO Installation

Ultralytics was successfully installed.

Verify:

```bash
python -c "import ultralytics; print(ultralytics.__version__)"
```

Successful result:

```text
8.4.117
```

Therefore:

```text
Ultralytics YOLO
        ↓
Version 8.4.117
        ↓
Installation successful
```

---

# 10. YOLO Evaluation Project

Project structure:

```text
yolo-test/
│
├── test_data/
│   ├── images/
│   └── videos/
│
├── tests/
│   ├── image_test.py
│   └── video_test.py
│
├── results/
│   ├── images/
│   ├── videos/
│   └── reports/
│
└── README.md
```

The directory structure was created using:

```bash
mkdir -p test_data/images
mkdir -p test_data/videos

mkdir -p tests

mkdir -p results/images
mkdir -p results/videos
mkdir -p results/reports
```

Important:

The tree representation should **not** be pasted into the terminal. It is documentation only.

---

# 11. YOLO Image Test

The first image test used the standard Ultralytics YOLO model.

The model successfully downloaded the required pretrained model and processed an image.

Example:

```text
image 1/1 .../bus.jpg:
640x480
4 persons
1 bus
```

Inference speed was approximately:

```text
Preprocess: 2.5 ms
Inference: 32.4 ms
Postprocess: 0.8 ms
```

The resulting image was successfully saved:

```text
results/images/yolo_test.jpg
```

Therefore:

* [x] Model loaded
* [x] Image downloaded
* [x] Image inference completed
* [x] Objects detected
* [x] Annotated image generated

---

# 12. YOLO Real Image Testing

Additional images were placed in:

```text
test_data/images/
```

Images included:

```text
images.jpeg
black-flashlight-1291568.webp
student-girl-writing-high-school-260nw-2734242461.webp
```

The image evaluation script successfully processed multiple images.

Example results:

```text
Testing: images.jpeg
No objects detected.
```

Another image:

```text
Testing: black-flashlight-1291568.webp

car: 49.24%

Saved:
results/images/detected_black-flashlight-1291568.jpg
```

Student/exam-related image:

```text
Testing: student-girl-writing-high-school-260nw-2734242461.webp

person: 91.22%
person: 91.17%
person: 81.22%
chair: 72.57%
chair: 44.01%
dining table: 41.96%

Saved:
results/images/detected_student-girl-writing-high-school-260nw-2734242461.jpg
```

---

# 13. YOLO False Detection Observation

During testing, several false or inappropriate detections were observed.

Examples:

```text
Calculator
    ↓
Detected as:
cell phone
```

Another observation:

```text
Printed/visual content
    ↓
Detected as:
keyboard
laptop
```

This is an important finding.

The current pretrained YOLO model is a **general-purpose object detector**, not an examination-specific object detector.

Therefore, its output cannot directly be interpreted as:

```text
YOLO detected phone
        ↓
Student is cheating
```

Instead, it should initially be treated as:

```text
Possible object detected
        ↓
Further verification
        ↓
Behavior analysis
        ↓
Risk calculation
```

---

# 14. YOLO Video Testing

A real exam video was added:

```text
test_data/videos/exam.mp4
```

Video size:

```text
13 MB
```

The video test successfully processed:

```text
Frames processed: 546
Original FPS: 29.97
```

Detected objects:

```text
person: 586
dining table: 284
laptop: 138
keyboard: 87
cell phone: 30
chair: 14
book: 7
```

Output video:

```text
results/videos/detected_exam.mp4
```

Therefore:

* [x] Video opened
* [x] Frames processed
* [x] YOLO inference performed
* [x] Objects detected
* [x] Detection counts generated
* [x] Annotated output video generated

---

# 15. Important YOLO Video Observation

Detection counts represent **detections across frames**, not necessarily unique physical objects.

For example:

```text
person: 586
```

does not mean:

```text
586 different people
```

It may represent:

```text
1 person
×
many frames
=
many detections
```

Similarly:

```text
cell phone: 30
```

could represent the same object detected repeatedly across multiple frames.

For the final proctoring system, this must be converted into an event/persistence system.

---

# 16. Why YOLO Alone Is Not Enough

Current findings indicate that YOLO can provide useful object-level evidence, but it should not directly generate cheating decisions.

A better architecture is:

```text
YOLO
 │
 ├── Possible phone
 ├── Possible book
 ├── Possible laptop
 └── Possible unauthorized object
          │
          ▼
     Persistence Check
          │
          ▼
     Behavior Analysis
          │
          ▼
     Risk Engine
          │
          ▼
        Alert
```

For example:

```text
YOLO:
Possible phone detected

+

MediaPipe:
Student looking toward object

+

MediaPipe:
Hand moving toward object

+

Temporal model:
Suspicious sequence continues

↓

High-risk event
```

This is the intended direction for the final AI proctoring architecture.

---

# 17. Current Model Evaluation Architecture

The project is currently following this methodology:

```text
                AI Proctoring Models
                       │
        ┌──────────────┼──────────────┐
        │              │              │
        ▼              ▼              ▼
    MediaPipe        YOLO         DeepFace
        │              │              │
        ▼              ▼              ▼
     Testing        Testing        Testing
        │              │              │
        └──────────────┼──────────────┘
                       │
                       ▼
                Compare Results
                       │
                       ▼
              Decide Which Models
                 Are Necessary
```

The objective is **not to automatically include all models from the research paper**.

Each model must demonstrate practical value first.

---

# 18. Planned Evaluation Order

The next models should be tested independently.

Recommended order:

```text
1. MediaPipe
      ↓
2. YOLO
      ↓
3. DeepFace
      ↓
4. LSTM
      ↓
5. LightGBM
      ↓
6. PyTorch
      ↓
7. Multi-model integration
```

---

# 19. Planned DeepFace Test

DeepFace will be evaluated for:

```text
Face verification
Face recognition
Identity matching
Wrong-person detection
```

Example:

```text
Registered Student
        │
        ▼
Reference Image
        │
        ▼
DeepFace
        │
        ├── Same person
        └── Different person
```

---

# 20. Planned LSTM Test

LSTM will not initially be used for direct object detection.

It will be evaluated for **temporal behavior analysis**.

Example sequence:

```text
Normal
 ↓
Looking away
 ↓
Hand movement
 ↓
Looking toward side
 ↓
Repeated movement
```

The LSTM can potentially learn whether a sequence resembles suspicious behavior.

---

# 21. Planned LightGBM Test

LightGBM can be evaluated using structured features such as:

```text
face_missing_duration
number_of_faces
head_turn_count
gaze_away_duration
hand_movement_count
phone_detection_count
object_detection_confidence
suspicious_event_count
```

Example:

```text
Feature Vector
      ↓
LightGBM
      ↓
Risk Classification
```

---

# 22. Planned Final Risk Engine

After individual model testing, the useful outputs can be combined.

Example:

```text
MediaPipe
    ↓
Head turned away: YES

YOLO
    ↓
Possible phone: YES

DeepFace
    ↓
Identity matched: YES

LSTM
    ↓
Suspicious sequence: YES

LightGBM
    ↓
High-risk probability
```

Then:

```text
             Risk Engine
                  │
       ┌──────────┼──────────┐
       ▼          ▼          ▼
      LOW       MEDIUM       HIGH
       │          │           │
    Log only    Warning    Proctor Alert
```

---

# 23. Live Examination — Planned Architecture

The final system is expected to process a live webcam stream.

```text
Student Browser
       │
       │ Webcam
       ▼
Live Video Stream
       │
       ▼
Python AI Service
       │
       ├── MediaPipe
       ├── YOLO
       ├── DeepFace
       └── Behavior Models
                │
                ▼
          Risk Engine
                │
                ▼
          Event Generation
                │
                ▼
          Laravel Backend
                │
                ▼
        Proctor Dashboard
```

The models will not necessarily process every camera frame.

A practical implementation can sample frames at an appropriate rate and use temporal persistence to avoid generating alerts from a single incorrect detection.

---

# 24. Planned Alert System

The final system should use multiple levels rather than treating every detection as cheating.

### Normal

```text
Face present
One person
Normal movement
No suspicious object
```

Action:

```text
No alert
```

### Suspicious

```text
Face temporarily missing
Repeated gaze-away
Possible unauthorized object
Unusual hand movement
```

Action:

```text
Record event
Capture evidence
Increase risk score
```

### High Risk

Example:

```text
Possible phone
+
Hand reaches toward phone
+
Gaze directed toward phone
+
Behavior persists
```

Action:

```text
Generate high-priority proctor alert
Store evidence
Notify human reviewer
```

---

# 25. Important Research Principle

The system should not assume:

```text
One AI prediction = cheating
```

Instead:

```text
Model Prediction
       ↓
Evidence
       ↓
Temporal Validation
       ↓
Cross-model Correlation
       ↓
Risk Score
       ↓
Human Review
```

This approach should reduce false positives and make the proctoring system more explainable.

---

# 26. Current Overall Status

## Successfully Completed

### Environment

* [x] Homebrew configured
* [x] Miniforge/Conda configured
* [x] Python 3.11 environment created for MediaPipe
* [x] Python 3.11 environment created for YOLO
* [x] Separate model environments established

### MediaPipe

* [x] Installed successfully
* [x] Imported successfully
* [x] Version verified: `1.0.0`
* [x] Evaluation project created
* [x] Webcam testing attempted
* [x] Model asset requirement identified

### YOLO

* [x] Ultralytics installed
* [x] Version verified: `8.4.117`
* [x] Image inference successful
* [x] Multiple real images tested
* [x] Output images generated
* [x] Video inference successful
* [x] 546 video frames processed
* [x] Output video generated
* [x] False detection behavior identified

---

# 27. Current Findings

### MediaPipe

```text
Installation:        SUCCESS
Import:              SUCCESS
Basic environment:   SUCCESS
Full evaluation:     IN PROGRESS
```

### YOLO

```text
Installation:        SUCCESS
Image inference:     SUCCESS
Video inference:     SUCCESS
Real exam image:     TESTED
Real exam video:     TESTED
False positives:     OBSERVED
Evaluation:          IN PROGRESS
```

---

# 28. Current Decision

At this stage:

```text
DO NOT remove YOLO yet.
DO NOT permanently select YOLO yet.
```

The correct conclusion is:

> The pretrained YOLO model successfully performs real-time/image/video object detection, but its general-purpose object classes produce false positives for examination-specific scenarios. Further evaluation is required to determine whether it should be used directly, fine-tuned for exam-specific objects, or replaced.

The same evaluation methodology will be applied to every other AI model.

---

# 29. Next Step

The immediate next model to evaluate is:

```text
DeepFace
```

After that:

```text
DeepFace
   ↓
LSTM
   ↓
LightGBM
   ↓
PyTorch
   ↓
Compare all models
   ↓
Select necessary models
   ↓
Design final AI proctoring architecture
   ↓
Integrate with Laravel
   ↓
Build live examination system
```

---

# 30. Project Philosophy

This project is being developed using a **model-first evaluation approach**.

Instead of assuming that every model used in an existing research paper is necessary:

```text
Research Paper
      ↓
Identify Models
      ↓
Install Independently
      ↓
Test Independently
      ↓
Measure Accuracy
      ↓
Measure Performance
      ↓
Analyze False Positives
      ↓
Evaluate Practical Value
      ↓
Select Required Models
      ↓
Build Final System
```

This allows the final university project to have a technically justified architecture rather than simply reproducing an existing implementation.
