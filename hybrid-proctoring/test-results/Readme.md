# AI Proctoring Evaluation — Phase 2

## Proctoring Decision Engine

**Project:** Hybrid AI Proctoring Evaluation
**Repository:** `hybrid-proctoring`
**Phase:** Phase 2 — Proctoring Decision Engine
**Status:** Completed and Tested
**Platform:** macOS — Apple Silicon
**Environment:** `mediapipe-deepface-test`

---

## 1. Phase Objective

The purpose of Phase 2 is to convert raw proctoring events into meaningful decisions.

The previous phase successfully detected and logged events such as:

* `NO_FACE`
* `MULTIPLE_FACES`
* `LOOKING_LEFT`
* `LOOKING_RIGHT`
* `LOOKING_UP`
* `LOOKING_DOWN`

However, event logging alone does not determine whether an event should be ignored, warned, or treated as a serious violation.

This phase introduces a **Proctoring Decision Engine** that evaluates each event based on:

* Event type
* Event duration
* Severity
* Event score
* Session-level accumulated risk

---

## 2. Current Architecture

```text
Webcam
   │
   ▼
MediaPipe Face Detection
   │
   ▼
Face Count
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
   Proctoring Event Logger
          │
          ▼
   proctoring_events.json
          │
          ▼
   Proctoring Decision Engine
          │
          ├── Event Decision
          ├── Event Score
          └── Session Risk
          │
          ▼
   proctoring_decisions.json
```

---

## 3. Files Used

```text
hybrid-proctoring/
│
├── tests/
│   ├── face_presence_tracker.py
│   ├── head_pose_live.py
│   ├── temporal_head_pose.py
│   ├── unified_proctoring.py
│   ├── proctoring_event_logger.py
│   └── proctoring_decision_engine.py
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
    └── proctoring_decisions.json
```

---

# 4. Decision Engine

File:

```text
tests/proctoring_decision_engine.py
```

The engine reads:

```text
results/proctoring_events.json
```

and generates:

```text
results/proctoring_decisions.json
```

The input event format is:

```json
{
    "timestamp": "2026-09-15T09:20:10",
    "event": "NO_FACE",
    "duration": 3.0,
    "face_count": 0
}
```

The decision engine adds:

```json
{
    "decision": "WARNING",
    "score": 3
}
```

---

# 5. Event Scores

The current prototype uses the following scores:

| Event               | Score |
| ------------------- | ----: |
| `NORMAL`            |     0 |
| `LOOKING_LEFT`      |     1 |
| `LOOKING_RIGHT`     |     1 |
| `LOOKING_UP`        |     1 |
| `LOOKING_DOWN`      |     1 |
| `NO_FACE`           |     3 |
| `MULTIPLE_FACES`    |     5 |
| `IDENTITY_MISMATCH` |    10 |

These values are **prototype scoring rules** for evaluation.

They are not scientifically validated cheating-detection scores.

---

# 6. Decision Rules

## 6.1 Normal

```text
NORMAL → NORMAL
Score = 0
```

No suspicious activity is detected.

---

## 6.2 Looking Away

For:

```text
LOOKING_LEFT
LOOKING_RIGHT
LOOKING_UP
LOOKING_DOWN
```

the current rule is:

```text
Duration < 2 seconds
        ↓
IGNORE

Duration >= 2 seconds
        ↓
WARNING
```

Example:

```text
LOOKING_LEFT
Duration: 1.5s
Decision: IGNORE
Score: 0
```

Example:

```text
LOOKING_RIGHT
Duration: 2.5s
Decision: WARNING
Score: 1
```

---

# 7. No Face Detection

The current rules are:

```text
Duration < 2 seconds
        ↓
IGNORE
```

```text
2–5 seconds
        ↓
WARNING
```

```text
> 5 seconds
        ↓
VIOLATION
```

Example:

```text
NO_FACE
Duration: 1.5s
Decision: IGNORE
Score: 0
```

```text
NO_FACE
Duration: 3.0s
Decision: WARNING
Score: 3
```

```text
NO_FACE
Duration: 6.0s
Decision: VIOLATION
Score: 3
```

---

# 8. Multiple Faces

The current prototype uses:

```text
Duration < 1 second
        ↓
IGNORE
```

```text
Duration >= 1 second
        ↓
HIGH
```

Example:

```text
MULTIPLE_FACES
Duration: 0.5s
Decision: IGNORE
Score: 0
```

```text
MULTIPLE_FACES
Duration: 1.5s
Decision: HIGH
Score: 5
```

```text
MULTIPLE_FACES
Duration: 3.0s
Decision: HIGH
Score: 5
```

---

# 9. Identity Mismatch

The decision engine reserves:

```text
IDENTITY_MISMATCH
```

as a critical event.

Current prototype rule:

```text
IDENTITY_MISMATCH
        ↓
CRITICAL
Score = 10
```

This event will later be connected to the DeepFace/FaceNet512 identity verification component.

---

# 10. Session Risk Calculation

After evaluating all events, the engine calculates the total session score.

Current prototype thresholds:

| Total Score | Session Risk    |
| ----------: | --------------- |
|         `0` | `NORMAL`        |
|       `1–3` | `LOW_RISK`      |
|       `4–7` | `MEDIUM_RISK`   |
|      `8–12` | `HIGH_RISK`     |
|       `>12` | `CRITICAL_RISK` |

The current system therefore produces both:

1. Individual event decisions
2. Overall session risk

---

# 11. Test 1 — Looking Away

Input:

```text
LOOKING_LEFT
Duration: 1.5s

LOOKING_RIGHT
Duration: 1.5s
```

Result:

```text
LOOKING_LEFT → IGNORE → 0
LOOKING_RIGHT → IGNORE → 0
```

Session:

```text
Total events: 2
Total score: 0
Session risk: NORMAL
```

### Result

**PASSED**

Short head movements are currently ignored.

---

# 12. Test 2 — No Face

Three different durations were tested:

```text
NO_FACE → 1.5s
NO_FACE → 3.0s
NO_FACE → 6.0s
```

Result:

```text
1.5s → IGNORE → 0
3.0s → WARNING → 3
6.0s → VIOLATION → 3
```

Session result:

```text
Total events: 3
Total score: 6
Session risk: MEDIUM_RISK
```

### Result

**PASSED**

The duration-based `NO_FACE` rules work correctly.

---

# 13. Test 3 — Multiple Faces

Test data:

```text
MULTIPLE_FACES → 0.5s
MULTIPLE_FACES → 1.5s
MULTIPLE_FACES → 3.0s
```

Result:

```text
0.5s → IGNORE → 0
1.5s → HIGH → 5
3.0s → HIGH → 5
```

Session result:

```text
Total events: 3
Total score: 10
Session risk: HIGH_RISK
```

### Result

**PASSED**

The multiple-face duration logic works correctly.

---

# 14. Test 4 — Mixed Proctoring Session

A realistic combination of different events was tested:

```text
LOOKING_LEFT      → 1.5s
LOOKING_RIGHT     → 2.5s
NO_FACE           → 3.0s
MULTIPLE_FACES    → 1.5s
LOOKING_UP        → 2.2s
```

Results:

```text
LOOKING_LEFT   → IGNORE    → 0
LOOKING_RIGHT  → WARNING   → 1
NO_FACE        → WARNING   → 3
MULTIPLE_FACES → HIGH      → 5
LOOKING_UP     → WARNING   → 1
```

Total:

```text
Total events: 5
Total score: 10
Session risk: HIGH_RISK
```

### Result

**PASSED**

The engine successfully combines different event types into a single session risk assessment.

---

# 15. Output Format

The decision engine generates:

```text
results/proctoring_decisions.json
```

Example:

```json
{
    "total_events": 5,
    "total_score": 10,
    "session_risk": "HIGH_RISK",
    "decisions": [
        {
            "timestamp": "2026-09-15T09:20:00",
            "event": "LOOKING_LEFT",
            "duration": 1.5,
            "face_count": 1,
            "decision": "IGNORE",
            "score": 0
        }
    ]
}
```

The output therefore contains both event-level and session-level information.

---

# 16. Testing Commands

From the project root:

```bash
cd ~/auto-oep-test/hybrid-proctoring
```

Activate the environment:

```bash
conda activate mediapipe-deepface-test
```

Compile:

```bash
python -m py_compile tests/proctoring_decision_engine.py
```

Run:

```bash
python tests/proctoring_decision_engine.py
```

View the result:

```bash
cat results/proctoring_decisions.json
```

---

# 17. Current Phase Status

### Detection

* [x] MediaPipe face detection
* [x] Face presence detection
* [x] Multiple-face detection
* [x] MediaPipe Face Landmarker
* [x] Head-pose detection
* [x] Temporal head-pose monitoring

### Identity

* [x] DeepFace/FaceNet512 verification
* [x] RetinaFace integration
* [x] Correct-person verification
* [x] Different-person mismatch testing
* [x] TensorFlow/Keras compatibility resolved using legacy Keras mode

### Event System

* [x] Unified proctoring detector
* [x] Event duration tracking
* [x] Proctoring event logger
* [x] JSON event storage

### Decision Engine

* [x] Event scoring
* [x] Duration-based decisions
* [x] Event severity
* [x] Session score
* [x] Session risk classification
* [x] Mixed-session testing

---

# 18. Important Performance Consideration

DeepFace/FaceNet512 identity verification is computationally expensive compared with MediaPipe detection.

Therefore, identity verification should **not** be executed on every webcam frame.

The current architectural direction is:

```text
Real-time webcam
       ↓
MediaPipe
       ↓
Fast event detection
       ↓
Periodic / triggered identity verification
       ↓
DeepFace / FaceNet512
```

This prevents the expensive identity model from becoming the bottleneck of the real-time proctoring pipeline.

---

# 19. Current Limitations

The current decision engine is a prototype.

It currently does not consider:

* Event frequency
* Repeated suspicious behavior
* Total suspicious duration
* Time intervals between events
* Multiple occurrences of the same event
* Context between different events
* Identity verification history
* Confidence values from models
* False-positive rates
* Student-specific calibration
* Examination-specific thresholds

The current score system should therefore be treated as an **experimental decision layer**, not a final cheating classifier.

---

# 20. Next Step — Event Aggregation

The next planned improvement is:

## Phase 2.2 — Event Aggregation

Instead of evaluating each event independently, the system will aggregate events across the examination session.

For example:

```text
LOOKING_RIGHT

Occurrences: 4
Total Duration: 11.2s
Average Duration: 2.8s
Maximum Duration: 4.1s
Severity: WARNING
```

Another example:

```text
NO_FACE

Occurrences: 2
Total Duration: 8.4s
Maximum Duration: 5.7s
Severity: VIOLATION
```

The system will eventually produce a session summary similar to:

```text
=== Proctoring Session Summary ===

Total Events: 12

LOOKING_LEFT:
    Occurrences: 3
    Total Duration: 7.4s

LOOKING_RIGHT:
    Occurrences: 4
    Total Duration: 11.2s

NO_FACE:
    Occurrences: 2
    Total Duration: 8.4s

MULTIPLE_FACES:
    Occurrences: 1
    Total Duration: 2.1s

Highest Severity:
    HIGH

Total Score:
    18

Session Risk:
    CRITICAL_RISK
```

This will make the decision layer significantly more useful for the eventual AI proctoring platform.

---

# 21. Phase 2 Summary

Phase 2 successfully transformed the system from:

```text
Detect Event
     ↓
Log Event
```

into:

```text
Detect Event
     ↓
Log Event
     ↓
Evaluate Event
     ↓
Assign Severity
     ↓
Assign Score
     ↓
Calculate Session Risk
```

The decision engine has been compiled and tested successfully against:

* Short head movements
* No-face events
* Multiple-face events
* Mixed suspicious behavior

The next objective is to make the decision engine **behavior-aware through event aggregation and repeated-event analysis**.
