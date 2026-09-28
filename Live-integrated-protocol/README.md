# AI Proctoring Evaluation — Phase 3.1 Live Integrated Proctoring

## Overview

This phase extends the previously completed **Phase 1 detection pipeline** and **Phase 2 event-analysis pipeline** into a single live proctoring system.

The main objective of this phase was to move from testing individual components independently to running them together in real time through a webcam.

The current system can:

* Detect whether a face is present
* Detect multiple faces
* Estimate head pose
* Detect sustained looking direction
* Apply temporal smoothing
* Apply decision rules
* Calculate a live session risk score
* Generate a session event log
* Display the current risk state in real time

This is still an **evaluation/prototype system**. The thresholds and scoring rules are engineering rules for testing and are not scientifically validated cheating-detection thresholds.

---

# 1. Previous Phase Status

Before Phase 3.1, the following components had already been implemented and tested:

### Phase 1 — Detection

* MediaPipe face detection
* Face presence detection
* Multiple-face detection
* MediaPipe Face Landmarker
* Head-pose estimation
* Temporal head-pose detection
* Unified proctoring detector
* Proctoring event logger
* DeepFace/FaceNet512 identity verification test

### Phase 2 — Event Analysis

* Proctoring decision engine
* Event scoring
* Session risk calculation
* Event aggregation
* Severity hierarchy
* Repeated-event escalation
* Session summary generation

The Phase 3.1 work integrates the core real-time detection and decision logic.

---

# 2. Phase 3.1 Objective

The goal of Phase 3.1 was to create a single live controller instead of manually running individual detection scripts.

Previously:

```text
Face Detection
      ↓
Head Pose
      ↓
Event Logger
      ↓
Decision Engine
      ↓
Aggregation
      ↓
Escalation
      ↓
Session Summary
```

Phase 3.1 moves the first part into a single live pipeline:

```text
Webcam
   ↓
MediaPipe Face Detection
   ↓
Face Count
   ↓
Head Pose
   ↓
Temporal Smoothing
   ↓
Sustained Event Detection
   ↓
Decision Engine
   ↓
Live Risk Score
   ↓
Session Event JSON
```

---

# 3. New File

The main new file created in this phase is:

```text
tests/live_proctoring_system.py
```

Its purpose is to operate the tested proctoring components together in real time.

---

# 4. Current Project Structure

The relevant project structure is now:

```text
hybrid-proctoring/
│
├── tests/
│   ├── models/
│   │   ├── blaze_face_short_range.tflite
│   │   └── face_landmarker.task
│   │
│   ├── face_presence_tracker.py
│   ├── head_pose_live.py
│   ├── temporal_head_pose.py
│   ├── unified_proctoring.py
│   ├── proctoring_event_logger.py
│   │
│   ├── proctoring_decision_engine.py
│   ├── proctoring_event_aggregator.py
│   ├── proctoring_escalation.py
│   ├── proctoring_session_summary.py
│   │
│   ├── live_proctoring_system.py
│   │
│   ├── live_hybrid.py
│   ├── live_hybrid_verify.py
│   ├── mediapipe_deepface.py
│   └── webcam_deepface_test.py
│
├── test_data/
│   ├── database/
│   │   ├── student_1/
│   │   ├── student_2/
│   │   └── student_3/
│   │
│   └── queries/
│
└── results/
    ├── proctoring_events.json
    ├── proctoring_decisions.json
    ├── proctoring_aggregation.json
    ├── proctoring_escalation.json
    ├── proctoring_session_summary.json
    │
    └── live_proctoring_events.json
```

---

# 5. Live Detection Pipeline

The live controller uses the existing MediaPipe models.

### Face Detection

Model:

```text
tests/models/blaze_face_short_range.tflite
```

The face detector determines the number of visible faces.

The current logic is:

```text
0 faces
    ↓
NO_FACE

1 face
    ↓
Head Pose Analysis

2+ faces
    ↓
MULTIPLE_FACES
```

---

# 6. Head Pose Detection

For a single detected face, the system uses:

```text
tests/models/face_landmarker.task
```

The facial transformation matrix is used to estimate:

* Yaw
* Pitch

Current threshold:

```text
15 degrees
```

Classification:

```text
Yaw < -15°
    → LOOKING LEFT

Yaw > +15°
    → LOOKING RIGHT

Pitch < -15°
    → LOOKING DOWN

Pitch > +15°
    → LOOKING UP

Otherwise
    → FORWARD
```

---

# 7. Temporal Smoothing

Raw head-pose measurements can fluctuate between frames.

The live system therefore maintains:

```text
SMOOTHING_FRAMES = 7
```

Yaw and pitch values are stored in rolling queues.

The average values are used for classification instead of relying on a single frame.

This reduces short-term noise and makes the event detection more stable.

---

# 8. Sustained Event Detection

The system does not immediately consider every detected state to be an event.

The initial detection threshold remains:

```text
SUSTAINED_DURATION = 1.5 seconds
```

However, this is different fro
