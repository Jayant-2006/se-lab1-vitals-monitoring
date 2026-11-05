# Requirements Specification: Remote Patient Vitals Alert & Monitoring App

## 1. Functional Requirements

| ID | Type | Description | Priority | Acceptance Criteria | Rationale |
|---|---|---|---|---|---|
| **FR-001** | Functional | The system shall continuously evaluate ingested patient vitals (SpO2, Heart Rate, BP) against the patient's configured clinical thresholds and flag a critical anomaly within 2 seconds of receipt. | High | **Pass:** Heart rate > 140 BPM or SpO2 < 90% is flagged as critical within 2 s of ingestion. **Fail:** The breach is ignored or flagged after > 5 s. | Post-operative deterioration can be fatal within minutes; fast, automated detection replaces manual observation. |
| **FR-002** | Functional | On a confirmed critical anomaly, the system shall create an emergency alert and notify the primary on-call caregiver via push notification and SMS within 3 seconds, including patient name, breached metric, value and timestamp. | High | **Pass:** Primary caregiver receives push and SMS containing all four fields within 3 s of flagging. **Fail:** No notification, missing fields, or delivery > 5 s. | The caregiver is the human responder; the alert must be immediate and actionable without opening the app. |
| **FR-003** | Functional | If the notified caregiver does not acknowledge an alert within 60 seconds, the system shall automatically escalate it to the next caregiver in the configured escalation matrix (secondary, then tertiary) until acknowledged. | High | **Pass:** An unacknowledged alert reaches the secondary caregiver at 60 s ± 2 s and the tertiary at 120 s ± 2 s. **Fail:** Alert stalls with the first caregiver or skips a level. | Guarantees that no emergency is lost when a caregiver is unreachable (the core purpose of the caregiver matrix). |
| **FR-004** | Functional | The system shall allow an on-call caregiver to acknowledge an alert in a single action, which stops further escalation and records the caregiver ID and acknowledgement time. | High | **Pass:** One tap/click acknowledges the alert, halts escalation within 2 s, and an audit record is stored. **Fail:** Escalation continues after acknowledgement or no record is saved. | Prevents alert fatigue and duplicate responses, and provides a clinical and legal audit trail. |
| **FR-005** | Functional | The system shall provide a dashboard on which the remote patient views their own live vitals and alert status, and the on-call caregiver views live vitals, alert history and threshold settings of assigned patients, with data refreshed at least every 5 seconds. | Medium | **Pass:** Dashboard shows the latest reading no older than 5 s; the caregiver sees only assigned patients. **Fail:** Stale data (> 10 s) or access to non-assigned patients. | Gives the patient reassurance and the caregiver situational awareness beyond the alert itself. Also provides the screen where caregivers maintain per-patient thresholds used by FR-001. |

## 2. Non-Functional Requirements

| ID | Type | Description | Priority | Acceptance Criteria | Rationale |
|---|---|---|---|---|---|
| **NFR-001** | Performance & Security | The telemetry ingestion gateway shall support at least 500 concurrent continuous telemetry streams with 99.99% monthly uptime, 95th-percentile ingestion latency ≤ 500 ms, and all device-to-gateway traffic encrypted with TLS 1.2 or higher with mutual device authentication. | High | **Pass:** Load test with 500 simulated streams shows p95 latency ≤ 500 ms and no data loss; security scan confirms TLS ≥ 1.2 and rejection of unauthenticated devices. **Fail:** Latency target missed, dropped streams, or unauthenticated device accepted. | Supports a hospital-scale cohort of patients at once; vitals are sensitive health data and must be protected in transit. |
| **NFR-002** | Reliability & Privacy/Compliance | The system shall deliver at least 99.9% of emergency alerts successfully, using retry and a fallback channel (voice call) on failure; patient health data shall be encrypted at rest (AES-256), protected by role-based access, and every access/alert action audit-logged in line with applicable health-data privacy law (e.g., India's DPDP Act 2023, HIPAA where applicable). | High | **Pass:** Fault-injection test (push outage) still delivers ≥ 99.9% of 1,000 test alerts via fallback; penetration/audit review confirms encryption at rest, RBAC and complete audit logs. **Fail:** Alerts lost during channel failure, or unencrypted or unlogged data access found. | A missed alert is a patient-safety failure, and health data carries legal obligations. |

## 3. Traceability (Requirements ↔ Use Cases)

| Use Case (Diagram) | Related Requirements |
|---|---|
| UC-01 Transmit Vitals | FR-001, NFR-001 |
| UC-02 Validate Telemetry Data (`«include»`) | FR-001, NFR-001 |
| UC-03 Raise Emergency Alert | FR-001, FR-002, NFR-002 |
| UC-04 Notify Caregiver (`«include»`) | FR-002, NFR-002 |
| UC-05 Escalate to Next Caregiver (`«extend»`) | FR-003 |
| UC-06 Acknowledge Alert | FR-004 |
| UC-07 View Live Vitals Dashboard | FR-005 |
| UC-08 Configure Thresholds | FR-001, FR-005 |
