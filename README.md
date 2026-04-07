# SIEM-Lab-using-ElasticSearch-and-Kibana
Elasticsearch is a distributed, open-source search and analytics engine designed for speed, scalability, and near real-time data processing. Built on Apache Lucene, it acts as a NoSQL, JSON-based document store used for full-text search, log analysis, and data visualization, often as the core component of the ELK Stack.
# Objective
I have conducted a hands-on lab that goes over installing servers, including them on a VPN network, integrating them with Elastic search to be able to be used as a SIEM, system hardening, replicating an attack on both the attacker side and on the defence side withing the SIEM Environment. Below is a diagram of the project scope.

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
