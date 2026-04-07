# SIEM-Lab-using-ElasticSearch-and-Kibana

Elasticsearch is a distributed, open-source search and analytics engine designed for speed, scalability, and near real-time data processing. Built on Apache Lucene, it acts as a NoSQL, JSON-based document store used for full-text search, log analysis, and data visualization, often as the core component of the ELK Stack.

## Objective

I have conducted a hands-on lab that goes over installing servers, including them on a VPN network, integrating them with Elastic search to be able to be used as a SIEM, system hardening, replicating an attack on both the attacker side and on the defence side withing the SIEM Environment. I will be setting up and configuring a VPC consisting of different servers and firewalls using VULTR. Below is a diagram of the project scope.

### Skills gained across the lab

- Designing SOC network architecture (logical diagrams, VPCs, segmentation)
- SIEM setup and management with the ELK stack
- Writing detection rules and KQL queries
- Building security dashboards and visualizations
- Simulating and detecting brute force attacks (SSH/RDP)
- Understanding attacker tactics using a real C2 framework (Mythic)
- Incident response and ticket-driven case management with osTicket
- Practical cloud skills (Vultr, VM management, firewall configuration)

### Tools used throughout the lab

**SIEM & Log Management**

- Elasticsearch: storing and querying security logs
- Logstash: ingesting and processing log data
- Kibana: visualizing logs and building dashboards
- Sysmon: capturing endpoint events (process creation, network connections)
- Elastic Agent & Fleet Server: centralized agent management across hosts

**Cloud & Network Infrastructure**

- Vultr: cloud platform for hosting VMs
- Virtual Private Cloud (VPC): isolated, secure network environment
- Firewall rules: restricting access by IP and port
- SSH: secure remote access to Linux servers

**Threat Detection & Alerting**

- KQL (Kibana Query Language): filtering and searching logs
- Windows Event IDs: identifying specific security events (e.g. 4625 for failed logins)
- Custom alert rules: triggering notifications on brute force thresholds
- RDP & SSH monitoring: detecting failed and successful login attempts

**Attack Simulation (Red Team)**

- Kali Linux: attacker machine for running offensive tools
- Mythic C2 framework: command and control server for simulating real attacks
- Payload generation & callbacks: simulating malware delivery and exfiltration

**Incident Response & Case Management**

- osTicket: open-source ticketing system integrated with Elastic alerts
- Alert triage workflow: investigating, prioritizing, and resolving security incidents
- Logical network diagrams: documenting SOC lab architecture with draw.io

**Supporting Skills**

- Windows Server (RDP enabled) & Ubuntu Server (SSH enabled): target machines for attack simulation
- NTLM authentication concepts: understanding alternative attack vectors beyond brute force
- Threat lifecycle tracing: following an attack from initial access to post-exploitation in logs

# Steps Taken

**Building Ubuntu Server from VULTR (Virtual Network)**

_Vultr: VPS Server and Network; I will start by installing an Ubuntu Server from VULTR and applying Elastic Search onto it._
- Vultr is a Global, automated cloud infrastructure from the broadest array of AMD and NVIDIA GPUs to virtual CPUs, bare metal, Kubernetes, storage, and networking.
- After making an account with VULTR, you will need to setup a VPN network, add a Server computer, and configure those together with a static IP, storage information, and applying an Ubuntu Server.
- You then Deploy and it will start building the Ubuntu Server
<img width="920" height="271" alt="Screenshot 2026-04-07 104456" src="https://github.com/user-attachments/assets/8fcde44c-a7a3-4550-8c38-2cef1775c4b6" />


**Install Elastic Search**

_Installing Elasticsearch on Ubuntu Server_
- We will be installing ElasticSearch via Powershell and SSH onto our hosted virtual machine from VULTR.
- Elasticsearch is a distributed, open-source search and analytics engine designed for speed, scalability, and near real-time data processing. Built on Apache Lucene, it acts as a NoSQL, JSON-based document store used for full-text search, log analysis, and data visualization, often as the core component of the ELK Stack.
<img width="976" height="468" alt="Screenshot 2026-04-07 104717" src="https://github.com/user-attachments/assets/75edae45-1362-4a5d-a6aa-0734b4437ff2" />

_Running SSH to our Hosted IP Address_
- We first wanna start by pointing our directory in Powershell over to our hosted Machine and it’s IP address.
- That can be found in the VULTUR environment under your dashboard.
<img width="911" height="58" alt="Screenshot 2026-04-07 104912" src="https://github.com/user-attachments/assets/575d1153-1863-473e-b640-79f1fd867929" />
- Next, we want to go get the download from ElasticSearch, copy the url from the download, and paste into Powershell using wget command to start the download on the actual hosted machine. (doing this for deb x86_64)
<img width="729" height="457" alt="Screenshot 2026-04-07 105037" src="https://github.com/user-attachments/assets/11e0068b-3d26-4039-9e09-35f5317ac260" />
<img width="773" height="87" alt="Screenshot 2026-04-07 105120" src="https://github.com/user-attachments/assets/351790a6-3263-442a-95d3-ebf67940be6e" />

_Installing and Starting Elastic Search Service_
- Once downloaded, we then run the installer, and then start the services. See image for script ran on Powershell (using ssh root@ipaddressofserver)

<img width="819" height="531" alt="Screenshot 2026-04-07 105505" src="https://github.com/user-attachments/assets/2b5ec018-6c03-4453-8ab2-79fcd0b97f9b" />

_Possible upgrade on Ubunto Server_
- An update may be needed for the Ubuntu Server
      
<img width="559" height="558" alt="Screenshot 2026-04-07 105505" src="https://github.com/user-attachments/assets/61755c07-8c1e-4f1e-a29d-dd01fe64bf9d" />

_Configuring IP address and Port for ElasticSearch_
- We run nano command to change these settings in Elasticsearch so it points to our hosted server.
    
<img width="739" height="472" alt="Screenshot 2026-04-07 105912" src="https://github.com/user-attachments/assets/235c9550-a6ca-4570-a161-da13e884713a" />

<img width="645" height="523" alt="Screenshot 2026-04-07 105956" src="https://github.com/user-attachments/assets/47113c19-2ecc-42d3-98ba-3e1469f46696" />

_Add a Firewall Group in VULTR_
- We then have to add a Firewall Group for our server to allow access for ElasticSearch
<img width="895" height="287" alt="Screenshot 2026-04-07 110103" src="https://github.com/user-attachments/assets/17dc0726-1949-4768-b4e7-e6c06dda2c7c" />

**_Installing Kibana and setting up with Elasticsearch_**
- We will be installing Kibana via Powershell and SSH onto our hosted virtual machine from VULTR.
- Kibana is an open-source data visualization and exploration tool designed for the Elastic Stack (ELK Stack). 
- It acts as the user interface to search, analyze, and visualize data stored in Elasticsearch, allowing users to create interactive dashboards, charts, and maps to analyze log and time-series data.
<img width="464" height="407" alt="Screenshot 2026-04-07 110442" src="https://github.com/user-attachments/assets/dc66fb37-6888-4001-ab1d-ab342b3bcb6c" />

_Downloading Kibana and running_
- We want to Dowload Kibana from the site using DEB x86_64, and then running on our powershell connected to the Linux server
<img width="414" height="542" alt="Screenshot 2026-04-07 110503" src="https://github.com/user-attachments/assets/445f714c-8bf6-4378-a9ff-61dfb59c135b" />
<img width="842" height="343" alt="Screenshot 2026-04-07 110653" src="https://github.com/user-attachments/assets/6a500d3f-2fff-4039-93da-714c5788dca5" />

_Installing Kibana_
- We then install Kibana such as shown in image <img width="842" height="343" alt="Screenshot 2026-04-07 110653" src="https://github.com/user-attachments/assets/cd48e2da-6040-4fa3-aeb2-83df39cce009" />

_Changed IP address and Port for Kibana using Nano_
- Back in Nano, we have to change the IP address and Port for Kibana
<img width="809" height="354" alt="Screenshot 2026-04-07 110800" src="https://github.com/user-attachments/assets/af204074-a055-4cd9-a8d4-3863e0691704" />
- And select exit prompt for Nano, and save changes

_Started Kibana Service, then applied Token for Elasticsearch_
<img width="960" height="540" alt="Setting up Elastic SIEM on Network using VULTR" src="https://github.com/user-attachments/assets/8d9f8ad4-a6d6-42ed-a173-f096032a4acf" />
- Used Token (hiding under red) and entered into Elasticsearch login screen
<img width="960" height="540" alt="Setting up Elastic SIEM on Network using VULTR (1)" src="https://github.com/user-attachments/assets/0a15700d-21ab-4e63-9cee-afc696778982" />

_Configured Firewall to IP range using TCP and a range_
- Back in VULTUR, we wnet to Manage Firewall Group, and added a TCP protocol for a range of 1 - 65535
<img width="960" height="540" alt="Setting up Elastic SIEM on Network using VULTR (2)" src="https://github.com/user-attachments/assets/d05eb368-9335-44a9-a418-c4e004cd3855" />

_We can now use Elastic with Kibana_
<img width="960" height="540" alt="Setting up Elastic SIEM on Network using VULTR (3)" src="https://github.com/user-attachments/assets/44824b34-cecd-4ca9-a033-a1691d62fc4f" />
<img width="960" height="540" alt="Setting up Elastic SIEM on Network using VULTR (4)" src="https://github.com/user-attachments/assets/65d3befe-936f-4155-b502-02ea35b2b7f9" />

<img width="960" height="540" alt="Setting up Elastic SIEM on Network using VULTR (5)" src="https://github.com/user-attachments/assets/abd90146-d795-42b6-b762-0c0fec642071" />

**Installing Fleet Server with Elastic Agent**

Elastic Agent  is a single, unified, and lightweight software agent used to collect logs, metrics, traces, and security data from hosts (servers, containers, workstations) and send it to ElasticSearch. It replaces multiple specialized "Beats" (e.g., Filebeat, Metricbeat) with one tool, simplifying deployment, management, and security via a central console called Fleet.

The types of beats done in a system are:
File Beat - Logs
Metric Beat - Metrics
Packet Beat - Network Data
Winlog Beat - Windows Events
Audit Beat - Windows Events
Audit Beat - Audit Data
Heartbeat Beat - Uptime

<img width="960" height="540" alt="Setting up Elastic SIEM on Network using VULTR (6)" src="https://github.com/user-attachments/assets/404d9419-119a-40d5-a610-0137c2521ad3" />

_Fleet Servers_
  
  A Fleet Server is a component of the elastic Stack that acts as a centralized control plane for managing Elastic Agents. It enables administrators to remotely manage, configure, and update thousands of agents—which collect logs and metrics—from a   single Kibana Interface.

<img width="960" height="540" alt="Setting up Elastic SIEM on Network using VULTR (7)" src="https://github.com/user-attachments/assets/b0b2452b-28b9-402c-8e8c-17973fce205e" />

_Install Fleet Servers on VULTR_
  
  -Install a new server with Ubunto, using our SOC network. You can use a machine with minimal cores and storage. Once installing, open up powershell and connect to the server via ‘SSH root@yournewipaddress’

<img width="960" height="540" alt="Setting up Elastic SIEM on Network using VULTR (8)" src="https://github.com/user-attachments/assets/528c28ff-95c5-47fc-9011-ac58d3297128" />

_Add Elastic on Fleet Servers_

<img width="960" height="540" alt="Setting up Elastic SIEM on Network using VULTR (9)" src="https://github.com/user-attachments/assets/d4b74a38-570b-4681-9a19-b24c9e5309e1" />

_Add install code from elastic to install on Fleet Server_
  
  -Use the code for Linux 86_64 and paste into Powershell, directed to Fleet Server via SSH

<img width="960" height="540" alt="Setting up Elastic SIEM on Network using VULTR (10)" src="https://github.com/user-attachments/assets/91baf875-f82f-46dc-ae5e-6234cb80c52b" />

We may need to run command ‘ufw allow 9200’, as well as  ‘ufw allow 8200’ and ‘ufw allow 443.
We may also need to allow these ports via TCP on our Firewall in VULTR


<img width="960" height="540" alt="Setting up Elastic SIEM on Network using VULTR (11)" src="https://github.com/user-attachments/assets/0fd6458b-1f66-42d8-8c99-3107d06115e9" />

We will want to add Elastic onto our newer Windows Server we made earlier.
One adding agent, we copy the script for the Windows x86_64 option

<img width="960" height="540" alt="Setting up Elastic SIEM on Network using VULTR (12)" src="https://github.com/user-attachments/assets/e71c0bd8-fd98-4060-9948-c439e0fca123" />

We then log into our Windows server, run PowerShell as Administrator, and run the code provide by elastic server.
It will tell us when enrollment and data are confirmed.

<img width="960" height="540" alt="Setting up Elastic SIEM on Network using VULTR (13)" src="https://github.com/user-attachments/assets/a328d6b4-3d16-40d7-9428-46dd00675500" />

On our Elastic Site, We can go to Analytics > Discover, and view our logs and data on our network we made so far.
We can search for a server name, like the one for our Windows Server, then view associated logs and view information on the right.

<img width="960" height="540" alt="Setting up Elastic SIEM on Network using VULTR (14)" src="https://github.com/user-attachments/assets/d2d65ae8-7b7b-494a-b0ad-fb936012e4f6" />
