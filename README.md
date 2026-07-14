# Home SOC Lab: AD + Sysmon + Splunk Detection Pipeline

## 1. Title & Summary
- Project name: Home SOC Lab: AD + Sysmon + Splunk Detection Pipeline
- Summary: A short 2-3 sentence description of what you built and why (e.g., simulate a small enterprise environment to practice detection engineering, incident response, and adversary emulation).

## 2. Objectives
- What you set out to learn/prove (e.g., build an AD environment, ingest logs into Splunk, simulate real attack techniques, and validate detections).

## 3. Architecture / Lab Environment
- Diagram showing: Windows Server (DC) ↔ Windows Client ↔ Splunk indexer
- Tools used: Windows Server 2022, Sysmon, Splunk Universal Forwarder + Splunk indexer, Atomic Red Team
- Network setup (IP scheme, domain name `mubbyspark1.local`, etc.)

## 4. Build Process
- AD DS setup (domain controller promotion)
- Client domain join
- Sysmon deployment + config used (SwiftOnSecurity config)
- Splunk Universal Forwarder + inputs.conf configuration
- Troubleshooting moment: crashed server → rebuild → DNS trust relationship issue

## 5. Attack Simulation #1 — Brute Force
- What you did (method/tool used)
- What you saw in Splunk (screenshot, search query, Event ID 4625)
- Detection logic/alert built:
  - Configured a scheduled Splunk alert to detect repeated 4625 (failed logon) events
  - Screenshot of alert configuration (search query, trigger condition, schedule)
  - Screenshot of the alert firing successfully in **Triggered Alerts** history, with timestamp
  - Validated the detection pipeline (search → condition → trigger) end-to-end

## 6. Attack Simulation #2 — Atomic Red Team
- Techniques run (e.g., T1136.001 - account creation)
- Sample terminal output
- Corresponding Splunk/Event Viewer detection (Event ID 4720, 4732, Sysmon Event ID 1)
- Screenshot of raw events found

## 7. Challenges & Troubleshooting
- Domain rebuild issue, cached credentials, cryptography/DNS warnings during AD DS install
- What happened and how you diagnosed it
- Alert scheduling conflict: initial alert configuration used a real-time search window, which conflicted with cron-based scheduling (`rt time values are not allowed` error). Resolved by switching to a relative time range (`-5m` to `now`) with a standard cron schedule, which allowed the alert to trigger successfully
- Scoped email delivery as a stretch goal; focused validation on confirming the core detection and alerting pipeline (search → trigger condition → fired alert) rather than the notification channel

## 8. Key Takeaways / Lessons Learned
- What you learned about AD trust relationships, Sysmon telemetry, log pipelines, detection engineering basics
- Understanding of Splunk's alert scheduling model (real-time vs. cron-based search windows) and how they interact

## 9. Next Steps / Future Work
- Additional MITRE ATT&CK techniques
- Splunk alerts/dashboards
- SIEM correlation rules
- Completing email/notification delivery for alerts
- Expanding to more clients
