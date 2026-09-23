# AI Proctoring System — Phase 2

## Proctoring Event Processing & Session Aggregation

This document records the work completed for the AI proctoring prototype during Phase 2.

The goal of this phase is to transform raw face-detection observations into structured proctoring events, decisions, and session-level statistics.

---

## 1. Current Pipeline

The current processing pipeline is:

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

The system is no longer limited to detecting faces. It now converts observations into structured events and evaluates those events at the session level.

---

## 2. Components Completed

### 2.1 MediaPipe Face Detection

MediaPipe is currently used as the face-detection component.

The detector provides the information required by the unified proctoring detector to identify events such as:

* No face detected
* Multiple faces detected
* Face direction / looking direction

---

### 2.2 Unified Proctoring Detector

The unified detector converts face-detection observations into standardized proctoring event types.

Current event examples include:

```text
NO_FACE
MULTIPLE_FACES
LOOKING_RIGHT
```

The purpose of using standardized event names is to keep the detection layer independent from the later decision and aggregation layers.

---

### 2.3 Proctoring Event Logger

Detected events are stored in:

```text
results/proctoring_events.json
```

The logger preserves event-level information that can later be processed by the decision engine.

The event log acts as the raw structured record of the proctoring session.

---

### 2.4 Decision Engine

The Decision Engine converts logged events into structured decisions.

Output:

```text
results/proctoring_decisions.json
```

The decision layer assigns information such as:

* Event type
* Severity
* Severity level
* Score
* Duration
* Event-related decision information

This creates a separation between:

```text
Detection
    ↓
Decision
```

This separation is important because detection determines **what happened**, while the decision engine determines **how the system interprets it**.

---

## 3. Event Severity Hierarchy

The severity system currently uses a hierarchy so that the most serious event can be identified correctly.

Example severity levels:

```text
Level 1 → WARNING
Level 2 → HIGH
Level 3 → VIOLATION
```

The exact event-to-severity mapping is controlled by the decision engine.

The important rule implemented today is:

> When determining the highest-severity event for a session, the system compares the numeric severity level rather than relying on alphabetical or arbitrary ordering.

---

## 4. Event Aggregator

The Event Aggregator processes:

```text
results/proctoring_decisions.json
```

and generates:

```text
results/proctoring_aggregation.json
```

The aggregator summarizes events by event type.

For each event type, it can provide information such as:

* Total occurrences
* Total duration
* Maximum duration
* Total score
* Highest severity
* Highest severity level

Example structure:

```text
Event
 ├── Occurrences
 ├── Total Duration
 ├── Maximum Duration
 ├── Total Score
 ├── Highest Severity
 └── Highest Severity Level
```

---

## 5. Severity Hierarchy Fix

A severity hierarchy issue was identified and corrected during testing.

The previous implementation could select the wrong event as the highest-severity event because severity names should not be compared as ordinary strings.

The corrected implementation uses the numerical severity level.

Conceptually:

```text
WARNING
   ↓
HIGH
   ↓
VIOLATION
```

The system now correctly selects the event with the highest numerical severity.

---

## 6. Verified Aggregation Result

The latest test produced:

```text
Total events: 6
Total score: 14
Session risk: CRITICAL_RISK

Highest severity event: NO_FACE
Highest severity level: 3
```

The event-level aggregation was:

| Event Type     | Occurrences | Total Duration | Max Duration | Score | Highest Severity |
| -------------- | ----------: | -------------: | -----------: | ----: | ---------------- |
| LOOKING_RIGHT  |           3 |           9.5s |         4.0s |     3 | WARNING          |
| NO_FACE        |           2 |           8.5s |         6.0s |     6 | VIOLATION        |
| MULTIPLE_FACES |           1 |           1.5s |         1.5s |     5 | HIGH             |

Therefore:

```text
Total Events = 3 + 2 + 1
             = 6
```

and:

```text
Total Score = 3 + 6 + 5
            = 14
```

The highest severity is:

```text
NO_FACE
Severity Level = 3
Severity = VIOLATION
```

The aggregation result is therefore internally consistent.

---

## 7. Current Output

The current session produces:

```text
results/
├── proctoring_events.json
├── proctoring_decisions.json
└── proctoring_aggregation.json
```

These files represent three different stages of processing:

```text
proctoring_events.json
        ↓
Raw structured proctoring events

proctoring_decisions.json
        ↓
Interpreted event decisions

proctoring_aggregation.json
        ↓
Session-level event statistics
```

---

## 8. Current Example

The current test session can be summarized as:

```text
Session Risk:
CRITICAL_RISK

Total Events:
6

Total Score:
14

Highest Severity:
NO_FACE

Highest Severity Level:
3
```

Event breakdown:

```text
LOOKING_RIGHT
    Occurrences: 3
    Total Duration: 9.5s
    Maximum Duration: 4.0s
    Score: 3
    Severity: WARNING


NO_FACE
    Occurrences: 2
    Total Duration: 8.5s
    Maximum Duration: 6.0s
    Score: 6
    Severity: VIOLATION


MULTIPLE_FACES
    Occurrences: 1
    Total Duration: 1.5s
    Maximum Duration: 1.5s
    Score: 5
    Severity: HIGH
```

---

## 9. Phase 2 Status

### Completed

```text
[✓] MediaPipe face detection
[✓] Unified proctoring detector
[✓] Proctoring event logging
[✓] Decision engine
[✓] Decision JSON output
[✓] Event aggregation
[✓] Session risk calculation
[✓] Event occurrence aggregation
[✓] Duration aggregation
[✓] Score aggregation
[✓] Severity hierarchy
[✓] Highest-severity event detection
```

### Current checkpoint

```text
Event Aggregator
       ↓
       ✓ VERIFIED
```

The current aggregator should not be modified while moving to the next component unless a new test identifies a real aggregation problem.

---

## 10. Next Component

The next planned component is:

```text
Repeated Event Escalation
```

The intended pipeline will become:

```text
proctoring_aggregation.json
        ↓
Repeated Event Escalation
        ↓
proctoring_escalation.json
        ↓
Final Session Summary
```

The purpose of this component is to distinguish isolated events from repeated events.

For example:

```text
LOOKING_RIGHT × 1
→ WARNING

LOOKING_RIGHT × 3
→ REPEATED_WARNING

LOOKING_RIGHT × 5+
→ ESCALATED
```

These thresholds are prototype research rules and should not be presented as scientifically validated cheating thresholds.

---

## 11. Design Principle for Escalation

The escalation layer should remain separate from the existing aggregation layer.

The architecture should be:

```text
Detection
    ↓
Event Logging
    ↓
Decision
    ↓
Aggregation
    ↓
Escalation
    ↓
Final Session Risk
```

The escalation engine should therefore consume the aggregation output rather than changing the detector or decision engine.

This keeps each component independently testable.

---

## 12. Important Research Note

The current severity levels, scores, repeated-event thresholds, and session-risk rules are **prototype decision rules** for the research implementation.

They should not currently be interpreted as scientifically validated measurements of cheating probability.

Validation should be performed later using controlled test sessions and appropriate evaluation metrics.

---

## 13. Phase 2 Development Roadmap

The planned progression is:

```text
Phase 2
│
├── Face Detection                     ✓
├── Unified Event Detection            ✓
├── Event Logging                      ✓
├── Decision Engine                    ✓
├── Event Aggregation                  ✓
├── Severity Hierarchy Fix             ✓
│
├── Repeated Event Escalation           → NEXT
│
├── Final Session Summary               → NEXT
├── Multi-camera Event Fusion           → LATER
├── AI/ML Behavioral Analysis           → LATER
└── Full Proctoring Session Evaluation  → LATER
```

---

## 14. Current Conclusion

The event-processing foundation of the proctoring prototype is now working.

The system can currently:

```text
Detect
   ↓
Record
   ↓
Interpret
   ↓
Aggregate
   ↓
Calculate Session Risk
   ↓
Identify Highest-Severity Event
```

The latest verified result is:

```text
6 Events
14 Total Score
CRITICAL_RISK
NO_FACE = Highest Severity
Severity Level = 3
```

The next implementation task is therefore:

```text
REPEATED EVENT ESCALATION ENGINE
```

No changes to the existing aggregator are required before starting that component.
