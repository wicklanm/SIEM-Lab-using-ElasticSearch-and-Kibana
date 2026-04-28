## Day 27 — Investigating an RDP Brute Force Attack

---

### 📋 Summary

This tutorial covers how a SOC Analyst investigates an **RDP (Remote Desktop Protocol) brute force attack** using the **Elastic SIEM (Kibana)**. The core investigation methodology is the same as SSH brute force (covered in Day 26), but the data sources, Windows Event IDs, and log fields differ significantly since RDP targets a **Windows server** rather than a Linux host.

By the end of this tutorial you will know how to:
- Triage an RDP brute force alert in Kibana
- Apply the 4-question investigation framework to RDP attacks
- Use Windows Event IDs to identify failed and successful RDP logins
- Trace post-authentication activity using Logon IDs
- Check the attacker's IP reputation with AbuseIPDB and GreyNoise
- Configure the RDP brute force Elastic rule to auto-create osTicket tickets

> 💡 **RDP vs SSH:** The investigation workflow is nearly identical to Day 26. The key differences are the log source (Windows Security Event Logs vs Linux auth.log), the event codes used (Windows Event IDs vs SSH event fields), and the target machine (Windows Server on port 3389 vs Linux server on port 22).

---

### 🛠️ Prerequisites

| Component | Purpose |
|-----------|---------|
| Elastic Stack (ELK) | SIEM — stores and queries logs |
| Kibana | Web UI for alert investigation |
| Elastic Agent on Windows Server | Forwards Windows Security Event logs |
| Sysmon (optional but recommended) | Additional process/network telemetry |
| RDP Brute Force detection rule | Created in a prior day of the challenge |
| osTicket with Webhook connector | Ticketing system (configured in Day 26) |
| AbuseIPDB / GreyNoise accounts | Free IP reputation lookups |

---

### 🔑 Key Windows Event IDs for RDP Investigation

Understanding these Event IDs is essential before diving into the investigation:

| Event ID | Meaning | Relevance |
|----------|---------|-----------|
| `4625` | Failed logon attempt | Core indicator of brute force activity |
| `4624` | Successful logon | Confirms whether an attacker got in |
| `4634` | Logoff / session end | Used to determine session duration |
| `4648` | Logon using explicit credentials | Can indicate lateral movement |

> RDP logons that succeed show `event.code: 4624` with `winlog.event_data.LogonType: 10` (RemoteInteractive). Always filter on Logon Type 10 to isolate true RDP sessions from other logon types.

---

### 🔍 Part 1 — Investigating the RDP Brute Force Alert

#### Step 1: Navigate to Alerts in Kibana

1. Log in to your **Kibana** web interface.
2. Click the **hamburger menu (☰)** in the top-left corner.
3. Under the **Security** section, click **Alerts**.
4. Filter the alert list to find alerts triggered by your RDP rule (e.g., `MyDFIR-RDP-Brute-Force-Attempt`).

You may see significantly more RDP alerts than SSH alerts — RDP is one of the most heavily scanned services on the internet and will attract real-world brute force traffic almost immediately after being exposed.

---

#### Step 2: Expand an Alert and Identify Key Fields

Click on one of the RDP brute force alerts to expand its details. Note the following fields:

| Field | Description |
|-------|-------------|
| `source.ip` | The attacker's IP address |
| `user.name` | The account targeted (e.g., `Administrator`) |
| `event.code` | Windows Event ID (look for `4625` = failed logon) |
| `winlog.event_data.LogonType` | Should be `3` (network) or `10` (RDP/RemoteInteractive) |
| `host.name` | The targeted Windows server |
| `event.count` | How many failed attempts in the detection window |

> **Example:** Source IP `91.238.181.8` made repeated failed logon attempts against the `Administrator` account on your Windows server.

---

#### Step 3: Apply the 4-Question Investigation Framework

Use this same framework from Day 26, now applied to RDP-specific data.

---

##### ❓ Question 1 — Is this IP known for brute force activity?

**Tool 1: [AbuseIPDB](https://www.abuseipdb.com)**
1. Copy the `source.ip` from the alert.
2. Search it on AbuseIPDB.
3. Review:
   - **Abuse Confidence Score** — how likely this IP is malicious
   - **Report categories** — look for "Brute-Force" or "Hacking" tags
   - **Total reports** — high counts indicate repeat offenders

**Tool 2: [GreyNoise](https://viz.greynoise.io)**
1. Search the same IP.
2. Look for tags such as:
   - `RDP Bruteforcer`
   - `RDP Scanner`
3. Classification will be **Malicious**, **Benign**, or **Unknown**.

> **Note for lab/on-prem environments:** If you're running an isolated lab where the attacker machine is on a private IP range, AbuseIPDB and GreyNoise won't return results for RFC 1918 addresses (e.g., `192.168.x.x`, `10.x.x.x`). In a real production environment, always run this check — it's a quick and high-value enrichment step.

> **Example finding:** IP `91.238.181.8` was found on AbuseIPDB with reports linked to RDP brute force attacks. Even at 26% abuse confidence, the report category context confirms this is attack traffic.

---

##### ❓ Question 2 — Were any other user accounts targeted by this IP?

Search Kibana for all failed logon events from the attacker's IP to see the full scope of targeted accounts:

```kql
source.ip : "<attacker-IP>" and event.code : "4625"
```

Add `user.name` to your selected columns to see which accounts were attempted. Common RDP targets include:
- `Administrator`
- `admin`
- `user`
- `guest`
- Custom usernames from breach wordlists

> If the attacker only targeted `Administrator`, it may be an automated scanner. If a long and specific username list is observed, it could suggest a more targeted attack.

---

##### ❓ Question 3 — Were any login attempts successful?

Search for successful RDP logons from the attacker's IP using Windows Event ID `4624`:

```kql
source.ip : "<attacker-IP>" and event.code : "4624"
```

To specifically isolate RDP sessions, add the Logon Type filter:

```kql
source.ip : "<attacker-IP>" and event.code : "4624" and winlog.event_data.LogonType : "10"
```

- **No results** → The brute force failed. Document findings and close/downgrade the alert.
- **Results found** → A successful RDP login occurred. Proceed immediately to Question 4.

---

##### ❓ Question 4 — If successful, what happened after the login?

This is the most critical part of the investigation. If you find a successful logon:

**Step 4a: Record the Logon Session Details**

From the `4624` event, note:
- **Timestamp** (date, time, and timezone — always use UTC)
- **`host.name`** — the compromised Windows server
- **`winlog.event_data.TargetUserName`** — the account that was logged into
- **`winlog.event_data.LogonId`** — the session ID (a hex value like `0x3E7`). This is your anchor for tracing all activity within this session.

**Step 4b: Trace the Session Using the Logon ID**

Query all events associated with this specific logon session:

```kql
winlog.event_data.LogonId : "<logon-id-hex>" and host.name : "<hostname>"
```

Review the events in chronological order. Look for:
- **Process creation events** (Sysmon Event ID 1 / Windows Event ID 4688) — what programs were launched?
- **File creation or modification** — were any scripts or tools dropped?
- **Registry modifications** — persistence mechanisms?
- **Network connections** — outbound C2 traffic?
- **Privilege escalation** — was the session used to add users or modify groups?

**Step 4c: Determine Session Duration**

Find the `4634` (logoff) event with the matching Logon ID to determine how long the session lasted:

```kql
event.code : "4634" and winlog.event_data.LogonId : "<logon-id-hex>"
```

> A very short session (seconds) may indicate an automated script or credential validation. A longer session (minutes to hours) suggests hands-on-keyboard activity and warrants a full incident response.

**Step 4d: Check for New Logon IDs**

During a session, Windows may generate new Logon IDs for spawned processes or privilege changes. Look at the latest logs in your session trace and check if new Logon IDs appear — query those as well to ensure full coverage.

> 🚨 **A confirmed successful RDP brute force is a security incident.** Escalate per your organization's incident response process, consider isolating the host, and reset the compromised account credentials immediately.

---

### 🔄 RDP vs SSH — Key Investigation Differences

| Aspect | SSH (Day 26) | RDP (Day 27) |
|--------|-------------|-------------|
| Target OS | Linux | Windows |
| Default Port | 22 | 3389 |
| Log Source | `/var/log/auth.log` | Windows Security Event Log |
| Failed Login Field | `system.auth.ssh.event: Failed` | `event.code: 4625` |
| Successful Login Field | `event.outcome: success` | `event.code: 4624` |
| Session Tracking | SSH session ID | Windows Logon ID (hex) |
| Logon Type Filter | N/A | `LogonType: 10` for RDP |
| Post-auth visibility | Shell commands, process spawns | Windows Event IDs + Sysmon |

---

### 🎫 Part 2 — Configuring osTicket for RDP Alerts

If you completed the osTicket webhook setup in Day 26 for SSH alerts, the RDP configuration follows the exact same process — you're simply applying it to a different rule.

#### Step 1: Edit the RDP Brute Force Detection Rule

1. In Kibana, go to:  
   **Security → Rules → Detection Rules (SIEM)**

2. Find your RDP Brute Force rule (e.g., `MyDFIR-RDP-Brute-Force-Attempt`).

3. Click **Edit rule settings**.

4. Go to the **Actions** tab.

5. Set **Action frequency** to: `Per rule run`

---

#### Step 2: Add the osTicket Webhook Action

1. Select your **osTicket Webhook connector** from the dropdown.

2. In the **Body** field, use the same XML payload template as your SSH rule:

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
  ]]></message>
</ticket>
```

3. Click **Save changes**.

---

#### Step 3: Verify Tickets Are Being Created

1. Trigger or wait for a new RDP brute force event.
2. Open your **osTicket** interface and confirm a new ticket was generated automatically.
3. Assign the ticket to yourself or a team member.
4. Work through the 4-question framework, document your findings in the ticket reply, and close or escalate accordingly.

> All tickets — open and closed — are stored in osTicket and serve as your **audit trail** for every alert investigated.

---

### ✅ Summary of Key Takeaways

| Topic | Key Point |
|-------|-----------|
| Alert triage | Filter Kibana Security Alerts by your RDP rule name |
| Critical Event IDs | `4625` = failed logon, `4624` = success, `4634` = logoff |
| IP enrichment | Use AbuseIPDB + GreyNoise for all external IPs |
| Scope check | Query `event.code: 4625` by attacker IP to see all targeted users |
| Success detection | Query `event.code: 4624` + `LogonType: 10` for RDP sessions |
| Session tracing | Use the Logon ID (hex) to pivot across all events in a session |
| Automation | Apply the same osTicket webhook action to the RDP rule |
| Protocol parity | The 4-question framework works the same for SSH and RDP |

---

### 🔗 Resources


> *Part of the MyDFIR 30-Day SOC Analyst Challenge. This tutorial is based on Day 27 content and is intended for educational purposes.*
