# 🎫 osTicket Setup Walkthrough
> **Based on:** [MyDFIR SOC Analyst Challenge – Day 24](https://www.youtube.com/watch?v=xgxQuLL33oU)  
> **Goal:** Deploy and configure osTicket, an open-source ticketing system, on a cloud-hosted Windows Server using XAMPP.

---

## 📋 Table of Contents
1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Step 1 – Deploy a Windows Server on Vultr](#step-1--deploy-a-windows-server-on-vultr)
4. [Step 2 – Connect via RDP & Install XAMPP](#step-2--connect-via-rdp--install-xampp)
5. [Step 3 – Configure Firewall Rules](#step-3--configure-firewall-rules)
6. [Step 4 – Configure phpMyAdmin](#step-4--configure-phpmyadmin)
7. [Step 5 – Edit XAMPP Config Files](#step-5--edit-xampp-config-files)
8. [Step 6 – Create the osTicket Database](#step-6--create-the-osticket-database)
9. [Step 7 – Download & Deploy osTicket](#step-7--download--deploy-osticket)
10. [Step 8 – Run the osTicket Installer](#step-8--run-the-osticket-installer)
11. [Step 9 – Log In & Verify](#step-9--log-in--verify)
12. [Troubleshooting](#troubleshooting)
13. [Next Steps](#next-steps)

---

## Overview

osTicket is an open-source help desk and ticketing system used by SOC teams to:
- **Log and track** security incidents
- **Route tickets** to the right analyst
- **Prioritize** alerts by severity

In this lab, we spin up a fresh Windows Server in Vultr, install the XAMPP web stack, and host osTicket on it so it can later be integrated with Elastic/Kibana alerts.

---

## Prerequisites

| Requirement | Details |
|---|---|
| Cloud Account | [Vultr](https://www.vultr.com) (new accounts get $300 free credit) |
| VPC Network | Same VPC used for your ELK stack from earlier challenge days |
| Firewall Group | `MYDFIR-30-Day-Challenge` group already created |
| RDP Client | Built-in Windows Remote Desktop or any RDP app |

---

## Step 1 – Deploy a Windows Server on Vultr

1. Log in to your **Vultr** dashboard and click **Deploy New Server**.

   ![New Server](https://deepalibolsekar.github.io/assets/OsTicket/new-server.png)

2. Select **Shared CPU** and pick the **same region** as your other servers (e.g., your ELK server).

   ![Shared CPU](https://deepalibolsekar.github.io/assets/OsTicket/shared-cpu.png)
   ![Location](https://deepalibolsekar.github.io/assets/OsTicket/location.png)

3. For the OS image, choose **Windows Standard Server**.

   ![Windows Image](https://deepalibolsekar.github.io/assets/OsTicket/win-img.png)

4. Select the plan: **1 vCPU / 2 GB RAM** is sufficient.

   ![Plan](https://deepalibolsekar.github.io/assets/OsTicket/plan.png)

5. Under **Additional Features**, enable **VPC 2.0** and attach it to the same private network as your ELK server.

   ![VPC](https://deepalibolsekar.github.io/assets/OsTicket/vpc.png)

6. Give the instance a descriptive hostname (e.g., `MYDFIR-osTicket-Server`) and click **Deploy Now**.

   ![Hostname](https://deepalibolsekar.github.io/assets/OsTicket/hostname.png)

7. Once deployed, go to **Firewall** settings and attach the `MYDFIR-30-Day-Challenge` firewall group to this server.

> ⚠️ **Note:** Wait until the server status shows **Running** before attempting to connect.

---

## Step 2 – Connect via RDP & Install XAMPP

1. From the Vultr dashboard, copy the server's **public IP** and the auto-generated **Administrator password**.
2. Open **Remote Desktop Connection** on your local machine, enter the IP, and log in.
3. Inside the server, open a browser and download **XAMPP 8.2.x** from:  
   👉 [https://www.apachefriends.org/download.html](https://www.apachefriends.org/download.html)

   ![XAMPP Download](https://deepalibolsekar.github.io/assets/OsTicket/xampp-download.png)

4. Run the installer. If a UAC or antivirus warning appears, click **OK** to continue.

   ![Warning OK](https://deepalibolsekar.github.io/assets/OsTicket/warning-ok.png)

5. Proceed through the installer with default settings and let it complete.

   ![Install XAMPP](https://deepalibolsekar.github.io/assets/OsTicket/install-xampp.png)

6. After installation, the **XAMPP Control Panel** will launch. Click **Start** next to both **Apache** and **MySQL**.

   ![Start XAMPP](https://deepalibolsekar.github.io/assets/OsTicket/start-xampp.png)

---

## Step 3 – Configure Firewall Rules

While XAMPP installs, set up Windows Firewall rules so the osTicket web interface is reachable.

1. Open **Windows Defender Firewall with Advanced Security**.
2. Click **Inbound Rules → New Rule**.
3. Select **Port**, then click Next.
4. Choose **TCP** and enter `80, 443` as the specific local ports.
5. Allow the connection, apply to all profiles, and name the rule (e.g., `XAMPP HTTP/HTTPS`).

   ![Create Firewall Rule](https://deepalibolsekar.github.io/assets/OsTicket/create-new-rule.png)

> 💡 Port **80** is used for HTTP and port **443** for HTTPS — both needed by Apache/XAMPP.

---

## Step 4 – Configure phpMyAdmin

By default, phpMyAdmin only accepts connections from `localhost`. We need to allow your public IP.

1. In the XAMPP Control Panel, click the **Admin** button next to Apache. This opens the XAMPP dashboard in a browser.
2. Click **phpMyAdmin** from the top menu.

   ![phpMyAdmin](https://deepalibolsekar.github.io/assets/OsTicket/phpmyadmin.png)

3. Navigate to **User Accounts**. Find the `root` user entry and click **Edit Privileges**.

   ![User Accounts](https://deepalibolsekar.github.io/assets/OsTicket/user-acc.png)

4. Update the **hostname** from `localhost` to your server's **public IP address**. Set a strong password and save.

   ![Root Account Created](https://deepalibolsekar.github.io/assets/OsTicket/root-acc-created.png)

5. Repeat the same process for the **pma** (phpMyAdmin) user account — update hostname to the server IP and set a password.

   ![PMA Account](https://deepalibolsekar.github.io/assets/OsTicket/pm-acc-created.png)

   ![Accounts Modified](https://deepalibolsekar.github.io/assets/OsTicket/acc-modified.png)

---

## Step 5 – Edit XAMPP Config Files

Two config files need to be updated to reflect the server's IP address instead of `localhost`.

### 5a – `httpd-xampp.conf` (Apache domain name)

1. Navigate to: `C:\xampp\apache\conf\extra\`
2. Open `httpd-xampp.conf` in Notepad (or Notepad++).
3. Find the line containing `apache_domainname` and replace `localhost` with your server's public IP.
4. Save and close.

   ![Properties Config](https://deepalibolsekar.github.io/assets/OsTicket/properties.png)

### 5b – `config.inc.php` (phpMyAdmin connection settings)

1. Navigate to: `C:\xampp\phpMyAdmin\`
2. Open `config.inc.php` in a text editor.
3. Update the following values:
   - `$cfg['Servers'][$i]['host']` → your server's **public IP**
   - `$cfg['Servers'][$i]['password']` for both `root` and `controlpass` → the passwords you just set
4. Save and close.

   ![Config INC](https://deepalibolsekar.github.io/assets/OsTicket/config.png)

> 🔄 After saving both files, **restart Apache** from the XAMPP Control Panel for changes to take effect.

---

## Step 6 – Create the osTicket Database

1. Open your browser and navigate to: `http://<YOUR-SERVER-IP>/phpmyadmin`
2. Log in with the `root` credentials you configured.
3. Click **New** in the left sidebar and create a database named `osticket`.

   ![Create Database](https://deepalibolsekar.github.io/assets/OsTicket/db-create.png)

4. With `osticket` selected, go to **Privileges → Add user account** (or grant the `root` user full privileges on this database).

---

## Step 7 – Download & Deploy osTicket

1. On the server, open a browser and go to the official osTicket download page:  
   👉 [https://osticket.com/download/](https://osticket.com/download/)

   ![osTicket Download](https://deepalibolsekar.github.io/assets/OsTicket/open-src-download.png)

2. Select **Self-Hosted** as the deployment type.

   ![Self Hosted](https://deepalibolsekar.github.io/assets/OsTicket/self-hosted.png)

3. Pick the latest stable release.

   ![Release](https://deepalibolsekar.github.io/assets/OsTicket/release.png)

4. Choose your preferred **language** (English is fine). Skip the **Plugins** step.

   ![Language](https://deepalibolsekar.github.io/assets/OsTicket/lang.png)

5. Click through any newsletter prompt (you can decline) and download the ZIP file.

6. **Extract** the ZIP file contents.

   ![Extract osTicket](https://deepalibolsekar.github.io/assets/OsTicket/extract-osticket.png)

7. Copy the **`scripts`** and **`upload`** folders from the extracted archive into a new folder:
   ```
   C:\xampp\htdocs\osticket\
   ```

   ![Copy to htdocs](https://deepalibolsekar.github.io/assets/OsTicket/copy-to-htdocs.png)

---

## Step 8 – Run the osTicket Installer

1. Open a browser and navigate to:
   ```
   http://<YOUR-SERVER-IP>/osticket/upload
   ```
   You should see the osTicket prerequisites/installer page.

   ![osTicket Installer](https://deepalibolsekar.github.io/assets/OsTicket/osticket-installer.png)

2. Before clicking **Continue**, rename the sample config file:
   - Navigate to: `C:\xampp\htdocs\osticket\upload\include\`
   - Rename `ost-sampleconfig.php` → `ost-config.php`

   ![OST Config](https://deepalibolsekar.github.io/assets/OsTicket/ost-config.png)

3. Return to the browser and click **Continue**. Fill in the installation form:

   | Field | Value |
   |---|---|
   | Helpdesk Name | Your chosen name (e.g., `MyDFIR SOC`) |
   | Default Email | An admin email address |
   | Admin Username | Your desired admin login |
   | Admin Password | A strong password (save this!) |
   | Database Name | `osticket` |
   | Database User | `root` |
   | Database Password | The root password you set earlier |
   | **Database Host** | Your server's **public IP** (not `localhost`!) |

   ![Install Fields](https://deepalibolsekar.github.io/assets/OsTicket/instal-fields-1.png)

4. Click **Install Now**.

   ![Install Now](https://deepalibolsekar.github.io/assets/OsTicket/install-now.png)

5. The installer will run — give it a moment.

   ![Doing Stuff](https://deepalibolsekar.github.io/assets/OsTicket/doing-stuff-page.png)

6. On the final page, follow the instructions to set `ost-config.php` to **read-only**:
   - Right-click `ost-config.php` → Properties → Security → Advanced
   - Set **Everyone** to **Read** only.

   ![File Permissions](https://deepalibolsekar.github.io/assets/OsTicket/file-perm.png)

   ![Success](https://deepalibolsekar.github.io/assets/OsTicket/success.png)

> 🎉 **Installation complete!**

7. **Bookmark** the two important URLs shown on the success page:

   | URL | Purpose |
   |---|---|
   | `http://<IP>/osticket/upload/` | End-user ticket portal |
   | `http://<IP>/osticket/upload/scp/` | Staff / Admin login panel |

   ![Staff Control Panel Link](https://deepalibolsekar.github.io/assets/OsTicket/copy-staff-contrl-panel.png)

---

## Step 9 – Log In & Verify

1. Navigate to the **Staff Control Panel**:
   ```
   http://<YOUR-SERVER-IP>/osticket/upload/scp/
   ```
2. Log in with the **admin credentials** you set during installation.

   ![Staff Login](https://deepalibolsekar.github.io/assets/OsTicket/staff-login.png)

3. You should now see the osTicket **Admin Panel** — your ticketing system is live!

   ![Admin Panel](https://deepalibolsekar.github.io/assets/OsTicket/admin-panel.png)

From here you can:
- Create and manage **agents**
- Configure **ticket categories** and **SLA plans**
- View and respond to incoming tickets

---

## Troubleshooting

| Problem | Likely Cause | Fix |
|---|---|---|
| Can't access phpMyAdmin from browser | Firewall blocking port 80 | Double-check Windows Firewall inbound rule for port 80 |
| Installer fails / DB connection error | Wrong DB host (`localhost` instead of IP) | Use the server's **public IP** in the DB Host field |
| `root` account not allowing remote login | phpMyAdmin user hostname still set to `localhost` | Re-edit the `root` user account hostname in phpMyAdmin |
| `ost-config.php` not found error | File not renamed yet | Rename `ost-sampleconfig.php` → `ost-config.php` |
| Blank page at `/osticket/upload` | Files in wrong directory | Ensure `scripts/` and `upload/` are directly inside `C:\xampp\htdocs\osticket\` |

---

## Next Steps

With osTicket running, the next step in the MyDFIR 30-Day SOC Challenge is **Day 25: Integrating osTicket with Elastic/Kibana**, where you'll:
- Generate an **API key** inside osTicket
- Configure an **Elastic webhook connector** to automatically create tickets from SIEM alerts
- Test end-to-end ticket creation from a brute-force detection rule

---

## Resources

- 📺 [Original YouTube Tutorial – MyDFIR Day 24](https://www.youtube.com/watch?v=xgxQuLL33oU)
- 🌐 [osTicket Official Site](https://osticket.com)
- 📦 [XAMPP Download](https://www.apachefriends.org/download.html)
- ☁️ [Vultr Free Credits ($300)](https://www.vultr.com/?ref=9632889-9J)
- 📖 [osTicket API Docs (for Day 25)](https://github.com/osTicket/osTicket/blob/develop/setup/doc/api/tickets.md)

---

*Part of the [MyDFIR 30-Day SOC Analyst Challenge](https://www.youtube.com/@MyDFIR) series.*
