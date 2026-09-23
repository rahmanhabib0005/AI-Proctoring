# AI Proctoring System — Phase 2

## Repeated Event Escalation & Final Session Summary

This document records the work completed after the initial Event Aggregation stage.

The previous Phase 2 implementation established the following pipeline:

```text
Webcam
   ↓
MediaPipe Face Detection
   ↓
Unified Proctoring Detector
   ↓
Proctoring Event Logger
   ↓
proctoring_events.json
   ↓
Decision Engine
   ↓
proctoring_decisions.json
   ↓
Event Aggregator
   ↓
proctoring_aggregation.json
```

This continuation adds:

```text
proctoring_aggregation.json
        ↓
Repeated Event Escalation
        ↓
proctoring_escalation.json
        ↓
Final Session Summary
        ↓
proctoring_session_summary.json
```

---

# 1. Repeated Event Escalation

## Purpose

The Repeated Event Escalation component identifies whether an event occurred repeatedly during the session.

The purpose is to distinguish between:

```text
Single Event
```

and:

```text
Repeated Event
```

without changing the original event severity or score.

The escalation layer is implemented separately from the Event Aggregator.

---

# 2. Escalation Input

The escalation engine reads:

```text
results/proctoring_aggregation.json
```

The aggregation file contains event-level information such as:

```text
occurrences
total_duration
average_duration
maximum_duration
total_score
highest_severity
```

Example:

```json
{
    "LOOKING_RIGHT": {
        "occurrences": 3,
        "total_duration": 9.5,
        "average_duration": 3.17,
        "maximum_duration": 4.0,
        "total_score": 3,
        "highest_severity": "WARNING"
    }
}
```

---

# 3. Escalation Rules

The current prototype uses occurrence-based escalation rules:

```text
Occurrences = 1
        ↓
NONE


Occurrences = 2
        ↓
REPEATED


Occurrences = 3–4
        ↓
REPEATED_WARNING


Occurrences = 5+
        ↓
ESCALATED
```

These are prototype rules for the research implementation.

They should not be presented as scientifically validated cheating thresholds.

---

# 4. Important Separation

The escalation layer does not replace the original severity.

For example:

```text
LOOKING_RIGHT
Base Severity = WARNING
Occurrences = 3
Escalation = REPEATED_WARNING
```

The system therefore preserves:

```text
Base Severity
```

and separately records:

```text
Repeated Status
Escalation Status
```

This allows later research experiments to modify escalation rules without modifying the original detector or decision engine.

---

# 5. Repeated Event Escalation Script

The component is implemented as:

```text
repeated_event_escalation.py
```

It reads:

```text
results/proctoring_aggregation.json
```

and writes:

```text
results/proctoring_escalation.json
```

The output contains:

```text
occurrences
base_severity
total_score
repeated
escalation
```

for each event type.

---

# 6. Verified Escalation Output

The current test session produced:

```text
Total Events: 6
Total Score: 14
Session Risk: CRITICAL_RISK
Unique Event Types: 3

Highest Severity Event: NO_FACE
Highest Severity Level: 3
```

Event-level result:

| Event Type     | Occurrences | Base Severity | Repeated | Escalation       |
| -------------- | ----------: | ------------- | -------- | ---------------- |
| LOOKING_RIGHT  |           3 | WARNING       | YES      | REPEATED_WARNING |
| NO_FACE        |           2 | VIOLATION     | YES      | REPEATED         |
| MULTIPLE_FACES |           1 | HIGH          | NO       | NONE             |

The result confirms that the escalation engine correctly processes the aggregation output.

---

# 7. Final Session Summary

After the escalation layer was verified, the Final Session Summary component was implemented.

Its purpose is to consolidate the results of the session into one structured output.

The final summary answers:

> What happened during the entire proctoring session?

---

# 8. Final Session Summary Input

The summary engine uses:

```text
results/proctoring_aggregation.json
```

and:

```text
results/proctoring_escalation.json
```

The two sources provide:

```text
Aggregation
    ↓
Session statistics


Escalation
    ↓
Repeated and escalated event information
```

These are combined into one final session-level representation.

---

# 9. Final Session Summary Output

The output file is:

```text
results/proctoring_session_summary.json
```

The summary contains:

```text
session_risk
total_events
total_score
unique_event_types
highest_severity_event
highest_severity_level
repeated_event_count
repeated_event_types
escalated_event_count
escalated_event_types
events
```

---

# 10. Final Verified Session Summary

The current session produced:

```text
Session Risk:
CRITICAL_RISK

Total Events:
6

Total Score:
14

Unique Event Types:
3

Highest Severity Event:
NO_FACE

Highest Severity Level:
3

Repeated Event Count:
2

Repeated Event Types:
LOOKING_RIGHT
NO_FACE

Escalated Event Count:
1

Escalated Event Types:
LOOKING_RIGHT
```

---

# 11. Complete Event Breakdown

## LOOKING_RIGHT

```text
Occurrences: 3
Base Severity: WARNING
Total Score: 3
Repeated: YES
Escalation: REPEATED_WARNING
```

## NO_FACE

```text
Occurrences: 2
Base Severity: VIOLATION
Total Score: 6
Repeated: YES
Escalation: REPEATED
```

## MULTIPLE_FACES

```text
Occurrences: 1
Base Severity: HIGH
Total Score: 5
Repeated: NO
Escalation: NONE
```

---

# 12. Complete Phase 2 Pipeline

The complete Phase 2 architecture is now:

```text
                    AI PROCTORING
                         │
                         ▼
                    Webcam Input
                         │
                         ▼
              MediaPipe Face Detection
                         │
                         ▼
             Unified Proctoring Detector
                         │
                         ▼
                  Event Logger
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
          Repeated Event Escalation
                         │
                         ▼
           proctoring_escalation.json
                         │
                         ▼
             Final Session Summary
                         │
                         ▼
       proctoring_session_summary.json
```

---

# 13. Current Results Directory

The current results directory contains:

```text
results/
├── proctoring_events.json
├── proctoring_decisions.json
├── proctoring_aggregation.json
├── proctoring_escalation.json
└── proctoring_session_summary.json
```

Each file represents a different processing stage.

```text
proctoring_events.json
        ↓
Event records


proctoring_decisions.json
        ↓
Event interpretation


proctoring_aggregation.json
        ↓
Session-level statistics


proctoring_escalation.json
        ↓
Repeated-event analysis


proctoring_session_summary.json
        ↓
Complete session summary
```

---

# 14. Architecture Principle

The processing pipeline intentionally uses separate layers:

```text
Detection
    ↓
Logging
    ↓
Decision
    ↓
Aggregation
    ↓
Escalation
    ↓
Summary
```

Each layer has a specific responsibility.

This makes the system easier to:

* Test
* Debug
* Extend
* Evaluate
* Replace individual components
* Perform controlled experiments

For example, the escalation rules can be changed without modifying the face detector.

Similarly, the face detector can later be replaced or extended without changing the final session summary format.

---

# 15. Risk Interpretation

The current system produces risk indicators based on prototype rules.

For example:

```text
CRITICAL_RISK
```

is currently a system-generated session classification.

It should not be treated as definitive proof that a student cheated.

The architecture intentionally separates:

```text
Observed Event
        ↓
System Classification
        ↓
Risk Indicator
        ↓
Human/Admin Review
```

This distinction will be important when the system is evaluated as an academic research project.

---

# 16. Phase 2 Completion Status

```text
[✓] MediaPipe Face Detection

[✓] Unified Proctoring Detector

[✓] Proctoring Event Logger

[✓] Decision Engine

[✓] Decision JSON Output

[✓] Event Aggregator

[✓] Session Risk Calculation

[✓] Event Occurrence Aggregation

[✓] Duration Aggregation

[✓] Score Aggregation

[✓] Severity Hierarchy

[✓] Highest Severity Detection

[✓] Repeated Event Escalation

[✓] Escalation JSON Output

[✓] Final Session Summary

[✓] Final Session Summary JSON
```

---

# 17. Phase 2 Final Checkpoint

The current verified result is:

```text
Total Events: 6
Total Score: 14
Session Risk: CRITICAL_RISK

Highest Severity:
NO_FACE

Severity Level:
3

Repeated Event Types:
2

Escalated Event Types:
1
```

The entire event-processing pipeline is now operational.

---

# 18. Next Phase

The next stage should focus on **controlled evaluation** rather than immediately adding more AI models.

The proposed Phase 3 pipeline is:

```text
Phase 2
Complete Event Processing Pipeline
        │
        ▼
Phase 3
Controlled Proctoring Scenarios
        │
        ▼
Scenario Test Runner
        │
        ▼
Expected vs Actual Results
        │
        ▼
Evaluation Metrics
        │
        ▼
Error Analysis
        │
        ▼
Phase 4
Advanced AI / Multi-Camera Proctoring
```

Example controlled scenarios include:

```text
Normal Face Presence
        ↓
Expected: No violation


No Face
        ↓
Expected: NO_FACE


Multiple Faces
        ↓
Expected: MULTIPLE_FACES


Repeated Looking Right
        ↓
Expected: LOOKING_RIGHT
        ↓
Repeated Event Escalation


Mixed Events
        ↓
Expected:
NO_FACE
MULTIPLE_FACES
LOOKING_RIGHT
        ↓
Correct Aggregation
        ↓
Correct Escalation
        ↓
Correct Session Summary
```

---

# 19. Research Evaluation Direction

The next phase should establish whether the current pipeline behaves consistently under controlled conditions.

The evaluation should eventually measure things such as:

```text
Event Detection Accuracy
Event Classification Accuracy
False Positive Rate
False Negative Rate
Event Duration Accuracy
Repeated Event Detection
Session-Level Consistency
```

These measurements should be collected from controlled test sessions rather than assumed from a single successful run.

---

# 20. Current Project Milestone

At this point, the project has progressed from:

```text
Raw Webcam Detection
```

to:

```text
Structured Proctoring Evidence Pipeline
```

The system can now:

```text
Detect
   ↓
Record
   ↓
Classify
   ↓
Aggregate
   ↓
Identify Repetition
   ↓
Identify Escalation
   ↓
Generate Session Summary
```

The next implementation target is:

```text
PHASE 3
CONTROLLED PROCTORING SCENARIO TEST RUNNER
```

The existing Phase 2 components should remain unchanged unless Phase 3 testing identifies a specific defect.
