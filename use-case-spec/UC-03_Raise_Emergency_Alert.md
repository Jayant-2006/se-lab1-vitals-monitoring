# Use-Case Specification: UC-03 Raise Emergency Alert

| Field | Detail |
|---|---|
| **Use Case ID / Name** | UC-03 Raise Emergency Alert |
| **Primary Actor** | On-Call Caregiver (receiver of the alert) |
| **Secondary Actor** | Remote Patient (source of telemetry) |
| **Trigger** | A vitals reading breaches a configured clinical threshold |
| **Related Requirements** | FR-001, FR-002, FR-003, FR-004, NFR-002 |
| **Relationships** | `«include»` UC-04 Notify Caregiver; extended by UC-05 Escalate to Next Caregiver |
| **Priority** | High |

**Brief Description:** When a patient's telemetry crosses a critical threshold, the system detects it, raises an emergency alert and notifies the on-call caregiver, who acknowledges it so the patient can receive timely care.

## Preconditions
1. The patient is enrolled and their wearable device is paired and streaming telemetry (UC-01 succeeded).
2. Clinical thresholds are configured for the patient (UC-08).
3. An escalation matrix with at least one on-call caregiver exists for the patient.
4. The caregiver has a registered device/phone number for notifications.

## Postconditions
**Success:** An alert record is stored with metric, value, timestamp, caregiver ID and acknowledgement time; escalation is stopped; the alert status is "Acknowledged" and visible on the dashboard (UC-07).

**Failure:** If no caregiver acknowledges after the full matrix is exhausted, the alert is marked "Unacknowledged – Critical", a fallback voice call is placed, and the event is logged for clinical review. No alert is silently dropped.

## Main Success Scenario
| Step | Actor | Action |
|---|---|---|
| 1 | Patient device | Streams a new vitals reading (e.g., HR = 152 BPM) to the system. |
| 2 | System | Validates the reading (UC-02) and compares it with the patient's thresholds. |
| 3 | System | Detects a critical breach (HR > 140 BPM) within 2 s and flags it (FR-001). |
| 4 | System | Creates an emergency alert record with patient, metric, value and timestamp. |
| 5 | System | Notifies the primary on-call caregiver by push and SMS within 3 s (UC-04, FR-002). |
| 6 | Caregiver | Opens the alert and views the patient's current vitals. |
| 7 | Caregiver | Acknowledges the alert with a single action (UC-06). |
| 8 | System | Stops escalation, records caregiver ID and acknowledgement time, and updates the dashboard. |
| 9 | System | Ends the use case; the caregiver proceeds with the clinical response. |

## Alternate Flow
**A1: Caregiver does not acknowledge within 60 s (extension point after step 5, `«extend»` UC-05)**
- A1.1 System waits 60 s without receiving an acknowledgement.
- A1.2 System escalates the alert to the next caregiver in the matrix (secondary, then tertiary) and sends push and SMS (FR-003).
- A1.3 System logs the escalation event with timestamp.
- A1.4 The new caregiver acknowledges; the flow resumes at **step 7**.
- A1.5 If the matrix is exhausted, the use case ends with the failure postcondition above.

## Exception (brief)
**E1: Push/SMS delivery fails at step 5:** the system retries once and falls back to a voice call (NFR-002), then continues at step 6.

## Special Requirements
- Detection ≤ 2 s; first notification ≤ 3 s (FR-001, FR-002).
- Alert content contains patient health data and must be encrypted and audit-logged (NFR-002).
