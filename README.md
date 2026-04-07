# SIEM-Lab-using-ElasticSearch-and-Kibana
Elasticsearch is a distributed, open-source search and analytics engine designed for speed, scalability, and near real-time data processing. Built on Apache Lucene, it acts as a NoSQL, JSON-based document store used for full-text search, log analysis, and data visualization, often as the core component of the ELK Stack.
# Objective
I have conducted a hands-on lab that goes over installing servers, including them on a VPN network, integrating them with Elastic search to be able to be used as a SIEM, system hardening, replicating an attack on both the attacker side and on the defence side withing the SIEM Environment. I will be setting up and configuring a VPC consisting of different servers and firewalls using VULTR. Below is a diagram of the project scope.

# Skills gained across the lab

-Designing SOC network architecture (logical diagrams, VPCs, segmentation)

-SIEM setup and management with the ELK stack

-Writing detection rules and KQL queries

-Building security dashboards and visualizations

-Simulating and detecting brute force attacks (SSH/RDP)

-Understanding attacker tactics using a real C2 framework (Mythic)

-Incident response and ticket-driven case management with osTicket

-Practical cloud skills (Vultr, VM management, firewall configuration)

# Tools used throughout the lab

**SIEM & Log Management**

-Elasticsearch: storing and querying security logs

-Logstash: ingesting and processing log data

-Kibana: visualizing logs and building dashboards

-Sysmon: capturing endpoint events (process creation, network connections)

-Elastic Agent & Fleet Server: centralized agent management across hosts

**Cloud & Network Infrastructure**

-Vultr: cloud platform for hosting VMs

-Virtual Private Cloud (VPC): isolated, secure network environment

-Firewall rules: restricting access by IP and port

-SSH: secure remote access to Linux servers

**Threat Detection & Alerting**

-KQL (Kibana Query Language): filtering and searching logs

-Windows Event IDs: identifying specific security events (e.g. 4625 for failed logins)

-Custom alert rules: triggering notifications on brute force thresholds

-RDP & SSH monitoring: detecting failed and successful login attempts

**Attack Simulation (Red Team)**

-Kali Linux: attacker machine for running offensive tools

-Mythic C2 framework: command and control server for simulating real attacks

-Payload generation & callbacks: simulating malware delivery and exfiltration

**Incident Response & Case Management**

-osTicket: open-source ticketing system integrated with Elastic alerts

-Alert triage workflow: investigating, prioritizing, and resolving security incidents

-Logical network diagrams: documenting SOC lab architecture with draw.io

**Supporting Skills**

-Windows Server (RDP enabled) & Ubuntu Server (SSH enabled): target machines for attack simulation

-NTLM authentication concepts: understanding alternative attack vectors beyond brute force

-Threat lifecycle tracing: following an attack from initial access to post-exploitation in logs

# Steps Taken

**Building Ubuntu Server from VULTR (Virtual Network)**

  -Vultr: VPS Server and Network; I will start by installing an Ubuntu Server from VULTR and applying Elastic Search onto it.
    -Vultr is a Global, automated cloud infrastructure from the broadest array of AMD and NVIDIA GPUs to virtual CPUs, bare metal, Kubernetes, storage, and networking.
    -After making an account with VULTR, you will need to setup a VPN network, add a Server computer, and configure those together with a static IP, storage information, and applying an Ubuntu Server.
    -You then Deploy and it will start building the Ubuntu Server
    <img width="920" height="271" alt="Screenshot 2026-04-07 104456" src="https://github.com/user-attachments/assets/8fcde44c-a7a3-4550-8c38-2cef1775c4b6" />

  -Installing Elasticsearch on Ubuntu Server
    -We will be installing ElasticSearch via Powershell and SSH onto our hosted virtual machine from VULTR.
    -Elasticsearch is a distributed, open-source search and analytics engine designed for speed, scalability, and near real-time data processing. Built on Apache Lucene, it acts as a NoSQL, JSON-based document store used       for full-text search, log analysis, and data visualization, often as the core component of the ELK Stack.
    <img width="976" height="468" alt="Screenshot 2026-04-07 104717" src="https://github.com/user-attachments/assets/75edae45-1362-4a5d-a6aa-0734b4437ff2" />

  -Running SSH to our Hosted IP Address
    -We first wanna start by pointing our directory in Powershell over to our hosted Machine and it’s IP address.
    -That can be found in the VULTUR environment under your dashboard.
    <img width="911" height="58" alt="Screenshot 2026-04-07 104912" src="https://github.com/user-attachments/assets/575d1153-1863-473e-b640-79f1fd867929" />
    -Next, we want to go get the download from ElasticSearch, copy the url from the download, and paste into Powershell using wget command to start the download on the actual hosted machine. (doing this for deb x86_64)
    <img width="729" height="457" alt="Screenshot 2026-04-07 105037" src="https://github.com/user-attachments/assets/11e0068b-3d26-4039-9e09-35f5317ac260" />
    <img width="773" height="87" alt="Screenshot 2026-04-07 105120" src="https://github.com/user-attachments/assets/351790a6-3263-442a-95d3-ebf67940be6e" />
