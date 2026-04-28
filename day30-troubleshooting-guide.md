## Day 30 — Troubleshooting Common Errors in the SOC Lab
---

### 📋 Summary

🎉 **Congratulations on reaching Day 30!** This final video is a troubleshooting reference — a collection of the most common errors encountered throughout the challenge, with step-by-step fixes for each. If something broke at any point during Days 1–29 and you couldn't move forward, this is where to look.

The video covers **six specific errors** that trip up most participants, spanning Kibana, Fleet Server, Elastic Agent enrollment, and the osTicket webhook integration. This document organizes each error as its own self-contained troubleshooting section so you can jump directly to the one you're hitting.

**Errors covered (in video order):**
1. Kibana Connection Error (cannot reach Kibana in the browser)
2. Fleet Server Failed — Timed Out
3. Fail to Execute Request to Fleet Server
4. x509 Certificate Signed by Unknown Authority
5. Cannot Connect — Invalid Settings (osTicket webhook connector)
6. Error Calling Webhook — Request Failed

---

### 🛠️ General Troubleshooting Principles

Before diving into specific errors, keep these principles in mind. They apply to nearly every problem you'll encounter in this lab:

**1. Always check service status first.**  
Most issues are simply a stopped service. Before anything else, verify the relevant service is running on its host.

**2. Check logs before guessing.**  
Every component writes logs. Reading them tells you the actual error, not just that something failed.

**3. Firewalls and VPC rules are the silent killers.**  
If two components can't talk to each other, check your cloud firewall (Vultr Firewall Group) and any OS-level firewall rules on the server (`ufw`) before assuming software is misconfigured.

**4. Restart services after editing config files.**  
Changes to `.yml` configuration files have no effect until the service is restarted.

**5. Give services time to start.**  
Elasticsearch and Kibana can take 30–90 seconds to be fully ready after a restart. A connection error immediately after starting a service is often just impatience.

---

### ❌ Error 1 — Kibana Connection Error
> **Video timestamp: 00:00**

#### Symptom
You navigate to `http://<ELK-server-IP>:5601` in your browser and receive a connection refused, timeout, or "site can't be reached" error.

#### Causes and Fixes

**Cause A: Kibana service is not running**

SSH into your ELK server and check the service:

```bash
systemctl status kibana.service
```

If the output shows `inactive (dead)` or `failed`, start it:

```bash
systemctl start kibana.service
```

To confirm it's running:

```bash
systemctl status kibana.service
```

Look for `active (running)` in the output. Then wait 30–60 seconds before trying to reach the UI in your browser.

---

**Cause B: Elasticsearch is not running (Kibana can't start without it)**

Kibana depends on Elasticsearch. If Elasticsearch is down, Kibana will fail to start or will show a connection error in the browser.

Check Elasticsearch first:

```bash
systemctl status elasticsearch.service
```

If it's not running, start it:

```bash
systemctl start elasticsearch.service
```

Then restart Kibana:

```bash
systemctl restart kibana.service
```

Always start Elasticsearch before Kibana.

---

**Cause C: Firewall is blocking port 5601**

If the service is running but you still can't reach Kibana, your cloud firewall may be blocking the port. In Vultr:

1. Log in to the Vultr dashboard.
2. Go to **Firewall** → find your Firewall Group.
3. Verify there is an **inbound rule** allowing **TCP port 5601** from your IP (or from anywhere, `0.0.0.0/0`, during the lab).
4. Verify the ELK server is **assigned** to this Firewall Group.

Also check the OS-level firewall on the ELK server:

```bash
ufw status
```

If `ufw` is active and port 5601 isn't listed, allow it:

```bash
ufw allow 5601/tcp
```

---

**Cause D: Kibana is bound to the wrong interface**

If Kibana is configured to listen only on `localhost` (127.0.0.1), it won't be reachable from outside. Check `/etc/kibana/kibana.yml`:

```bash
grep server.host /etc/kibana/kibana.yml
```

If `server.host` is set to `localhost` or `127.0.0.1`, change it to the server's public IP or `0.0.0.0`:

```bash
nano /etc/kibana/kibana.yml
```

Update:

```yaml
server.host: "0.0.0.0"
```

Save, then restart Kibana:

```bash
systemctl restart kibana.service
```

---

### ❌ Error 2 — Fleet Server Failed: Timed Out
> **Video timestamp: 03:08**

#### Symptom
When enrolling a Fleet Server using the setup command in Kibana, the process hangs and eventually returns a timeout error. The Fleet Server never appears as healthy in **Management → Fleet → Agents**.

#### Causes and Fixes

**Cause A: Elasticsearch is not reachable from the Fleet Server**

The Fleet Server needs to communicate with Elasticsearch on port **9200**. If the ELK server's firewall doesn't allow the Fleet Server's IP to connect on port 9200, enrollment will time out.

In Vultr, add an inbound rule to the ELK server's Firewall Group:
- Protocol: TCP
- Port: 9200
- Source: `<Fleet-server-IP>/32` (restrict to Fleet Server only, more secure) or `0.0.0.0/0` (open for the lab)

Also confirm Elasticsearch is listening on the correct interface (not just localhost). In `/etc/elasticsearch/elasticsearch.yml`:

```bash
grep network.host /etc/elasticsearch/elasticsearch.yml
```

If it's set to `localhost`, update it:

```yaml
network.host: 0.0.0.0
```

Restart Elasticsearch after any change:

```bash
systemctl restart elasticsearch.service
```

---

**Cause B: Fleet Server port 8220 is blocked**

Agents communicate with Fleet Server on port **8220**. Ensure that port is open in the Vultr Firewall Group for the Fleet Server.

Add an inbound rule on the Fleet Server's firewall group:
- Protocol: TCP
- Port: 8220
- Source: `0.0.0.0/0` (or restrict to your VPC range)

---

**Cause C: Incorrect Fleet Server URL in Kibana**

In Kibana, go to **Management → Fleet → Settings** and verify the **Fleet Server host** URL matches the Fleet Server's actual public IP:

```
https://<Fleet-server-IP>:8220
```

If this URL is wrong (e.g., still set to a placeholder or old IP), update it and save.

---

### ❌ Error 3 — Fail to Execute Request to Fleet Server
> **Video timestamp: 04:46**

#### Symptom
When running the Elastic Agent enrollment command on a target host (Windows Server or Ubuntu Server), you receive an error like:

```
Error: fail to enroll: fail to execute request to fleet-server: ...
```

The agent cannot reach the Fleet Server at all.

#### Causes and Fixes

**Cause A: Network connectivity between the agent host and Fleet Server is blocked**

From the host you're trying to enroll, test connectivity to the Fleet Server:

```bash
# On Linux
curl -k https://<Fleet-server-IP>:8220

# On Windows PowerShell
Test-NetConnection -ComputerName <Fleet-server-IP> -Port 8220
```

If the connection is refused or times out, the Fleet Server's firewall is blocking the connection. Review the Vultr Firewall Group for the Fleet Server and ensure port 8220 is open.

---

**Cause B: Fleet Server service is not running on the Fleet Server host**

SSH into the Fleet Server and check:

```bash
systemctl status elastic-agent.service
```

If it's not running:

```bash
systemctl start elastic-agent.service
```

---

**Cause C: Using the wrong IP or port in the enrollment command**

Double-check the enrollment command generated by Kibana. The `--url` flag must point to the **Fleet Server's public IP** on port **8220**, not the ELK server:

```bash
# Correct format
sudo ./elastic-agent install \
  --url=https://<FLEET-SERVER-IP>:8220 \
  --enrollment-token=<your-token> \
  --insecure
```

A common mistake is accidentally using the ELK server's IP (port 9200 or 5601) instead of the Fleet Server's IP (port 8220).

---

### ❌ Error 4 — x509: Certificate Signed by Unknown Authority
> **Video timestamp: 08:57**

#### Symptom
When running the Elastic Agent enrollment command, the process fails with:

```
Error: fail to enroll: fail to execute request to fleet-server:
x509: certificate signed by unknown authority
Error: enroll command failed with exit code: 1
```

#### Explanation

This error occurs because the Fleet Server uses a **self-signed TLS certificate** (automatically generated during setup in this lab environment). The Elastic Agent on the enrolling host does not recognize this certificate as trusted because it wasn't issued by a known Certificate Authority (CA).

This is expected behavior in a self-managed lab — you are not using a publicly-trusted certificate. The fix is to explicitly tell the agent to accept the self-signed certificate.

#### Fix: Add the `--insecure` flag

Append `--insecure` to the end of your enrollment command:

**Linux (Ubuntu Server):**

```bash
sudo ./elastic-agent install \
  --url=https://<Fleet-server-IP>:8220 \
  --enrollment-token=<your-token> \
  --insecure
```

**Windows Server (PowerShell, run as Administrator):**

```powershell
.\elastic-agent.exe install `
  --url=https://<Fleet-server-IP>:8220 `
  --enrollment-token=<your-token> `
  --insecure
```

> ⚠️ **Security note:** The `--insecure` flag tells the agent to skip certificate chain validation. This is acceptable in a controlled lab environment, but **never use this flag in production**. In production, always configure Fleet Server with a certificate signed by a trusted CA. For more information, see [Elastic's SSL/TLS documentation](https://www.elastic.co/guide/en/fleet/current/secure-connections.html).

---

**Already installed the agent without `--insecure`?**

If the agent was already partially installed and is now in a broken state, uninstall it first, then re-install with the flag:

```bash
# Linux
sudo /usr/share/elastic-agent/bin/elastic-agent uninstall

# Windows (PowerShell as Administrator)
& "C:\Program Files\Elastic\Agent\elastic-agent.exe" uninstall
```

Then re-run the install command with `--insecure` as shown above.

---

**Alternative: Re-enroll an already-installed agent**

If the agent is installed but failing to connect, you can re-enroll without reinstalling:

```bash
# Linux
sudo elastic-agent enroll \
  --url=https://<Fleet-server-IP>:8220 \
  --enrollment-token=<your-token> \
  --insecure

# Windows (PowerShell as Administrator)
elastic-agent enroll `
  --url=https://<Fleet-server-IP>:8220 `
  --enrollment-token=<your-token> `
  --insecure
```

Then restart the agent:

```bash
# Linux
sudo systemctl restart elastic-agent.service

# Windows
Restart-Service elastic-agent
```

---

### ❌ Error 5 — Cannot Connect: Invalid Settings (osTicket Webhook Connector)
> **Video timestamp: 11:28**

#### Symptom
When testing your osTicket webhook connector in Kibana (**Stack Management → Connectors**), clicking **Test** returns an error like:

```
Cannot connect - invalid settings
```
or
```
Connection refused
```

No tickets are being created in osTicket even when alerts fire.

#### Causes and Fixes

**Cause A: osTicket server is not running**

SSH into your osTicket server and verify Apache (or your web server) and MySQL are running:

```bash
# Check Apache
systemctl status apache2

# Check MySQL
systemctl status mysql
```

If either is stopped, start it:

```bash
systemctl start apache2
systemctl start mysql
```

---

**Cause B: Wrong URL in the connector configuration**

In Kibana, go to **Stack Management → Connectors** and open your osTicket connector. Verify the URL matches this exact format:

```
http://<osTicket-server-IP>/osticket/api/tickets.json
```

Common mistakes:
- Using `https://` when osTicket is running plain HTTP
- Missing `/osticket/` in the path
- Missing `/api/tickets.json` at the end
- Using a hostname instead of an IP address that isn't resolving

---

**Cause C: Wrong or missing API key**

The osTicket API key in the connector must match the key you created in osTicket and must be associated with the correct IP address.

In your osTicket admin panel:

1. Go to **Admin Panel → Manage → API Keys**.
2. Verify the API key exists and is **active**.
3. Verify the **IP address** listed for the key matches the IP address of your **ELK server** (the source of the webhook request).
   > If the IP doesn't match, osTicket will silently reject the request. Update the IP to match your ELK server's IP.
4. Copy the full API key string and paste it into the Kibana connector's **API Key** field — there should be no leading or trailing spaces.

---

**Cause D: Firewall is blocking port 80 between ELK and osTicket servers**

The ELK server needs to reach the osTicket server on port **80** (HTTP). Add a firewall rule on the osTicket server allowing inbound TCP port 80 from the ELK server IP (or from `0.0.0.0/0` for the lab).

Also verify the osTicket server's firewall group in Vultr includes this rule.

---

**Cause E: osTicket `ost-config.php` is still in setup mode or has wrong database credentials**

If osTicket was recently installed and the setup directory was not removed, or if the database credentials in `ost-config.php` are incorrect, API requests will fail. Verify the configuration:

```bash
cat /var/www/html/osticket/bootstrap.php
# or
cat /var/www/html/osticket/include/ost-config.php
```

Confirm `DBHOST`, `DBNAME`, `DBUSER`, and `DBPASS` match your MySQL setup.

---

### ❌ Error 6 — Error Calling Webhook: Request Failed
> **Video timestamp: 17:46**

#### Symptom
The osTicket connector tests successfully, but when an Elastic alert fires, the action history shows:

```
Error calling webhook: request failed
```

Tickets are not being created in osTicket.

#### Causes and Fixes

**Cause A: Malformed XML in the request body**

The osTicket API expects a very specific XML format. Any syntax error — a missing closing tag, an incorrect attribute, or invalid characters in the body — will cause the webhook to fail silently or return an error.

Verify your action body in Kibana matches this template exactly:

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

Common XML mistakes:
- Forgetting the `<?xml version...?>` declaration on the first line
- Using double curly braces `{{` and `}}` for variables — this is correct, do not change these
- Missing the `<![CDATA[...]]>` wrapper around the message body
- Unclosed tags

---

**Cause B: Missing or incorrect `Content-Type` header**

The Kibana webhook connector must send a `Content-Type: application/xml` (or `text/xml`) header. In your connector settings, verify the header is set:

| Header | Value |
|--------|-------|
| `Content-Type` | `application/xml` |

If this header is missing, osTicket may reject the body as unrecognizable.

---

**Cause C: `server.publicBaseUrl` is not set in Kibana**

The `{{rule.url}}` variable in the ticket body will be empty if `server.publicBaseUrl` is not configured in `kibana.yml`. While this doesn't directly cause the webhook to fail, it can cause the entire body to be malformed in some configurations.

Ensure this is set in `/etc/kibana/kibana.yml`:

```yaml
server.publicBaseUrl: "http://<ELK-server-IP>:5601"
```

Restart Kibana after making changes:

```bash
systemctl restart kibana.service
```

---

**Cause D: Rule action frequency is misconfigured**

If the action frequency is set to **Per alert** instead of **Per rule run**, the action may throttle or batch in unexpected ways. For this challenge, the recommended setting is:

- Go to the detection rule → **Edit rule settings → Actions tab**
- Set **Action frequency** to: `Per rule run`

This ensures a webhook fires every time the rule evaluates and finds a match.

---

**Cause E: osTicket API key IP restriction is rejecting requests**

Even if the connector test passes from the Kibana UI, the actual alert-triggered webhook fires from the Kibana server process on the ELK server backend. If the osTicket API key is restricted to a specific IP, verify that IP matches the ELK server's address precisely — including whether it's a public or private IP depending on how your network is set up.

---

### 🔧 Quick-Reference Diagnostic Commands

Use these commands on the appropriate servers when troubleshooting:

**ELK Server:**
```bash
systemctl status elasticsearch.service
systemctl status kibana.service
journalctl -u kibana.service -n 50       # Last 50 Kibana log lines
journalctl -u elasticsearch.service -n 50
tail -f /var/log/kibana/kibana.log        # Live Kibana logs
```

**Fleet Server:**
```bash
systemctl status elastic-agent.service
journalctl -u elastic-agent.service -n 50
tail -f /var/log/elastic-agent/elastic-agent.log
```

**Windows Server (PowerShell, as Administrator):**
```powershell
Get-Service elastic-agent                              # Check agent service
Restart-Service elastic-agent                         # Restart agent
Get-Content "C:\Program Files\Elastic\Agent\logs\elastic-agent-*.log" -Tail 50
```

**osTicket Server:**
```bash
systemctl status apache2
systemctl status mysql
tail -f /var/log/apache2/error.log       # Web server errors
tail -f /var/log/apache2/access.log      # Incoming requests (look for POST to /api/tickets.json)
```

---

### 🔑 Port Reference Card

Keep this handy when diagnosing connectivity issues between components:

| Source | Destination | Port | Protocol | Purpose |
|--------|-------------|------|----------|---------|
| Browser | ELK Server | 5601 | TCP/HTTP | Kibana web UI |
| ELK Server | ELK Server | 9200 | TCP/HTTP | Elasticsearch API (internal) |
| Fleet Server | ELK Server | 9200 | TCP/HTTPS | Fleet → Elasticsearch |
| Any Agent | Fleet Server | 8220 | TCP/HTTPS | Agent enrollment & check-in |
| ELK Server | osTicket Server | 80 | TCP/HTTP | Webhook API calls |
| Analyst Laptop | osTicket Server | 80 | TCP/HTTP | osTicket web UI |
| Windows Server | Mythic C2 | 9999 / 80 | TCP | C2 agent beacon |

---

### ✅ Summary of Errors and Fixes

| Error | Root Cause | Fix |
|-------|-----------|-----|
| Kibana connection error | Service down, firewall blocking 5601, wrong bind address | Start service, open firewall, set `server.host: 0.0.0.0` |
| Fleet Server timed out | Firewall blocking 9200 or 8220, wrong Fleet URL | Open ports, verify Fleet URL in Kibana settings |
| Fail to execute request to Fleet | Network blocked, Fleet service down, wrong IP/port in enroll command | Test connectivity, start Fleet service, check `--url` flag |
| x509 certificate unknown authority | Self-signed cert not trusted by enrolling agent | Add `--insecure` flag to enroll/install command |
| osTicket cannot connect — invalid settings | Service down, wrong URL, wrong API key, firewall blocking port 80 | Start Apache/MySQL, fix URL, verify API key and IP |
| Webhook request failed | Malformed XML body, wrong Content-Type header, misconfigured action frequency | Fix XML template, add Content-Type header, set frequency to "Per rule run" |

---

## 🎓 Challenge Complete — What You've Built

Over 30 days, you have deployed and configured a fully operational mini SOC environment:

```
┌─────────────────────────────────────────────────────────────┐
│                      VULTR VPC                              │
│                                                             │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐  │
│  │  ELK Server  │◄──►│ Fleet Server │◄──►│Windows Server│  │
│  │ Elastic +    │    │              │    │ (RDP + Sysmon│  │
│  │   Kibana     │    │              │    │ + Elastic    │  │
│  │   + SIEM     │    │              │    │   Defend)    │  │
│  └──────┬───────┘    └──────────────┘    └──────────────┘  │
│         │                                ┌──────────────┐  │
│         │                                │Ubuntu Server │  │
│         │                                │  (SSH +      │  │
│         │                                │ Elastic Agt) │  │
│         │                                └──────────────┘  │
└─────────┼───────────────────────────────────────────────────┘
          │
          ▼ (outside VPC)
   ┌──────────────┐    ┌──────────────┐
   │  osTicket    │    │  Mythic C2   │
   │   Server     │    │    Server    │
   └──────────────┘    └──────────────┘
```

**Skills and tools you've practiced:**
- Deploying and configuring the full ELK Stack (Elasticsearch, Kibana, Logstash)
- Installing and managing Elastic Agents via Fleet Server
- Ingesting Windows Event Logs, Sysmon telemetry, and Linux auth logs
- Writing KQL queries to hunt for threats
- Building Kibana dashboards for SSH/RDP brute force visualization
- Creating and tuning custom SIEM detection rules
- Investigating SSH brute force, RDP brute force, and Mythic C2 attacks
- Using Sysmon Process GUIDs to trace attack chains
- Enriching alerts with AbuseIPDB and GreyNoise threat intelligence
- Configuring osTicket and automating ticket creation from Elastic alerts
- Deploying Elastic Defend EDR and automating host isolation responses

---

## 🔗 Resources

- 📖 [Elastic Fleet Troubleshooting — Official Docs](https://www.elastic.co/docs/troubleshoot/ingest/fleet/common-problems)
- 📖 [Elastic Agent x509 Certificate Error](https://www.elastic.co/docs/troubleshoot/ingest/fleet/common-problems#agent-enrollment-x509)
- 📖 [osTicket API Documentation](https://github.com/osTicket/osTicket/blob/develop/setup/doc/api/tickets.md)
- 📖 [Kibana Configuration Reference](https://www.elastic.co/guide/en/kibana/current/settings.html)
---

> *Part of the MyDFIR 30-Day SOC Analyst Challenge. This is Day 30 — the final day. Well done for completing the challenge!*

