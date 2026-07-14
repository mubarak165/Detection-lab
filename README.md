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

## Build Process

### Active Directory Domain Services (AD DS) Setup
- Installed Windows Server 2022 and promoted it to a Domain Controller using AD DS
  <img width="1600" height="774" alt="image" src="https://github.com/user-attachments/assets/5c879af1-dcd8-46c0-a712-0f435e19c5df" />

- Created the domain `mubbyspark1.local`
  <img width="3822" height="1657" alt="Screenshot 2026-07-14 032223" src="https://github.com/user-attachments/assets/d0d559cd-eab7-4c93-8b05-b7bd4cb602af" />

- Configured static IP `192.168.10.7` for the DC to ensure DNS resolution stayed consistent across reboots
<img width="3202" height="1147" alt="Screenshot 2026-07-14 033317" src="https://github.com/user-attachments/assets/1f9e97e2-ac67-4e2a-992d-f304161941e6" />

### Client Domain Join
- Deployed a Windows 10 client on the same subnet (`192.168.10.0/24`)
  <img width="3802" height="1490" alt="image" src="https://github.com/user-attachments/assets/f869f8d0-12ce-4b7e-82e0-e5f349339584" />

- Joined the client to the `mubbyspark1.local` domain
  <img width="3377" height="1860" alt="image" src="https://github.com/user-attachments/assets/e1dbfe81-02c4-40cf-884b-aa88640e8318" />

- Verified domain join by confirming the client appeared under **Active Directory Users and Computers**
  <img width="3825" height="1640" alt="image" src="https://github.com/user-attachments/assets/2f63e5bb-8a75-43e7-b0af-284dd45f433b" />

## Sysmon Deployment
- Installed Sysmon on both the Domain Controller and the Windows 10 client for detailed endpoint telemetry (process creation, network connections, etc.)
  -Sysmon active on the  Windows Server(Activedirectory)
  <img width="3800" height="1917" alt="image" src="https://github.com/user-attachments/assets/198b0284-81de-4d21-bc29-0f18a3cd8d06" />

  -Sysmon active on the  Windows Client
  <img width="3797" height="1760" alt="image" src="https://github.com/user-attachments/assets/bbd81ecd-0b82-4b09-8d81-6e233b2dc385" />

- Used [Olaf Hartong's Sysmon-Modular config](https://github.com/olafhartong/sysmon-modular) as a base configuration, which provides a modular, MITRE ATT&CK-aligned ruleset for broader detection coverag

### Splunk Universal Forwarder Setup
- Installed the Splunk Universal Forwarder on the Windows server(Active directory) and the Windows 10 client
  -Splunk  Fowarder  running in WIndows CLient
  <img width="3820" height="2007" alt="image" src="https://github.com/user-attachments/assets/38013c5d-7f9f-4beb-b8eb-1a632241706b" />

-Splunk  Fowarder  running in WIndows Server(Active directory)
 <img width="3820" height="2007" alt="image" src="https://github.com/user-attachments/assets/1a2ae292-bbd5-4dfd-81e5-77a715cfdc8a" />

- Configured `inputs.conf` to forward:C
  - Windows Security Event Logs
  - Sysmon Event Logs
    <img width="3840" height="2160" alt="image" src="https://github.com/user-attachments/assets/cfc37576-5461-47f2-9825-f77cd596d087" />

- Pointed forwarders to the Splunk indexer at `192.168.10.10`
  <img width="3840" height="2160" alt="image" src="https://github.com/user-attachments/assets/0a89d76c-d16a-46ee-b0dc-8b30d78787b2" />

- Confirmed data ingestion in Splunk via `index=* | stats count by host`
<img width="3800" height="2015" alt="image" src="https://github.com/user-attachments/assets/ddfeacc2-f34b-4a65-8e89-ca716f15c7ec" />

### Troubleshooting: Server Crash → Rebuild → Client Connectivity Issue
- Mid-build, the Domain Controller crashed and had to be reconfigured/rebuilt
- After reconfiguring the server, the Windows 10 client — which had originally been joined to the old server instance — no longer connected properly to the rebuilt DC
- This happened because the client still held onto its original domain trust/machine account information tied to the old server instance, which no longer matched after the rebuild
- Diagnosed the issue by checking DNS resolution from the client (`nslookup mubbyspark1.local`) to confirm whether the client could properly resolve and communicate with the rebuilt DC
- Resolved by removing the client from the domain and rejoining it to `mubbyspark1.local`, which re-established a valid trust relationship with the rebuilt DC
  
## 5. Attack Simulation #1 — Brute Force

### Method
- Used **Crowbar** on Kali Linux to perform a brute force attack against the domain-joined Windows 10 client
- Configured a custom password list of 20 passwords, including the correct password within the list, to simulate a realistic brute force scenario against the target

<!-- SCREENSHOT: password list file (e.g. passwords.txt) showing the 20 entries -->
<img width="2065" height="1037" alt="image" src="https://github.com/user-attachments/assets/680e656e-d0b1-4148-8e48-083cf11ba802" />

<!-- SCREENSHOT: Crowbar running in terminal, showing the command/flags and attack in progress -->
<img width="2932" height="275" alt="image" src="https://github.com/user-attachments/assets/863c5564-85b4-42b3-9ab4-ebf84584705a" />

### Detection in Splunk
- Failed and successful login attempts were captured via Windows Security Event Logs, forwarded through the Splunk Universal Forwarder
- Queried Splunk for **Event ID 4625** (failed logon) to confirm the brute force attempts were being logged
- Search query used: `index=* EventCode=4625 | table _time, host, Account_Name, Failure_Reason`

<!-- SCREENSHOT 2: Splunk search results showing the raw 4625 events -->
![Splunk 4625 Search Results](images/splunk-4625-results.png)

### Detection Logic / Alert Built
- Configured a scheduled Splunk alert to detect repeated 4625 (failed logon) events

<!-- SCREENSHOT 3: Alert configuration screen (search query, trigger condition, schedule) -->
![Splunk Alert Configuration](images/splunk-alert-config.png)

- Validated the detection pipeline (search → condition → trigger) end-to-end by confirming the alert fired successfully

<!-- SCREENSHOT 4: Triggered Alerts history showing the alert fired with a timestamp -->
![Splunk Triggered Alert](images/splunk-triggered-alert.png)

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
