# Home SOC Lab: AD + Sysmon + Splunk Detection Pipeline

## Title & Summary
- Project name: Home SOC Lab: AD + Sysmon + Splunk Detection Pipeline
- Summary:This project simulates a small enterprise environment to practice detection engineering and incident response from the ground up. I built an Active Directory domain (mubbyspark1.local) with a domain controller and joined client, deployed Sysmon for endpoint telemetry, and forwarded logs into a Splunk indexer using the Universal Forwarder. To validate the pipeline, I simulated real attack techniques including a brute force login attack and Atomic Red Team techniques like account creation and built a scheduled Splunk alert to detect failed logon activity (Event ID 4625), confirming end-to-end detection from raw event to triggered alert.

## Objectives

- Build a functioning Active Directory environment, including domain controller setup and client domain join
- Deploy Sysmon for endpoint telemetry and forward logs into Splunk
- Simulate real attack techniques (brute force login, Atomic Red Team account creation) to generate detectable activity
- Build and validate a working Splunk alert to detect suspicious activity end-to-end
- Troubleshoot real infrastructure issues as they came up — AD rebuild, DNS trust relationships, alert scheduling conflicts — rather than following a scripted walkthrough
- 
## Architecture / Lab Environment

The lab simulates a small enterprise network with a domain controller, Splunk indexer, Windows 10 client, and an isolated attacker machine.

<img width="600" height="453" alt="Home SOC Lab Architecture Diagram" src="https://github.com/user-attachments/assets/38804634-681d-4a09-a3ad-642c0b6e5c43" />

**Network Setup:**
- Domain: `mubbyspark1.local`
- Network: `192.168.10.0/24`
- Splunk server: `192.168.10.10`
- Active Directory: `192.168.10.7`
- Attacker machine: `192.168.10.250`

**Tools used:**
- Windows Server 2022 (Domain Controller)
- Windows 10 (Domain-joined client)
- Sysmon (endpoint telemetry)
- Splunk Universal Forwarder + Splunk indexer
- Atomic Red Team (attack simulation)

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
