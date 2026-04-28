# Day 28 — Investigating a Mythic C2 Agent
> **MyDFIR 30-Day SOC Analyst Challenge | Day 28 of 30**  
> 🎥 [Watch the original video](https://www.youtube.com/watch?v=b11TuDx_CjU&list=PLG6KGSNK4PuBb0OjyDIdACZnb8AoNBeq6&index=30)

---

## 📋 Summary

This is the most advanced investigation in the challenge so far. Day 28 shifts from reactive alert triage (brute force) to a **deeper post-compromise investigation** — analyzing telemetry from a **Mythic C2 (Command and Control) agent** that was planted on your Windows server during the attack simulation in Day 20.

You'll learn how to detect a C2 agent both when you know its name and when you don't, how to reconstruct a full attack timeline using **Sysmon telemetry and Process GUIDs**, and how to create a new detection rule that alerts on C2-driven shell activity. Finally, you'll wire the Mythic C2 alert into osTicket for automated ticket generation.

**What you'll learn:**
- What C2 frameworks are and how Mythic works
- How to find a C2 agent in Kibana when you know its name
- How to hunt for C2 activity without prior knowledge using dashboards and network telemetry
- How to use **Sysmon Event IDs 1, 3, 11, and 29** to build an attack timeline
- How to pivot through events using **Process GUIDs** and **Parent Process GUIDs**
- How to extract and use file hashes for further threat intelligence lookups
- How to create a custom Elastic detection rule for C2 shell command activity
- How to configure **Custom Highlighted Fields** in Elastic alert rules
- How to integrate the Mythic C2 alert with osTicket

---

## 🛠️ Prerequisites

| Component | Purpose |
|-----------|---------|
| Elastic Stack (ELK) | SIEM — stores and queries Sysmon + Windows logs |
| Kibana | Web UI for investigation and rule management |
| Sysmon on Windows Server | Critical — provides process, file, and network telemetry |
| Mythic C2 server (from Day 20) | The C2 infrastructure used to generate telemetry |
| Apollo agent (or equivalent) | The C2 agent that beaconed from your Windows server |
| Suspicious Activity dashboard | Built in a prior day; used for threat hunting |
| osTicket Webhook connector | Already configured in Days 26–27 |

> ⚠️ **Sysmon is essential for this investigation.** Without it, you will not have process creation (Event ID 1), network connection (Event ID 3), file creation (Event ID 11), or file hash (Event ID 29) events. If Sysmon is not generating logs in your Elastic instance, revisit your agent configuration before proceeding.

---

## 🧠 Background: What Is a C2 Framework?

A **Command and Control (C2) framework** is a tool used by attackers to remotely control compromised machines after gaining initial access. The compromised machine runs a lightweight **agent** that periodically calls back ("beacons") to the attacker's C2 server to receive instructions.

**Mythic** is an open-source C2 framework commonly used in red team simulations and security research. In this challenge, the **Apollo agent** was used as the payload — a Windows-compatible agent configured to communicate with the Mythic server over HTTP (port 80) or TCP (port 9999).

**Common C2 behaviors that generate detectable telemetry:**

| Behavior | Sysmon Event | What It Looks Like |
|---------|-------------|-------------------|
| Agent executable created on disk | Event ID 11 (File Create) | `.exe` dropped in `C:\Users\Public\Downloads` |
| Agent process starts | Event ID 1 (Process Create) | Unusual process spawned by PowerShell |
| Agent beacons to C2 server | Event ID 3 (Network Connection) | Outbound connection to external IP on port 80/443/9999 |
| File hash logged | Event ID 29 (File Executable Detected) | SHA1/SHA256 of the agent on disk |
| Shell commands executed via C2 | Event ID 1 (Process Create) | `cmd.exe` spawned as a child of the agent |

---

## 🔍 Part 1 — Investigating When You Know the Agent Name

In a lab setting, you may already know the name of the malicious executable from a prior alert or incident report. Use it as your starting point.

### Step 1: Search for the Agent in Kibana

1. Log in to Kibana and go to **Analytics → Discover** (or use the search bar in the Security section).
2. Search for the known agent filename:

```kql
process.name : "svchost-yourname.exe"
```

> Replace `svchost-yourname.exe` with the actual name of your Mythic agent. In this challenge, participants typically name it something like `svchost-[username].exe` to keep it distinct.

3. **Sort events from oldest to newest** — this is critical. Starting from the first event lets you trace the attack chain forward in time rather than backwards.

4. Note the key fields in each result:
   - `@timestamp` — when the event occurred
   - `winlog.event_data.UtcTime` — the actual time recorded on the endpoint (important — this may differ from Elastic ingestion time)
   - `process.pid` — the process ID
   - `winlog.event_data.ProcessGuid` — the globally unique process identifier (your primary pivot point)
   - `winlog.event_data.ParentProcessGuid` — what spawned this process

> 💡 **Log ingestion delay is real.** Elastic does not always receive events in the exact order they occurred. Always compare `winlog.event_data.UtcTime` (the endpoint timestamp) against `@timestamp` (the ingestion time) to establish the true sequence of events.

---

## 🔍 Part 2 — Hunting for C2 When You Don't Know the Agent Name

In real-world investigations, you won't always know the agent's filename in advance. Here are two effective methods to discover it.

### Method A: Suspicious Activity Dashboard

1. In Kibana, open the **Suspicious Activity dashboard** you built in a prior day.
2. Change the time range to the last **30 days** to capture all historical activity.
3. Look at the **"Process Initiated Network Connections"** table.

**Red flags to look for:**
- An executable running from an unusual path like `C:\Users\Public\Downloads\` — legitimate applications almost never run from here
- Multiple processes all connecting to the **same destination IP** — a strong indicator of C2 beaconing
- Outbound connections on port **80 (HTTP)** or **443 (HTTPS)** from a process that has no business making web connections (e.g., a random `.exe` in Downloads)
- PowerShell making outbound network connections

Once you identify a suspicious process and destination IP, use those as your investigation starting points.

---

### Method B: Network Telemetry Query

Search directly for outbound network connections (Sysmon Event ID 3) and look for unusual destinations:

```kql
event.code : "3" and not winlog.event_data.DestinationIp : ("127.0.0.1" OR "::1")
```

Add `winlog.event_data.DestinationIp`, `process.name`, and `process.executable` as visible columns. Sort by destination IP to group connections to the same external address — repeated beaconing to a single IP is a hallmark of C2 traffic.

> 💡 **RITA (Real Intelligence Threat Analytics)** by Black Hills Information Security is a dedicated tool for detecting C2 beaconing in network logs. It identifies long-duration, periodic connections that are typical of C2 frameworks. While not used directly in this challenge, it's worth exploring for deeper network-based C2 detection.

---

## 🔍 Part 3 — Building the Attack Timeline with Process GUIDs

The **Process GUID** is the most powerful pivot point available in Sysmon telemetry. Unlike a PID (which can be reused by the OS), a Process GUID uniquely identifies a specific process instance across its entire lifetime.

### Step 1: Find the First Suspicious Network Connection

Search for outbound connections to the C2 server's IP:

```kql
event.code : "3" and winlog.event_data.DestinationIp : "<C2-server-IP>"
```

1. Open the **earliest** event.
2. Copy the **`winlog.event_data.ProcessGuid`** value.
3. Note all timestamps from these network events — take note if multiple GUIDs appear (this indicates different processes made C2 connections).

---

### Step 2: Pivot Using the Process GUID

Wrap the GUID in quotes and search all events for that specific process:

```kql
"<ProcessGuid-value>"
```

> Always wrap GUIDs in double quotes — without them, Elastic may parse the hyphens and return incomplete results.

Sort results **oldest to newest**. You should now see the full lifecycle of this process:

| Event | Sysmon ID | What It Shows |
|-------|-----------|---------------|
| PowerShell session opens | 1 | Parent process that initiated the download |
| Network connection (download) | 3 | File download from attacker's HTTP server |
| File creation | 11 | Agent `.exe` written to `C:\Users\Public\Downloads\` |
| Executable file detected | 29 | SHA1/SHA256 hash of the agent recorded |
| Process creation (agent starts) | 1 | Agent `.exe` launched — **ProcessGuid becomes ParentProcessGuid** |
| Network connection (beacon) | 3 | Agent calls back to C2 on port 80 |
| Child process creation (shell cmds) | 1 | `cmd.exe` spawned for each `shell <command>` run via C2 |

---

### Step 3: Pivot to the Agent's Own Process GUID

Once the agent starts, its **ProcessGuid becomes the ParentProcessGuid** for everything it spawns. Search for the agent's new GUID to trace its post-execution activity:

```kql
"<Agent-ProcessGuid>"
```

Here you'll find:
- The outbound HTTP beacon connections the agent makes
- Every `cmd.exe` child process spawned when the attacker runs `shell <command>` through the C2

> **Why are C2 shell commands distinct?** When an attacker uses the `shell <command>` option in Mythic, it spawns a **new `cmd.exe` process** for each command. Each command gets its own Process GUID as a child of the agent, making them individually traceable. Commands run natively within the C2 framework's internal functions (without using `shell`) are less likely to generate cmd.exe logs.

---

### Step 4: Extract the File Hash

From the **Sysmon Event ID 29** (Executable File Detected) log, extract the file hash:

- `winlog.event_data.SHA1` — the SHA1 hash of the agent executable
- `winlog.event_data.SHA256` — the SHA256 hash (if available)

Use these hashes to look up the file on threat intelligence platforms:
- **[VirusTotal](https://www.virustotal.com)** — check if the hash is flagged by any AV engines
- **[MalwareBazaar](https://bazaar.abuse.ch)** — check if the sample is a known malware family

> In this lab scenario, your custom-compiled Mythic agent will likely show as "unknown" or "clean" on VirusTotal since it was freshly generated. In real incidents, checking file hashes is a quick way to confirm whether a file is known malware.

---

### Step 5: Look for Evidence of Data Access

Search for file access events around the time of the compromise. In particular, look for sensitive files being opened:

```kql
process.name : "notepad.exe" OR file.name : "passwords*"
```

In the challenge simulation, an attacker commonly opens a `passwords.txt` file to simulate data exfiltration. Document any such findings — accessed file name, timestamp, and the process that opened it.

---

### Step 6: Document Your Timeline

As you investigate, build a written timeline of events. Here's an example format:

```
[YYYY-MM-DD HH:MM:SS UTC] — Network connection from PowerShell to <C2-IP>:9999 (file download)
[YYYY-MM-DD HH:MM:SS UTC] — File created: C:\Users\Public\Downloads\svchost-[name].exe
[YYYY-MM-DD HH:MM:SS UTC] — Executable detected (SHA1: <hash>)
[YYYY-MM-DD HH:MM:SS UTC] — Process created: svchost-[name].exe (PID: XXXX)
[YYYY-MM-DD HH:MM:SS UTC] — Network connection from agent to <C2-IP>:80 (C2 beacon)
[YYYY-MM-DD HH:MM:SS UTC] — cmd.exe spawned: shell whoami
[YYYY-MM-DD HH:MM:SS UTC] — cmd.exe spawned: shell ipconfig
[YYYY-MM-DD HH:MM:SS UTC] — notepad.exe accessed: passwords.txt
```

A documented timeline like this is what you would include in an incident report or osTicket response.

---

## 🛡️ Part 4 — Creating a New Detection Rule for C2 Shell Activity

Beyond the existing Mythic agent detection rule, you can create a second, more targeted rule that fires whenever the C2 executes shell commands — even if the agent itself is not explicitly identified.

### Step 1: Build the Query

The following query targets `cmd.exe` processes that were spawned by a non-system user (i.e., an interactive session, not a Windows service):

```kql
event.code : "1" and winlog.event_data.OriginalFileName : "Cmd.Exe" and not winlog.event_data.ParentUser : "NT AUTHORITY\\SYSTEM"
```

Verify the query returns the expected shell command events before creating the rule.

### Step 2: Create the Detection Rule

1. In Kibana, go to **Security → Rules → Detection Rules (SIEM) → Create New Rule**.
2. Select **Custom Query** as the rule type.
3. Paste the query above.
4. Under **Required Fields**, add:
   - `winlog.event_data.CommandLine` — shows the actual command that was run
   - `winlog.event_data.ParentUser` — who spawned the cmd process
   - `winlog.event_data.ProcessGuid` — for pivoting
   - `host.name` — which machine it ran on
5. Set **Severity**: Medium
6. Set **Risk Score**: 60
7. Set the rule to run **every 1 minute**
8. Add a **Webhook Action** pointed at your osTicket connector with the same XML body used in Days 26–27.

### Step 3: Add Custom Highlighted Fields

Custom Highlighted Fields make the most important context visible at the top of each alert without having to expand every field manually.

1. In the rule's **Edit rule settings**, go to the **About** tab.
2. Scroll to **Custom Highlighted Fields**.
3. Add the same fields you set as Required Fields:
   - `winlog.event_data.CommandLine`
   - `winlog.event_data.ParentUser`
   - `winlog.event_data.ProcessGuid`
   - `host.name`
4. Save the rule.

Now when an alert fires, those fields appear prominently at the top of the alert detail view.

### Step 4: Test the Rule

1. Log in to your **Mythic server** and navigate to **Callbacks**.
2. Open your active agent session and run:
   ```
   shell whoami
   ```
3. Wait up to 60 seconds, then check **Security → Alerts** in Kibana for the new alert.
4. Confirm a ticket was also auto-created in osTicket.

---

## 🎫 Part 5 — Connecting the Existing Mythic C2 Rule to osTicket

If you haven't already wired your pre-existing Mythic detection rule (created in Day 20) to osTicket, do so now using the same process as Days 26–27:

1. Go to **Security → Rules → Detection Rules (SIEM)**.
2. Find your `MyDFIR-Mythic-C2-Apollo-Agent-Detected` rule.
3. Click **Edit rule settings → Actions**.
4. Select the **osTicket Webhook connector**.
5. Set the body:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ticket alert="true" autorespond="true" source="API">
  <n>Elastic</n>
  <email>api@osticket.com</email>
  <subject>{{rule.name}}</subject>
  <phone>318-555-8634X123</phone>
  <message type="text/plain"><![CDATA[
    Please investigate the alert generated by {{rule.name}}.
    ---
    Description: {{rule.description}}
    ---
    Link: {{rule.url}}
    Alert ID: {{alert.id}}
  ]]></message>
</ticket>
```

6. Click **Save changes**.
7. Trigger the agent again to confirm a ticket is created in osTicket.

---

## ✅ Summary of Key Takeaways

| Topic | Key Point |
|-------|-----------|
| C2 detection (known agent) | Search by filename in Kibana, sort oldest to newest |
| C2 detection (unknown agent) | Use Suspicious Activity dashboard → Process Initiated Network Connections |
| Primary pivot point | `ProcessGuid` — uniquely identifies a process and all events it generated |
| Parent/child relationships | When the agent spawns cmd.exe, its GUID becomes the `ParentProcessGuid` |
| Log ingestion delay | Always compare `UtcTime` (endpoint) vs `@timestamp` (ingestion) for true sequence |
| File hash extraction | Sysmon Event ID 29 logs SHA1/SHA256 of executables — use with VirusTotal |
| Shell command detection | Query `event.code: "1"` for `Cmd.Exe` processes not spawned by SYSTEM |
| Custom Highlighted Fields | Surfaces critical alert context without manual field expansion |
| Timeline documentation | Build a timestamped (UTC) event log to include in your incident report/ticket |

---

## 🔑 Key Sysmon Event IDs Reference

| Event ID | Name | Use in This Investigation |
|----------|------|--------------------------|
| 1 | Process Create | Trace agent launch, PowerShell session, cmd.exe shell commands |
| 3 | Network Connection | Identify C2 beacon traffic and download connections |
| 11 | File Create | Confirm agent was written to disk |
| 29 | File Executable Detected | Extract SHA1/SHA256 hash of the agent |

---

## 🔗 Resources

- 📺 [Original Video — MyDFIR Day 28](https://www.youtube.com/watch?v=b11TuDx_CjU)
- 📺 [Day 20 — Mythic C2 Setup](https://www.youtube.com/playlist?list=PLG6KGSNK4PuBb0OjyDIdACZnb8AoNBeq6) *(prerequisite — where the agent was deployed)*
- 📖 [Mythic C2 Framework (GitHub)](https://github.com/its-a-feature/Mythic)
- 📖 [Sysmon Event ID Reference — Microsoft](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)
- 🔎 [RITA — Real Intelligence Threat Analytics](https://github.com/activecm/rita)
- 🔎 [VirusTotal](https://www.virustotal.com)
- 🔎 [MalwareBazaar](https://bazaar.abuse.ch)
- 📖 [Elastic Rule Action Variables](https://www.elastic.co/guide/en/security/current/rules-ui-create.html#rule-action-variables)
- 🌐 [MyDFIR SOC Community](https://www.skool.com/mydfir)

---

> *Part of the MyDFIR 30-Day SOC Analyst Challenge. This tutorial is based on Day 28 content and is intended for educational purposes only. Mythic C2 and similar tools should only be used in authorized lab environments.*
