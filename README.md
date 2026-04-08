# SOC-Analyst-Lab-using-ElasticSearch-and-Kibana

Elasticsearch is a distributed, open-source search and analytics engine designed for speed, scalability, and near real-time data processing. Built on Apache Lucene, it acts as a NoSQL, JSON-based document store used for full-text search, log analysis, and data visualization, often as the core component of the ELK Stack. 

## Objective

I have conducted a hands-on lab that goes over installing servers, including them on a VPN network, integrating them with Elastic search to be able to be used as a SIEM, system hardening, replicating an attack on both the attacker side and on the defence side withing the SIEM Environment. I will be setting up and configuring a VPC consisting of different servers and firewalls using VULTR. Below is a diagram of the project scope.

This is a a series of labs I have followed along with from [MyDFIR] on Youtube. His play list for this series is called [30-Day MYDFIR SOC Analyst Challenge]
**Source** [https://www.youtube.com/playlist?list=PLG6KGSNK4PuBb0OjyDIdACZnb8AoNBeq6]

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

## Building Ubuntu Server from VULTR (Virtual Network)

_Vultr: VPS Server and Network; I will start by installing an Ubuntu Server from VULTR and applying Elastic Search onto it._
- Vultr is a Global, automated cloud infrastructure from the broadest array of AMD and NVIDIA GPUs to virtual CPUs, bare metal, Kubernetes, storage, and networking.
- After making an account with VULTR, you will need to setup a VPN network, add a Server computer, and configure those together with a static IP, storage information, and applying an Ubuntu Server.
- You then Deploy and it will start building the Ubuntu Server
<img width="920" height="271" alt="Screenshot 2026-04-07 104456" src="https://github.com/user-attachments/assets/8fcde44c-a7a3-4550-8c38-2cef1775c4b6" />


## Install Elastic Search**

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

## Installing Kibana and setting up with Elasticsearch
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

## Installing Windows Server
- For our entire network and Soc environment, we will want to setup a Windows Server, as most offices still use them as their main server for many systems and software to this day.
- See below as my example for setting this up.
<img width="888" height="418" alt="Screenshot 2026-04-07 150208" src="https://github.com/user-attachments/assets/29b2cc0c-fb62-4ad7-875d-fb58498350a5" />
<img width="942" height="476" alt="Screenshot 2026-04-07 150301" src="https://github.com/user-attachments/assets/d4aa0a49-ba19-4909-8fbc-5959c7957692" />
- Make sure it finishes installing, then login with the credentials set in VULTR
<img width="1063" height="480" alt="Screenshot 2026-04-07 150550" src="https://github.com/user-attachments/assets/ea630603-8331-402d-81f1-3029c71132f4" />

## Installing Fleet Server with Elastic Agent**

Elastic Agent is a single, unified, and lightweight software agent used to collect logs, metrics, traces, and security data from hosts (servers, containers, workstations) and send it to ElasticSearch. It replaces multiple specialized "Beats" (e.g., Filebeat, Metricbeat) with one tool, simplifying deployment, management, and security via a central console called Fleet.

The types of beats done in a system are:
- File Beat - Logs
- Metric Beat - Metrics
- Packet Beat - Network Data
- Winlog Beat - Windows Events
- Audit Beat - Windows Events
- Audit Beat - Audit Data
- Heartbeat Beat - Uptime

_Fleet Servers_
  
A Fleet Server is a component of the elastic Stack that acts as a centralized control plane for managing Elastic Agents. It enables administrators to remotely manage, configure, and update thousands of agents—which collect logs and metrics—from a   single Kibana Interface.

<img width="960" height="540" alt="Setting up Elastic SIEM on Network using VULTR (7)" src="https://github.com/user-attachments/assets/b0b2452b-28b9-402c-8e8c-17973fce205e" />

_Install Fleet Servers on VULTR_
- Install a new server with Ubunto, using our SOC network. You can use a machine with minimal cores and storage. Once installing, open up powershell and connect to the server via ‘SSH root@yournewipaddress’
<img width="960" height="540" alt="Setting up Elastic SIEM on Network using VULTR (8)" src="https://github.com/user-attachments/assets/528c28ff-95c5-47fc-9011-ac58d3297128" />

_Add Elastic on Fleet Servers_

<img width="960" height="540" alt="Setting up Elastic SIEM on Network using VULTR (9)" src="https://github.com/user-attachments/assets/d4b74a38-570b-4681-9a19-b24c9e5309e1" />

_Add install code from elastic to install on Fleet Server_
- Use the code for Linux 86_64 and paste into Powershell, directed to Fleet Server via SSH
<img width="960" height="540" alt="Setting up Elastic SIEM on Network using VULTR (10)" src="https://github.com/user-attachments/assets/91baf875-f82f-46dc-ae5e-6234cb80c52b" />

- We may need to run command ‘ufw allow 9200’, as well as  ‘ufw allow 8200’ and ‘ufw allow 443.
- We may also need to allow these ports via TCP on our Firewall in VULTR

<img width="960" height="540" alt="Setting up Elastic SIEM on Network using VULTR (11)" src="https://github.com/user-attachments/assets/0fd6458b-1f66-42d8-8c99-3107d06115e9" />

- We will want to add Elastic onto our newer Windows Server we made earlier.
- One adding agent, we copy the script for the Windows x86_64 option

<img width="960" height="540" alt="Setting up Elastic SIEM on Network using VULTR (12)" src="https://github.com/user-attachments/assets/e71c0bd8-fd98-4060-9948-c439e0fca123" />

- We then log into our Windows server, run PowerShell as Administrator, and run the code provide by elastic server.
- It will tell us when enrollment and data are confirmed.

<img width="960" height="540" alt="Setting up Elastic SIEM on Network using VULTR (13)" src="https://github.com/user-attachments/assets/a328d6b4-3d16-40d7-9428-46dd00675500" />

- On our Elastic Site, We can go to Analytics > Discover, and view our logs and data on our network we made so far.
- We can search for a server name, like the one for our Windows Server, then view associated logs and view information on the right.

<img width="960" height="540" alt="Setting up Elastic SIEM on Network using VULTR (14)" src="https://github.com/user-attachments/assets/d2d65ae8-7b7b-494a-b0ad-fb936012e4f6" />

## Sysmon Setup
Sysmon (System Monitor) is a Windows system service and device driver from Microsoft Sysinternals that provides advanced, real-time monitoring of system activity, including process creation, network creations, and file changes. It is designed for deep security visibility, helping detection teams identify malicious behavior and “living-off-the-land” attacks that standard Windows logs often miss.

_Key Features & Benefits_
- Detailed Process Tracking: Logs full command-line arguments for processes, including parent processes.
- File Hash Integrity: Records SHA1, MD5, or SHA256 hashes of process images to detect modified malware.
- Network Connections: Tracks TCP/UDP connections, including the source process, IP addresses, and ports.
- Advanced Detection: Detects file creation time changes (timestomping), driver loading, and raw disk access.
- Rule Filtering: Allows including or excluding specific events to reduce noise and optimize storage.

_Download Sysmon on Windows Server_
- You should be able to RDP into it via the IP address and password listed in VULTR
- Then, we can Download Latest Symon exe

<img width="798" height="493" alt="Screenshot 2026-04-07 153645" src="https://github.com/user-attachments/assets/48bf5079-2c5c-4d15-9266-01db77dc0ec0" />

- There is a Sysmon Configuration script we will need to add to the Sysmon installer. We can get that from Github. When we open that and go to ‘raw’, we can save as and then save it INTO the Sysmon download folder.

<img width="694" height="500" alt="Screenshot 2026-04-07 153713" src="https://github.com/user-attachments/assets/29051234-d58f-4010-96d7-81958f38ffdb" />

- We then run the exe from Powershell on the Windows Server machine.

<img width="736" height="534" alt="Screenshot 2026-04-07 153751" src="https://github.com/user-attachments/assets/beb547e5-4d7e-4e27-b2ca-b9f25c0d4698" />

- Once installed, we can see it is installed and running in Services.

<img width="650" height="503" alt="Screenshot 2026-04-07 153811" src="https://github.com/user-attachments/assets/8efb90a0-715c-4fc4-9a6e-9aa917413764" />

## ElasticSearch Ingest Data
Ingesting data into Elasticsearch involves collecting, transforming, and sending data to an index where it can be searched.
- Click on Add Integration
- Search for Custom Windows event log package

- The custom Windows event log package allows you to ingest events from any Windows event log
(external, opens in a new tab or window)
channel. You can get a list of available event log channels by running Get-WinEvent -ListLog * | Format-List -Property LogName
(external, opens in a new tab or window)
- in PowerShell on Windows Vista or newer. If Get-WinEvent is not available, Get-EventLog *

<img width="1905" height="954" alt="1_add_integrations" src="https://github.com/user-attachments/assets/82333f6a-4ef5-4e1c-b5ab-797bc39b291a" />
<img width="1352" height="518" alt="2_Windowscustomevenlogs_choose" src="https://github.com/user-attachments/assets/fad2a6f2-ec16-448e-8039-8d6518461205" />

_Windows Custom Field Mapping_
- here, we can review the fields that we wormally view from Windows event viewer. These fields and ID's are what we can use to search for them in our Elasticsearch to view logs and do analysis.
<img width="968" height="899" alt="3_FieldMappings" src="https://github.com/user-attachments/assets/b4dd2d06-a83c-4562-8139-26d01d75103e" />

_Add Event Logs integration_
- Enter in the fields needed for this integration
- Choose a name

<img width="958" height="791" alt="4_addintegrationfields" src="https://github.com/user-attachments/assets/e4609a56-6f53-45fb-bdb5-52736cd92c0f" />

- For the channel name, it SHOULD be the full name of the Sysmon Operation Logs found in Event Viewer, see image below.

<img width="988" height="684" alt="5_EventviewerSysmonlogs" src="https://github.com/user-attachments/assets/90096d83-679d-484d-8f8e-2eac17e68150" />

- We want to add this to an existing host - Choose our existing Windows Policy made previously in Elastic.

<img width="801" height="254" alt="6_Existinghost" src="https://github.com/user-attachments/assets/70c87ed9-8726-43be-8920-d985bccab511" />

_Research Windows Event ID’s we want_
- We then want to add another policy from our Windows Defender
- First, we wanna check some id’s we would want to include in our policy and log notifications

<img width="894" height="843" alt="8_WindowsEventViewerlog_id_1116" src="https://github.com/user-attachments/assets/dfb09e01-2c95-4012-a4c3-7c89188fcf6a" />
<img width="566" height="232" alt="8_WindowsEventViewerlog_id_5001" src="https://github.com/user-attachments/assets/f51f0581-0137-4cfc-8233-b537ff0bec31" />

- Then, test by disabling firewall on Windows Server

<img width="1187" height="621" alt="9_Disable_firewall_to_test_logs" src="https://github.com/user-attachments/assets/3e51883b-14a7-4b0c-828b-99f286bd001a" />

- Then, we wanna check event viewer to make sure we can get a 5001 notification.

<img width="635" height="299" alt="Screenshot 2026-04-07 165852" src="https://github.com/user-attachments/assets/3a8ba071-b264-452f-bf0b-c687fca7550c" />

- After that, re-enable firewall

## Add Windows Defender Integration
- Add new integration and enter in the approraiet fields

<img width="981" height="463" alt="7_Windows_SysmonAddedtoPolicy" src="https://github.com/user-attachments/assets/62e936b4-a641-42cd-baa6-3aced4cf9dae" />
<img width="843" height="616" alt="10_Start_addingWindefenderIntegration" src="https://github.com/user-attachments/assets/c8f84b5d-98d7-400c-9fca-ffa454f78ca6" />


- We are getting the channel name for the Windows Defender Operation Logs under Applications section in Event Viewer, and then clicking on properties to view name.
- Enter in the event id’s we want. In our case, enter 1116,1117,5001 (just like that with no spaces)

<img width="437" height="248" alt="10_addeventids" src="https://github.com/user-attachments/assets/0485ff7a-1e1f-4b1d-883b-c5faa0973a9e" />


- If you want to leave out an id, add a ‘-’ infront of that id.
- Add this integration to the existing Windows policy

<img width="433" height="127" alt="11_addtowindowspolicy" src="https://github.com/user-attachments/assets/6d5bc21a-6fa6-49ed-baa8-0d303bfc3d7b" />

- We now have our integrated Windows Policies

<img width="933" height="435" alt="12_we_have_our_integration_and_logs_pulled_from_window_server" src="https://github.com/user-attachments/assets/bf5d9e74-0ea0-4908-a9a3-cfcef0a307ea" />

_We may have to add TCP port 9200 to Anywhere in our Firewall set in VULTR_

<img width="1502" height="478" alt="15_add_Firewall_rule_9200" src="https://github.com/user-attachments/assets/55bfb396-bcf9-4ff6-843a-dd9679f30bc6" />

_Confirm we have live data coming into our Windows Server_
- In our Elastic Server site, click on the hamburger on the top left.
- Under Management section, click on Fleet
- We should be able to see we have CPU usage and memory for both Fleet Policies (one for Fleet Server and one for Windows Server)
- Click on our Windows Server policy
- We can see we have usage, memory, and that it is currently Healthy.

<img width="1225" height="899" alt="16_wenowhave_memoryandvalues" src="https://github.com/user-attachments/assets/e8fa15cd-5338-4fb1-8e81-00c02812e18a" />

_Finally, make sure we can view logs coming from our Windows Server's Event Viewer_.
- Click on the hamburger in the upper left
- Under Analytics section, click on Discover
- in the search bar above the graph, type in something like 'winlog.event_id: 1'
- Click on the arrows left of a tab.
- Click on the second page under document information.
- Then see we have event provider as Microsoft-Windows-Sysmon

<img width="1916" height="914" alt="17_Microsoft_Windows_sysmon_eventID_Dashboard" src="https://github.com/user-attachments/assets/1b71936e-673f-4a2c-97f3-c5e4d1b7c5ac" />

- Now, in the search, type in 'winlog.event_id: 5001'
- You may have to filter the calendar to include Today or the last week.
- We should have at least one record showing after waiting a bit.
- Click on the arrows to show documentation, then go to 2nd page and see our event provider is Microsoft-Windows-Windows Defender

<img width="1911" height="912" alt="18_Microsoft_Windows_windowsDefender_eventID_Dashboard" src="https://github.com/user-attachments/assets/2ec366b0-fe48-4e77-adc5-538b766406b2" />

_And Congrats! We have data coming into our ElasticServer, and we are ready to play around to view varoius events and information in our SIEM._
