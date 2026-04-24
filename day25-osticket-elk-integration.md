# Day 25: osTicket + ELK Integration
> **MyDFIR 30-Day SOC Analyst Challenge**  
> Based on: [YouTube Tutorial — Day 25](https://www.youtube.com/watch?v=P9YxutqWAF0&list=PLG6KGSNK4PuBb0OjyDIdACZnb8AoNBeq6&index=26)

---

## 🎯 Objective

Integrate your osTicket ticketing system with the ELK Stack (Elasticsearch + Kibana) so that **security alerts automatically create support tickets** — enabling structured incident tracking and response.

---

## 📋 Prerequisites

Before starting, ensure you have:

- ✅ A running **osTicket** instance (Windows Server with XAMPP, or Ubuntu)
- ✅ A configured **ELK Stack** (Elasticsearch + Kibana) — accessible via web UI on port `5601`
- ✅ Both servers hosted in the **same VPC / private network** (e.g., Vultr VPC 2.0)
- ✅ Basic networking knowledge (IP addressing, firewall rules)
- ✅ An active **Kibana 30-day trial** (required for Webhook connectors)

### Architecture Overview

```
┌────────────────────────────────────────────────┐
│                  Vultr VPC                     │
│                                                │
│   ┌─────────────────┐    Webhook (HTTP POST)   │
│   │  Elastic/Kibana │ ──────────────────────► │
│   │    Server       │                          │
│   └─────────────────┘    ┌──────────────────┐  │
│                          │  osTicket Server  │  │
│                          │  (XAMPP/Windows)  │  │
│                          └──────────────────┘  │
└────────────────────────────────────────────────┘
```

> **Key point:** Both servers must be on the same private network. The osTicket API key is tied to the **private IP** of the ELK server within the VPC.

---

## Part 1 — Generate an API Key in osTicket

The osTicket API key authorizes Elastic to create tickets on your behalf.

### Steps

1. Log in to your **osTicket Staff Control Panel**  
   `http://<osticket-public-ip>/scp/`

2. In the top-right corner, switch to **Admin Panel**

   ![Admin Panel toggle](https://i.imgur.com/placeholder-admin-panel.png)
   > *Toggle between Staff and Admin panel using the link in the top navigation bar.*

3. Navigate to:  
   **Manage → API → Add New API Key**

4. Fill in the fields:

   | Field | Value |
   |-------|-------|
   | **IP Address** | Private IP of your ELK server (e.g., `172.16.0.x`) |
   | **Can Create Tickets** | ✅ Checked |
   | **Internal Notes** | e.g., `ELK Stack Integration Key` |

   > ⚠️ Use the **private/VPC IP** of your ELK server — not the public IP. Both servers must be on the same VPC subnet (e.g., `172.16.0.0/24`).

5. Click **Add Key**

6. **Copy and save the generated API key** — you will need it in Part 2.

   ```
   Example key (yours will differ):
   a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4
   ```

---

## Part 2 — Create a Webhook Connector in Kibana

Kibana's connector sends an HTTP POST to osTicket whenever an alert fires.

### Activate the 30-Day Trial (if not done yet)

1. In Kibana, open the **hamburger menu (☰)** → scroll down to **Management** → **Stack Management**
2. Under **License Management**, click **Start a 30-day trial**  
   > This unlocks the Webhook connector type.

### Create the Webhook Connector

1. In **Stack Management**, under **Alerts and Insights**, click **Connectors**

2. Click **Create connector**

3. Select **Webhook**

4. Configure the connector:

   | Setting | Value |
   |---------|-------|
   | **Connector Name** | `osTicket` (or any name you prefer) |
   | **Method** | `POST` |
   | **URL** | `http://<osticket-private-ip>/api/tickets.json` |
   | **Authentication** | None |
   | **Add HTTP header** | `X-API-Key` : `<your-osticket-api-key>` |
   | **Content-Type header** | `application/json` |

   > ⚠️ Use the **private IP** of your osTicket server for the URL (since both servers are on the same VPC).

   **Headers summary:**
   ```
   X-API-Key: a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4
   Content-Type: application/json
   ```

5. Click **Save & Test** — do NOT run the test yet; first configure the body.

---

## Part 3 — Configure the Ticket Body (JSON Payload)

The webhook body tells osTicket what content to put in the ticket. This uses osTicket's JSON API format.

### Reference

The full osTicket API documentation (including payload examples) is available here:  
📄 [osTicket Tickets API — GitHub](https://github.com/osTicket/osTicket/blob/develop/setup/doc/api/tickets.md)

### JSON Payload (Basic Test)

Paste the following into the **Body** field of the webhook connector:

```json
{
    "alert": true,
    "autorespond": true,
    "source": "API",
    "name": "Angry User",
    "email": "api@osticket.com",
    "phone": "3185558634X123",
    "subject": "Testing API",
    "ip": "123.211.233.122",
    "message": "data:text/html,MESSAGE <b>HERE</b>"
}
```

> This is taken directly from the [osTicket API docs](https://github.com/osTicket/osTicket/blob/develop/setup/doc/api/tickets.md). It's a static test payload — you'll enrich it with dynamic alert data in Part 4.

### Enriched Payload (with Kibana Alert Variables)

Once the basic test works, replace the body with a dynamic version that pulls real alert context:

```json
{
    "alert": true,
    "autorespond": true,
    "source": "API",
    "name": "Kibana Alert",
    "email": "soc@yourdomain.com",
    "subject": "{{rule.name}}",
    "ip": "{{context.host.ip}}",
    "message": "data:text/html,<b>Alert:</b> {{rule.name}}<br><b>Severity:</b> {{context.rule.severity}}<br><b>Host:</b> {{context.host.name}}<br><b>Source IP:</b> {{context.source.ip}}<br><b>Time:</b> {{context.date}}<br><b>Rule ID:</b> {{rule.id}}"
}
```

> **Tip:** Kibana uses Mustache `{{variable}}` syntax for dynamic alert fields. Available variables depend on your alert rule type.

---

## Part 4 — Test the Integration

### Run a Manual Test

1. Back in the Webhook connector page, click **Run**
2. The body will be sent to osTicket
3. If successful, you'll see:  
   ```
   HTTP 201 Created
   ```

### Verify in osTicket

1. Go to your osTicket **Staff Control Panel**
2. Click **Admin Panel** → check the **Tickets** tab
3. You should see a new ticket created — e.g., **"Testing API"** from *Angry User*

   > ✅ If the ticket appears, the integration is working!

### Troubleshooting Common Issues

| Problem | Fix |
|---------|-----|
| `401 Unauthorized` | Check that the `X-API-Key` header value matches your osTicket API key exactly |
| `Connection refused` | Verify the osTicket server IP/URL is correct and accessible from ELK; check firewall rules |
| No ticket created | Restart XAMPP on the osTicket server, then re-run the test |
| `400 Bad Request` | Validate your JSON payload formatting (no trailing commas, valid JSON) |
| Wrong IP in API key | The osTicket API key is IP-locked — make sure you used the ELK server's **private** IP |

**Restart procedure (if tickets stop generating):**
1. RDP into your osTicket Windows server
2. Open XAMPP Control Panel → Stop & Start **Apache** and **MySQL**
3. Log back in to osTicket
4. Re-run the Kibana connector test

---

## Part 5 — Attach the Connector to an Alert Rule

Once tested, wire the connector to a real alert so tickets are created automatically.

1. In Kibana, go to **Security → Rules** (or **Alerts → Manage Rules**)
2. Open an existing rule (e.g., your SSH brute force or RDP brute force rule)
3. Edit the rule → scroll to **Actions**
4. Click **Add action** → select your `osTicket` webhook connector
5. Paste the enriched JSON payload (from Part 3) into the body
6. Set the action to trigger **On each alert** or **On new alert**
7. **Save** the rule

Now, whenever the rule fires, a ticket is automatically created in osTicket!

---

## 🔍 What the Ticket Looks Like

After a successful alert-to-ticket flow, osTicket will show something like:

```
Ticket #: 123456
Subject:  SSH Brute Force Detected
From:     Kibana Alert <soc@yourdomain.com>
Status:   Open
Priority: Normal

Alert: SSH Brute Force Detected
Severity: High
Host: MYDFIR-Linux-Server
Source IP: 185.220.101.5
Time: 2024-09-24T18:45:00Z
Rule ID: abc123-...
```

> **Best practice:** Once you open a ticket, assign it to yourself to prevent duplicate investigation. Close the ticket with documented notes once resolved.

---

## ✅ Summary

| Step | Action | Result |
|------|--------|--------|
| 1 | Generate API key in osTicket | ELK is authorized to create tickets |
| 2 | Create Webhook connector in Kibana | Kibana can POST to osTicket |
| 3 | Configure JSON body payload | Ticket content is defined |
| 4 | Test the connector | Verify ticket appears in osTicket |
| 5 | Attach to alert rule | Alerts auto-generate tickets going forward |

---

## 📚 References

- 📺 [MyDFIR YouTube — Day 25](https://www.youtube.com/watch?v=P9YxutqWAF0&list=PLG6KGSNK4PuBb0OjyDIdACZnb8AoNBeq6&index=26)
- 📄 [osTicket Tickets API Documentation](https://github.com/osTicket/osTicket/blob/develop/setup/doc/api/tickets.md)
- 🌐 [MyDFIR Website](https://www.mydfir.com)
- ☁️ [Vultr Free $300 Credit](https://www.vultr.com/?ref=9632889-9J) *(affiliate, new accounts only)*

---

*Part of the [MyDFIR 30-Day SOC Analyst Challenge](https://www.youtube.com/playlist?list=PLG6KGSNK4PuBb0OjyDIdACZnb8AoNBeq6)*
