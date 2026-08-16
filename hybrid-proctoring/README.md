# Hybrid Proctoring — Phase 4 Testing

This document records the tests performed after the initial MediaPipe + DeepFace identity verification tests.

The objective of this phase was to move from individual model testing toward a unified real-time proctoring pipeline.

---

## 1. Phase Objective

The previous phase successfully tested:

- MediaPipe face detection
- Multiple-face detection
- DeepFace FaceNet512 identity verification
- Live identity verification
- FaceNet512 distance and threshold evaluation

However, identity verification alone is not sufficient for an online examination proctoring system.

This phase focuses on detecting and recording suspicious examination behaviors:

- No face detected
- Multiple faces detected
- Looking left
- Looking right
- Looking up
- Looking down
- Normal/forward position
- Sustained suspicious behavior
- Proctoring event logging

---

# 2. Environment

The tests were performed on:

- macOS
- Apple M4
- Python 3.11 environment
- MediaPipe
- OpenCV
- DeepFace
- TensorFlow
- Keras

Current environment used for the project:

```text
Environment:
mediapipe-deepface-test