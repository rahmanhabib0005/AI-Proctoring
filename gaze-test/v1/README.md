# Hybrid AI Proctoring System

A real-time AI-powered examination proctoring system that combines face detection, student identity verification, head-pose estimation, gaze tracking, temporal event detection, risk scoring, and JSON-based session reporting.

The project is being developed incrementally. Individual components are tested first, then integrated into the live proctoring pipeline.

## Table of Contents

- [Project Overview](#project-overview)
- [Technology Stack](#technology-stack)
- [Development Status](#development-status)
- [Project Structure](#project-structure)
- [Phase 1: Core Detection](#phase-1-core-detection)
- [Phase 2: Event Processing Pipeline](#phase-2-event-processing-pipeline)
- [Phase 3: Live Proctoring Integration](#phase-3-live-proctoring-integration)
- [Installation and Environment](#installation-and-environment)
- [Running the Tests](#running-the-tests)
- [Risk Scoring](#risk-scoring)
- [Output Files](#output-files)
- [Known Issues and Limitations](#known-issues-and-limitations)
- [Next Steps](#next-steps)

## Project Overview

The system monitors a webcam stream and identifies potentially suspicious examination behavior.

### Core capabilities

- Real-time face detection and face-count monitoring.
- Face-presence tracking.
- Student identity verification using DeepFace and FaceNet512.
- Head-pose estimation using facial transformation matrices.
- Iris-based gaze direction estimation.
- Temporal smoothing and sustained-event detection.
- Event duration measurement.
- Rule-based decisions and risk scoring.
- Event aggregation and repeated-event escalation.
- JSON-based event logs and session summaries.

**Important:** These are proctoring signals, not definitive proof of misconduct. Identity and gaze predictions can fluctuate and should be evaluated in context.

## Technology Stack

| Technology | Purpose |
|---|---|
| Python 3.11.15 | Main development environment |
| OpenCV | Webcam capture and image processing |
| MediaPipe Tasks | Face detection, facial landmarks, and transformation matrices |
| DeepFace | Face verification |
| FaceNet512 | Facial identity embeddings |
| RetinaFace | Face detection backend for identity verification |
| TensorFlow / Keras | Deep-learning runtime |
| NumPy | Numerical calculations |
| JSON | Event logs and session reports |

### Environment notes

Development and testing have been performed on macOS running on Apple Silicon (M4).

The active environment used for recent tests is:

`mediapipe-deepface-test`

For DeepFace/RetinaFace compatibility with the installed TensorFlow/Keras environment, the identity-verification script sets:

`TF_USE_LEGACY_KERAS=1`

MediaPipe Tasks API is used because the installed MediaPipe version does not expose the older `mp.solutions` API used by legacy examples.

## Development Status

| Phase | Component | Status |
|---|---|---|
| 1.1 | MediaPipe face detection | Tested |
| 1.2 | DeepFace / FaceNet512 verification | Tested |
| 1.3 | Face-presence tracking | Tested |
| 1.4 | Head-pose estimation | Tested |
| 1.5 | Temporal head-pose detection | Tested |
| 1.6 | Unified proctoring detector | Tested |
| 1.7 | Proctoring event logger | Tested |
| 2.1 | Decision engine | Tested |
| 2.2 | Event aggregator | Tested |
| 2.3 | Repeated-event escalation | Tested |
| 2.4 | Session summary generator | Tested |
| 3.1 | Live proctoring system | Tested |
| 3.2 | Live session-processing pipeline | Tested |
| 3.3 | Accurate event duration | Tested |
| 3.4 | Live identity verification | Tested |
| 3.5 | Gaze tracking and temporal gaze detection | Tested |
| 3.6 | Live gaze + head-pose + face-presence integration | Ran successfully; cleanup remains |

The status above reflects the recorded development tests. It does not mean the system has been validated for production examination use.

## Project Structure

The following lists the main files used in the completed development phases. Other files may exist in the repository.

```text
hybrid-proctoring/
├── tests/
│   ├── models/
│   │   ├── blaze_face_short_range.tflite
│   │   └── face_landmarker.task
│   ├── face_presence_tracker.py
│   ├── head_pose_live.py
│   ├── temporal_head_pose.py
│   ├── unified_proctoring.py
│   ├── proctoring_event_logger.py
│   ├── proctoring_decision_engine.py
│   ├── proctoring_event_aggregator.py
│   ├── proctoring_escalation.py
│   ├── proctoring_session_summary.py
│   ├── live_proctoring_system.py
│   ├── live_session_pipeline.py
│   ├── live_hybrid_verify.py
│   ├── gaze_tracking.py
│   ├── temporal_gaze.py
│   └── live_gaze_proctoring.py
├── test_data/
│   ├── database/
│   │   ├── student_1/
│   │   ├── student_2/
│   │   └── student_3/
│   └── queries/
├── results/
│   ├── proctoring_events.json
│   ├── proctoring_decisions.json
│   ├── proctoring_aggregation.json
│   ├── proctoring_escalation.json
│   ├── proctoring_session_summary.json
│   ├── live_proctoring_events.json
│   ├── live_proctoring_decisions.json
│   ├── live_proctoring_aggregation.json
│   ├── live_proctoring_escalation.json
│   ├── live_proctoring_session_summary.json
│   └── live_gaze_proctoring_events.json
└── README.md
```

The structure is representative of the documented files; confirm the actual filenames in the repository before assuming every output or test file exists.

## Phase 1: Core Detection

### 1.1 Face detection

**File:** `tests/models/blaze_face_short_range.tflite`

MediaPipe Tasks face detection was tested with a live webcam. The detector successfully handled frames with no faces, one face, and multiple faces.

Live detection performance observed during testing was approximately 23–30 FPS, depending on the test run.

### 1.2 Student identity verification

**File:** `tests/live_hybrid_verify.py`

The identity-verification component uses DeepFace with FaceNet512 embeddings and the RetinaFace detector backend.

Reference images are stored in the student database directory.

A successful comparison between the same student's reference image and a captured frame returned a verified result. A captured frame was also successfully compared against the reference image.

The component was tested independently and then in a live webcam script.

Identity verification is more computationally expensive than ordinary face detection. It should not run on every frame; periodic or event-triggered verification is more practical.

### 1.3 Face-presence tracking

**File:** `tests/face_presence_tracker.py`

The tracker identifies the following states:

- `NO_FACE`
- `NORMAL`
- `MULTIPLE_FACES`

The standalone live test successfully detected transitions between these states.

### 1.4 Head-pose estimation

**File:** `tests/head_pose_live.py`

The head-pose detector estimates yaw and pitch from MediaPipe facial transformation matrices.

Supported states include:

- `FORWARD`
- `LOOKING_LEFT`
- `LOOKING_RIGHT`
- `LOOKING_UP`
- `LOOKING_DOWN`
- `NO_FACE`

The test successfully produced directional classifications when the user's head moved.

The initial threshold used in the integrated live scripts is approximately 15 degrees. This is a configurable heuristic, not a universal threshold for suspicious behavior.

### 1.5 Temporal head-pose detection

**File:** `tests/temporal_head_pose.py`

Temporal processing was tested to reduce unstable classifications caused by rapid changes between frames.

The test successfully ran after a timer initialization issue was corrected. Multiple-face detection remains the responsibility of the unified detector.

### 1.6 Unified proctoring detector

**File:** `tests/unified_proctoring.py`

This component combines face detection, face counting, head-pose estimation, smoothing, and sustained-state detection.

It uses the MediaPipe face detector and face landmarker models.

Tested behavior includes detection of `NO_FACE`, `MULTIPLE_FACES`, and directional head-pose states.

### 1.7 Proctoring event logger

**File:** `tests/proctoring_event_logger.py`

The event logger stores detected events in:

`results/proctoring_events.json`

Each logged event includes fields such as:

- `timestamp`
- `event`
- `duration`
- `face_count`

The standalone event logger successfully recorded directional head movement and multiple-face events.

## Phase 2: Event Processing Pipeline

Phase 2 processes recorded events separately from the live webcam. This separation makes the decision rules easier to test using repeatable input data.

### 2.1 Decision engine

**File:** `tests/proctoring_decision_engine.py`

Input:

`results/proctoring_events.json`

Output:

`results/proctoring_decisions.json`

The decision engine assigns a decision and score to each event using its type and duration.

| Event | Duration | Decision | Score |
|---|---:|---|---:|
| `NORMAL` | Any | NORMAL | 0 |
| `LOOKING_*` | Less than 2 seconds | IGNORE | 0 |
| `LOOKING_*` | 2 seconds or more | WARNING | 1 |
| `NO_FACE` | Less than 2 seconds | IGNORE | 0 |
| `NO_FACE` | 2–5 seconds | WARNING | 3 |
| `NO_FACE` | More than 5 seconds | VIOLATION | 3 |
| `MULTIPLE_FACES` | Less than 1 second | IGNORE | 0 |
| `MULTIPLE_FACES` | 1 second or more | HIGH | 5 |
| `IDENTITY_MISMATCH` | Recorded mismatch | CRITICAL | 10 |

The `LOOKING_*` rule applies to the head-pose directional events handled by this decision engine. Gaze-direction events are introduced in the later live integration.

The decision engine was tested with short events, sustained events, multiple-face events, and mixed-event sequences.

### 2.2 Event aggregation

**File:** `tests/proctoring_event_aggregator.py`

Input:

`results/proctoring_decisions.json`

Output:

`results/proctoring_aggregation.json`

The aggregator groups events by type and calculates:

- Number of occurrences.
- Total duration.
- Average duration.
- Maximum duration.
- Total score.
- Highest severity for each event type.

The repeated-event test successfully grouped six events into three unique event types.

### 2.3 Repeated-event escalation

**File:** `tests/proctoring_escalation.py`

Input:

`results/proctoring_aggregation.json`

Output:

`results/proctoring_escalation.json`

Escalation rules tested include:

| Event type | Repeated warning | Escalation |
|---|---:|---:|
| Directional head-pose event | 3 occurrences | 5 occurrences |
| `NO_FACE` | 2 occurrences | 3 occurrences |
| `MULTIPLE_FACES` | 2 occurrences | 3 occurrences |
| `IDENTITY_MISMATCH` | Not required | Critical when present |

The normal repeated-event test and a synthetic 11-event escalation test both ran successfully.

### 2.4 Session summary

**File:** `tests/proctoring_session_summary.py`

Output:

`results/proctoring_session_summary.json`

The session summary combines aggregation and escalation information.

One successfully tested six-event dataset produced:

- Total events: 6
- Unique event types: 3
- Total score: 14
- Session risk: `CRITICAL_RISK`
- Highest severity event: `NO_FACE`
- Repeated event types: `LOOKING_RIGHT`, `NO_FACE`

These values belong to that particular test dataset, not to every session.

## Phase 3: Live Proctoring Integration

### 3.1 Live proctoring system

**File:** `tests/live_proctoring_system.py`

This component integrates face presence, face count, head pose, smoothing, temporal duration, event decisions, scores, and session risk.

Output:

`results/live_proctoring_events.json`

The original implementation could record an event at the decision threshold rather than after measuring its full duration. This was corrected in Phase 3.3.

### 3.2 Live session-processing pipeline

**File:** `tests/live_session_pipeline.py`

The pipeline processes live event output using the Phase 2 decision, aggregation, escalation, and summary rules.

Outputs:

- `results/live_proctoring_decisions.json`
- `results/live_proctoring_aggregation.json`
- `results/live_proctoring_escalation.json`
- `results/live_proctoring_session_summary.json`

The pipeline was tested without modifying the standalone Phase 2 scripts.

### 3.3 Accurate event duration

**File:** `tests/live_proctoring_system.py`

The live system was updated to finalize an event when the detected state changes, or when the session ends. It measures the elapsed duration for the completed state and then applies the decision rule.

A successful test recorded events with durations including approximately:

- `NO_FACE`: 3.32 seconds
- `LOOKING_UP`: 3.78 seconds
- `LOOKING_UP`: 4.97 seconds
- `LOOKING_LEFT`: 3.82 seconds
- `LOOKING_UP`: 2.24 seconds
- `LOOKING_RIGHT`: 4.62 seconds

The test completed and generated its JSON output.

### 3.4 Live identity verification

**File:** `tests/live_hybrid_verify.py`

This component combines MediaPipe face detection with periodic DeepFace/FaceNet512 verification against a reference student image.

The live test successfully produced both verified and identity-mismatch results.

**Known limitation:** Verification results fluctuated between `True` and `False` in some tests. A single failed comparison can produce a false mismatch, so stable identity decisions should use repeated evidence, confidence handling, and an explicit unknown state before being used in a high-stakes examination.

### 3.5 Gaze tracking

**File:** `tests/gaze_tracking.py`

The gaze tracker uses MediaPipe facial landmarks and iris positions to estimate gaze direction.

Supported states:

- `GAZE_LEFT`
- `GAZE_RIGHT`
- `GAZE_UP`
- `GAZE_DOWN`
- `GAZE_CENTER`
- `NO_FACE`

The standalone test successfully detected the directional states.

The values are estimates based on facial and iris landmarks, not calibrated eye-tracking measurements.

#### Temporal gaze detection

**File:** `tests/temporal_gaze.py`

Temporal gaze processing uses smoothing and sustained-duration checks to reduce rapid changes in raw gaze classifications.

The tested configuration used:

- Horizontal left threshold: `0.38`
- Horizontal right threshold: `0.62`
- Vertical up threshold: `0.38`
- Vertical down threshold: `0.62`
- Smoothing window: 7 frames
- Sustained duration: 2 seconds

A sustained `GAZE_LEFT` event was successfully detected.

### 3.6 Live gaze proctoring

**File:** `tests/live_gaze_proctoring.py`

This is the latest tested integration. It combines:

- Face presence and face count.
- Head-pose estimation.
- Iris-based gaze estimation.
- Temporal smoothing.
- Sustained-event duration.
- Decision rules and score calculation.
- Session risk calculation.
- JSON event output.

Output:

`results/live_gaze_proctoring_events.json`

The script ran successfully on macOS with the Apple M4 and completed a 38.07-second test session.

The recorded terminal summary was:

```text
Session duration: 38.07 seconds
Total events: 22
Total score: 6
Session risk: MEDIUM_RISK
Saved to: results/live_gaze_proctoring_events.json
```

The output included sustained gaze and head-pose events, `NO_FACE`, and transitions involving `MULTIPLE_FACES` and `NORMAL`.

**The integration runs, but the event log still needs cleanup.** Several short `NORMAL` states were logged with `Decision: UNKNOWN` and score `0`. These entries inflate the event count without adding meaningful violations. The issue has been identified but has not yet been fixed.

## Installation and Environment

Activate the existing environment before running the tests.

For example, if using the existing Conda environment:

```bash
conda activate mediapipe-deepface-test
cd ~/auto-oep-test/hybrid-proctoring
```

Check the Python version and installed packages:

```bash
python --version
python -m pip show mediapipe opencv-python deepface tensorflow tf-keras
```

The project uses two MediaPipe model files:

- `tests/models/blaze_face_short_range.tflite`
- `tests/models/face_landmarker.task`

Confirm both files exist before running the relevant tests. The identity-verification tests also require valid student reference images under `test_data/database/`.

Dependency versions should be checked against the active environment before recreating it. Do not assume that every TensorFlow, Keras, MediaPipe, and RetinaFace version combination is compatible.

## Running the Tests

Run commands from the repository root.

### Core detection tests

```bash
python tests/face_presence_tracker.py
python tests/head_pose_live.py
python tests/temporal_head_pose.py
python tests/unified_proctoring.py
python tests/proctoring_event_logger.py
```

### Phase 2 pipeline

Run these in sequence when the preceding output files have been generated:

```bash
python tests/proctoring_decision_engine.py
python tests/proctoring_event_aggregator.py
python tests/proctoring_escalation.py
python tests/proctoring_session_summary.py
```

### Phase 3 live tests

```bash
python tests/live_proctoring_system.py
python tests/live_session_pipeline.py
python tests/live_hybrid_verify.py
python tests/gaze_tracking.py
python tests/temporal_gaze.py
python tests/live_gaze_proctoring.py
```

These scripts may open the webcam and some may write output files. Run them individually so you can observe each test's behavior and verify its output before proceeding.

### Check syntax before execution

For a script you have just edited:

```bash
python -m py_compile tests/live_gaze_proctoring.py
```

No output indicates that the script passed Python's syntax compilation check. It does not guarantee that runtime behavior is correct.

Press `Q` in the relevant webcam window to end the live test when instructed by the script.

## Risk Scoring

The system uses a rule-based scoring approach. The risk thresholds implemented in the documented pipeline are:

| Total score | Risk classification |
|---:|---|
| 0 | `NORMAL` |
| 1–3 | `LOW_RISK` |
| 4–7 | `MEDIUM_RISK` |
| 8–12 | `HIGH_RISK` |
| More than 12 | `CRITICAL_RISK` |

The same event can contribute different decisions depending on its type and duration. Repeated-event escalation is a separate processing stage.

These thresholds are configurable engineering rules. They have not been demonstrated to represent statistically calibrated probabilities of cheating.

## Output Files

The primary outputs are JSON files in the `results/` directory.

| Output | Purpose |
|---|---|
| `proctoring_events.json` | Standalone recorded events |
| `proctoring_decisions.json` | Decisions and scores per event |
| `proctoring_aggregation.json` | Grouped event statistics |
| `proctoring_escalation.json` | Repeated-event escalation |
| `proctoring_session_summary.json` | Consolidated session results |
| `live_proctoring_events.json` | Live face-presence and head-pose events |
| `live_proctoring_decisions.json` | Decisions from the live session pipeline |
| `live_proctoring_aggregation.json` | Aggregated live-session statistics |
| `live_proctoring_escalation.json` | Live repeated-event escalation |
| `live_proctoring_session_summary.json` | Live session summary |
| `live_gaze_proctoring_events.json` | Live gaze, head-pose, and face-presence events |

Some output files are generated only after the corresponding script runs. Outputs from earlier runs may be overwritten by later tests, depending on each script's implementation.

## Known Issues and Limitations

1. **Normal-event logging:** The Phase 3.6 integration can record short `NORMAL` states with `UNKNOWN` decisions and zero scores. These should be excluded from the meaningful event log.
2. **Identity verification instability:** DeepFace verification can fluctuate across frames. A single failed comparison should not automatically be treated as definitive identity fraud.
3. **Gaze calibration:** Gaze thresholds are heuristic and can vary with lighting, camera placement, head position, glasses, and individual eye geometry.
4. **Head-pose calibration:** A fixed yaw/pitch threshold may not suit every student or camera setup.
5. **Multiple-face detection:** Face-count accuracy depends on camera framing, occlusion, lighting, and detector confidence.
6. **Score interpretation:** A high risk score is a rule-based alert, not proof of misconduct.
7. **Performance:** Identity verification is significantly more expensive than lightweight frame-level detection and should not be executed on every frame.
8. **Validation:** Successful development tests do not establish accuracy, fairness, or reliability across different users and real examination conditions.

## Next Steps

Recommended development order:

1. **Clean up Phase 3.6 event logging:** Do not record `NORMAL`, `IGNORE`, or `UNKNOWN` events as violations.
2. **Re-test event durations:** Confirm that each recorded event represents its actual sustained duration.
3. **Stabilize identity verification:** Introduce repeated comparisons or a confidence-based confirmation rule and test genuine and mismatched identities.
4. **Integrate gaze events into the processing pipeline:** Ensure gaze-direction events receive explicit decisions and are included in aggregation and session summaries.
5. **Run repeatable test scenarios:** Test no face, one face, multiple faces, sustained head movement, sustained gaze movement, and identity mismatch.
6. **Measure performance:** Record frame rate, identity-verification latency, event detection delay, and false-positive rates.
7. **Evaluate with consent-based test data:** Test different lighting conditions, camera positions, and users before considering real examinations.

## Current Milestone

**Completed through Phase 3.6: live integration of face presence, head pose, gaze tracking, temporal event duration, scoring, risk calculation, and JSON output.**

The immediate remaining task is to clean up `NORMAL`/`UNKNOWN` event logging and then re-test the integrated pipeline before adding further functionality.
