# Hybrid AI Proctoring System

A research prototype for an AI-powered examination proctoring system combining real-time face detection, head-pose analysis, temporal event detection, rule-based decision scoring, event aggregation, repeated-event escalation, and session-level risk analysis.

This repository is being developed as part of the university project:

> **Next-Generation AI-Powered Smart Examination Platform with Blockchain Certificate Verification and Digital Twin Analytics**

---

## 1. Project Objective

The objective of this prototype is to develop and evaluate the AI proctoring components independently before integrating them into the complete examination platform.

The current prototype focuses on:

* Face presence detection
* Multiple-face detection
* Head-pose detection
* Temporal head-pose analysis
* Unified proctoring event detection
* Proctoring event logging
* Rule-based event decision making
* Event aggregation
* Repeated-event detection
* Repeated-event escalation
* Session-level risk analysis
* Session summary generation

The system is intentionally being developed incrementally so that each component can be tested independently before live integration.

---

# 2. Current Architecture

The current offline analysis pipeline is:

```text
Webcam
   │
   ▼
MediaPipe Face Detection
   │
   ├── 0 Faces ───────────────► NO_FACE
   │
   ├── 2+ Faces ──────────────► MULTIPLE_FACES
   │
   └── 1 Face
          │
          ▼
   MediaPipe Face Landmarker
          │
          ▼
      Head Pose
          │
          ▼
   Temporal Event Detection
          │
          ▼
   Unified Proctoring Detector
          │
          ▼
   Proctoring Event Logger
          │
          ▼
   proctoring_events.json
          │
          ▼
   Decision Engine
          │
          ▼
   proctoring_decisions.json
          │
          ▼
   Event Aggregator
          │
          ▼
   proctoring_aggregation.json
          │
          ▼
   Repeated-Event Escalation
          │
          ▼
   proctoring_escalation.json
          │
          ▼
   Session Summary
          │
          ▼
   proctoring_session_summary.json
```

---

# 3. Environment

The current development environment is:

```text
Operating System: macOS
Architecture: Apple Silicon / M4
Python: 3.11.15
Environment: mediapipe-deepface-test
```

Installed/verified packages:

```text
MediaPipe: 1.0.0
TensorFlow: 2.21.0
Keras: 3.15.1
DeepFace: 0.0.100
RetinaFace: 0.0.18
tf-keras: 2.21.0
```

Environment activation:

```bash
conda activate mediapipe-deepface-test
```

Project directory:

```bash
cd ~/auto-oep-test/hybrid-proctoring
```

---

# 4. Project Structure

Current important structure:

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
│   ├── proctoring_decision_engine.py
│   ├── proctoring_event_aggregator.py
│   ├── proctoring_escalation.py
│   └── proctoring_session_summary.py
│
├── test_data/
│   ├── database/
│   │   ├── student_1/
│   │   ├── student_2/
│   │   └── student_3/
│   │
│   └── queries/
│       └── student_1_2.jpeg
│
├── results/
│   ├── debug_frame.jpg
│   ├── proctoring_events.json
│   ├── proctoring_decisions.json
│   ├── proctoring_aggregation.json
│   ├── proctoring_escalation.json
│   └── proctoring_session_summary.json
│
└── README.md
```

---

# 5. Phase 1 — Real-Time Detection

## 5.1 MediaPipe Face Detection

The original MediaPipe `mp.solutions` API was not available in the installed MediaPipe version.

The project was therefore migrated to the MediaPipe Tasks API.

Model:

```text
blaze_face_short_range.tflite
```

Location:

```text
tests/models/blaze_face_short_range.tflite
```

The detector was successfully tested with:

* No face
* One face
* Multiple faces

Observed performance during testing was approximately:

```text
23–30 FPS
```

depending on the test conditions.

---

# 6. Face Presence Tracker

File:

```text
tests/face_presence_tracker.py
```

Run:

```bash
python tests/face_presence_tracker.py
```

The tracker identifies:

```text
NO_FACE
NORMAL
MULTIPLE_FACES
```

Example:

```text
Event: NO FACE
Event: NORMAL
Event: MULTIPLE FACES
Event: NORMAL
```

Status:

```text
PASSED
```

---

# 7. Face Identity Verification

DeepFace with FaceNet512 was tested for identity verification.

Example:

```text
DeepFace.verify(
    image1,
    image2,
    model_name="Facenet512",
    detector_backend="retinaface",
    enforce_detection=False
)
```

A same-person comparison successfully returned:

```text
verified: True
distance: 0.0
threshold: 0.3
confidence: 100%
```

A saved live frame was also successfully verified.

Example observed result:

```text
verified: True
distance: 0.197025
threshold: 0.3
confidence: 74.03%
```

Different-person comparisons produced distances above the verification threshold in tested cases.

---

## 7.1 DeepFace / RetinaFace Compatibility

TensorFlow/Keras compatibility was encountered during RetinaFace/DeepFace testing.

The working environment requires legacy Keras handling:

```bash
TF_USE_LEGACY_KERAS=1
```

Example:

```bash
TF_USE_LEGACY_KERAS=1 python -c "from deepface import DeepFace; ..."
```

This resolved the KerasTensor-related compatibility problem encountered during live verification.

---

## 7.2 Identity Verification Architecture Decision

FaceNet512/DeepFace verification is significantly slower than MediaPipe real-time detection.

Therefore, it should **not** be executed on every webcam frame.

The current architectural direction is:

```text
MediaPipe
    ↓
Real-time tracking
    ↓
Periodic / triggered identity verification
    ↓
DeepFace / FaceNet512
```

This keeps the real-time monitoring pipeline lightweight.

---

# 8. Head Pose Detection

File:

```text
tests/head_pose_live.py
```

Model:

```text
tests/models/face_landmarker.task
```

The system calculates head orientation and classifies:

```text
FORWARD
LOOKING_LEFT
LOOKING_RIGHT
LOOKING_UP
LOOKING_DOWN
NO_FACE
```

Example tested values:

```text
FORWARD
Yaw: -4.9
Pitch: 2.1

LOOKING_UP
Yaw: 5.2
Pitch: 15.4

LOOKING_DOWN
Yaw: 7.6
Pitch: -15.0

LOOKING_RIGHT
Yaw: 15.1
Pitch: -26.2

LOOKING_LEFT
Yaw: -15.2
Pitch: 32.8
```

Status:

```text
PASSED
```

---

# 9. Temporal Head Pose

File:

```text
tests/temporal_head_pose.py
```

The temporal detector prevents every short head movement from immediately becoming a proctoring violation.

It tracks how long a particular head-pose event persists.

Example concept:

```text
LOOKING_RIGHT
      │
      ├── Short movement
      │      ↓
      │    Ignore
      │
      └── Sustained movement
             ↓
          Event
```

A timing bug involving `None` event start time was identified and fixed.

The script was successfully compiled and executed.

Status:

```text
PASSED
```

Multiple-face detection is intentionally handled by the unified detector rather than the temporal head-pose component.

---

# 10. Unified Proctoring Detector

File:

```text
tests/unified_proctoring.py
```

Architecture:

```text
Webcam
   ↓
MediaPipe Face Detection
   ↓
Face Count
   │
   ├── 0 → NO_FACE
   │
   ├── 2+ → MULTIPLE_FACES
   │
   └── 1
        ↓
   Face Landmarker
        ↓
   Head Pose
        ↓
   Smoothing
        ↓
   Duration Tracking
        ↓
   Proctoring Event
```

The prototype uses temporal smoothing and sustained event duration to reduce noise.

Example event:

```text
EVENT: LOOKING_UP | Duration: 1.5s
```

Status:

```text
PASSED
```

---

# 11. Proctoring Event Logger

File:

```text
tests/proctoring_event_logger.py
```

Run:

```bash
python tests/proctoring_event_logger.py
```

Output file:

```text
results/proctoring_events.json
```

Each event contains:

```json
{
    "timestamp": "...",
    "event": "LOOKING_RIGHT",
    "duration": 1.5,
    "face_count": 1
}
```

Example events tested:

```text
LOOKING_UP
LOOKING_RIGHT
MULTIPLE_FACES
LOOKING_UP
```

The logger successfully generated structured JSON event data.

Status:

```text
PASSED
```

---

# 12. Phase 2 — Proctoring Decision Engine

File:

```text
tests/proctoring_decision_engine.py
```

Input:

```text
results/proctoring_events.json
```

Output:

```text
results/proctoring_decisions.json
```

The decision engine converts raw events into decisions and scores.

---

## 12.1 Prototype Event Scores

Current prototype scoring:

| Event             | Score |
| ----------------- | ----: |
| NORMAL            |     0 |
| LOOKING_LEFT      |     1 |
| LOOKING_RIGHT     |     1 |
| LOOKING_UP        |     1 |
| LOOKING_DOWN      |     1 |
| NO_FACE           |     3 |
| MULTIPLE_FACES    |     5 |
| IDENTITY_MISMATCH |    10 |

These are **prototype engineering rules**, not scientifically validated cheating-detection thresholds.

---

# 13. Decision Rules

### Head movement

```text
Duration < 2s
→ IGNORE

Duration >= 2s
→ WARNING
```

### NO_FACE

```text
Duration < 2s
→ IGNORE

2–5s
→ WARNING

>5s
→ VIOLATION
```

### MULTIPLE_FACES

```text
Duration < 1s
→ IGNORE

Duration >= 1s
→ HIGH
```

### IDENTITY_MISMATCH

```text
→ CRITICAL
```

---

# 14. Session Risk

The prototype calculates session risk from the total score:

```text
0
→ NORMAL

1–3
→ LOW_RISK

4–7
→ MEDIUM_RISK

8–12
→ HIGH_RISK

>12
→ CRITICAL_RISK
```

---

# 15. Decision Engine Testing

Multiple test cases were successfully completed.

### Test 1

```text
LOOKING_LEFT 1.5s
LOOKING_RIGHT 1.5s
```

Result:

```text
Both → IGNORE
Total score → 0
Session → NORMAL
```

### Test 2

```text
NO_FACE 1.5s
NO_FACE 3.0s
NO_FACE 6.0s
```

Result:

```text
IGNORE
WARNING
VIOLATION

Total score: 6
Session: MEDIUM_RISK
```

### Test 3

```text
MULTIPLE_FACES 0.5s
MULTIPLE_FACES 1.5s
MULTIPLE_FACES 3.0s
```

Result:

```text
IGNORE
HIGH
HIGH

Total score: 10
Session: HIGH_RISK
```

### Test 4

Mixed events were tested successfully.

### Test 5

Repeated events were tested:

```text
LOOKING_RIGHT × 3
NO_FACE × 2
MULTIPLE_FACES × 1
```

Result:

```text
Total score: 14
Session risk: CRITICAL_RISK
```

---

# 16. Event Aggregator

File:

```text
tests/proctoring_event_aggregator.py
```

Input:

```text
results/proctoring_decisions.json
```

Output:

```text
results/proctoring_aggregation.json
```

The aggregator calculates:

* Occurrences
* Total duration
* Average duration
* Maximum duration
* Total score
* Highest severity

---

# 17. Severity Hierarchy

The severity hierarchy was corrected during testing.

Current hierarchy:

```text
NORMAL / IGNORE
       ↓
WARNING
       ↓
HIGH
       ↓
VIOLATION
       ↓
CRITICAL
```

Numerical levels:

```text
NORMAL / IGNORE = 0
WARNING          = 1
HIGH             = 2
VIOLATION        = 3
CRITICAL         = 4
```

This ensures that:

```text
VIOLATION > HIGH
```

For example, when `NO_FACE` reaches `VIOLATION` and `MULTIPLE_FACES` is `HIGH`, the highest severity is correctly identified as:

```text
NO_FACE
```

---

# 18. Current Aggregation Result

Current validated test data:

```text
Total events: 6
Total score: 14
Session risk: CRITICAL_RISK
Unique event types: 3
Highest severity event: NO_FACE
Highest severity level: 3
```

### LOOKING_RIGHT

```text
Occurrences: 3
Total duration: 9.5s
Average duration: 3.17s
Maximum duration: 4.0s
Total score: 3
Highest severity: WARNING
```

### NO_FACE

```text
Occurrences: 2
Total duration: 8.5s
Average duration: 4.25s
Maximum duration: 6.0s
Total score: 6
Highest severity: VIOLATION
```

### MULTIPLE_FACES

```text
Occurrences: 1
Total duration: 1.5s
Average duration: 1.5s
Maximum duration: 1.5s
Total score: 5
Highest severity: HIGH
```

---

# 19. Repeated-Event Escalation

File:

```text
tests/proctoring_escalation.py
```

Input:

```text
results/proctoring_aggregation.json
```

Output:

```text
results/proctoring_escalation.json
```

The purpose of this layer is to detect repeated behavior rather than relying only on individual event scores.

---

## 19.1 Prototype Escalation Rules

For head-pose events:

```text
1 occurrence
→ NORMAL

3 occurrences
→ REPEATED_WARNING

5+ occurrences
→ ESCALATED
```

For `NO_FACE`:

```text
2 occurrences
→ REPEATED_WARNING

3+ occurrences
→ ESCALATED
```

For `MULTIPLE_FACES`:

```text
2 occurrences
→ REPEATED_WARNING

3+ occurrences
→ ESCALATED
```

These thresholds are prototype rules for system development and evaluation.

---

# 20. Escalation Testing

The following test was performed:

```text
LOOKING_RIGHT × 5
NO_FACE × 3
MULTIPLE_FACES × 3
```

Expected and observed:

```text
LOOKING_RIGHT → ESCALATED
NO_FACE → ESCALATED
MULTIPLE_FACES → ESCALATED
```

Result:

```text
Total events: 11
Total score: 31
Session risk: CRITICAL_RISK
Escalated event count: 3
```

After testing, the original aggregation data was restored.

---

# 21. Current Escalation Result

For the normal 6-event test dataset:

```text
LOOKING_RIGHT × 3
→ REPEATED_WARNING

NO_FACE × 2
→ REPEATED_WARNING

MULTIPLE_FACES × 1
→ NORMAL
```

Therefore:

```text
Escalated event count: 0
Escalated events: []
```

---

# 22. Session Summary

File:

```text
tests/proctoring_session_summary.py
```

Input:

```text
results/proctoring_aggregation.json
results/proctoring_escalation.json
```

Output:

```text
results/proctoring_session_summary.json
```

The session summary combines the results from the previous analysis layers.

It contains:

* Session risk
* Total events
* Total score
* Unique event types
* Highest severity event
* Highest severity level
* Repeated event count
* Repeated events
* Escalated event count
* Escalated events
* Complete event summary

---

# 23. Current Final Session Summary

The latest validated result is:

```text
Session risk: CRITICAL_RISK
Total events: 6
Total score: 14
Unique event types: 3
Highest severity event: NO_FACE
Highest severity level: 3
Repeated event count: 2
Escalated event count: 0
```

Repeated events:

```text
LOOKING_RIGHT
Occurrences: 3
Severity: WARNING

NO_FACE
Occurrences: 2
Severity: VIOLATION
```

Non-repeated event:

```text
MULTIPLE_FACES
Occurrences: 1
Severity: HIGH
```

---

# 24. Current Completed Components

| Component                          | Status |
| ---------------------------------- | ------ |
| MediaPipe Face Detection           | PASSED |
| Face Presence Detection            | PASSED |
| Multiple Face Detection            | PASSED |
| DeepFace / FaceNet512 Verification | PASSED |
| Head Pose Detection                | PASSED |
| Temporal Head Pose                 | PASSED |
| Unified Proctoring Detector        | PASSED |
| Event Logger                       | PASSED |
| Decision Engine                    | PASSED |
| Event Aggregator                   | PASSED |
| Severity Hierarchy                 | PASSED |
| Repeated-Event Escalation          | PASSED |
| Escalation Threshold Testing       | PASSED |
| Session Summary                    | PASSED |

---

# 25. Important Design Decisions

## Real-Time Detection

MediaPipe is used for lightweight real-time detection.

```text
MediaPipe
→ Real-time monitoring
```

## Identity Verification

DeepFace / FaceNet512 is not intended to run on every webcam frame.

```text
MediaPipe
→ Trigger / periodic verification
→ DeepFace
```

## Multiple Faces

Multiple-face detection is handled at the unified detector level.

```text
Face count >= 2
→ MULTIPLE_FACES
```

## Temporal Filtering

Short events should not automatically become serious violations.

Duration and repeated occurrence are considered separately.

```text
Duration
+
Occurrence count
+
Severity
+
Session score
```

are used to produce the current prototype's session analysis.

---

# 26. Current Limitations

This is still a research prototype.

The following areas are not yet fully implemented:

* Live end-to-end decision processing
* Real-time identity verification integration
* Gaze estimation
* Eye-state analysis
* Mouth movement detection
* Hand proximity detection
* Object/prohibited-item detection
* YOLO-based object detection
* LSTM temporal behavior model
* Screen monitoring
* Browser lockdown
* Audio analysis
* Advanced anti-spoofing
* Production database integration
* Web-based examination interface
* Blockchain certificate integration
* Digital twin analytics

The current rule-based thresholds are also not scientifically validated.

They are being used to develop and test the architecture.

---

# 27. Current Development Philosophy

The system is being developed incrementally.

Instead of immediately combining every AI model, each component is first:

```text
Implemented
    ↓
Compiled
    ↓
Executed
    ↓
Tested
    ↓
Validated
    ↓
Integrated
```

This makes it easier to identify:

* Model compatibility issues
* Performance bottlenecks
* False detections
* Incorrect thresholds
* Integration problems
* Data-flow problems

---

# 28. Next Development Phase

The next major phase is to move from the currently validated offline JSON-processing pipeline to a live integrated proctoring system.

Target architecture:

```text
Webcam
   ↓
MediaPipe Face Detection
   ↓
Face Count
   ↓
Head Pose
   ↓
Temporal Analysis
   ↓
Event Detection
   ↓
Event Decision
   ↓
Session Aggregation
   ↓
Repeated Event Analysis
   ↓
Session Summary
```

The eventual goal is to run the system through a single command such as:

```bash
python tests/live_proctoring_system.py
```

rather than manually processing intermediate JSON files.

---

# 29. Development Status

Current status:

```text
Phase 1 — Individual AI Detection
        COMPLETED

Phase 2 — Proctoring Event Analysis
        COMPLETED

Phase 3 — Live Integrated Proctoring Pipeline
        NEXT

Phase 4 — Additional AI Models
        PLANNED

Phase 5 — Examination Platform Integration
        PLANNED
```

---

# 30. Prototype Disclaimer

This repository represents an experimental university research prototype.

The current scoring, severity, duration, and escalation thresholds are engineering rules created for testing the system architecture.

They should not be interpreted as scientifically validated indicators of academic misconduct.

Further validation using representative examination-session datasets will be required before these rules can be considered for real-world deployment.
