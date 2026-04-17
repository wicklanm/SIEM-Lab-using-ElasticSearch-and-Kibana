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

## Setup SSH Server and View Authentication Logs
- Create a new server in VULTUR
- Give in 1 GB memory and one RAM
- Choose your location
- Choose Ubunto (at this time selecting v25)

<img width="923" height="591" alt="1_installing_Ubuntu_onVULTR" src="https://github.com/user-attachments/assets/e08ea51d-28df-4249-b119-40206937a176" />

_Log into our Ubunto server via Powershell and SSH (SSH root@ipaddressofLinuxserver)_
- update our server by running 'apt-get update && apt-get -y

<img width="1054" height="794" alt="2_apt_get_update" src="https://github.com/user-attachments/assets/3f598000-a51e-41a9-a7ce-8e81e09eb596" />

<img width="1136" height="852" alt="3_varlogs" src="https://github.com/user-attachments/assets/d5a2adcf-af1a-4d37-b93a-fc4ae241f18d" />

_Activating logs on Ubuntu Server_
- Go to our directory for Var/Log, and then, run:

- cat auth.log
- This will eventually load up with logs showing failed authentications

<img width="827" height="208" alt="4runprompts_inorder_to_get_logs" src="https://github.com/user-attachments/assets/3a22281d-27af-43ca-8e5e-a9f4dd2e7362" />

## Install Elastic Agent on our newly added Linux Ubuntu Server
- We will want to create a new Fleet policy
<img width="1191" height="583" alt="1_Create_new_Fleet_Policy" src="https://github.com/user-attachments/assets/d771ab33-3ff6-49a2-a54f-0037618ce72f" />

<img width="1196" height="581" alt="2_Create_new_Fleet_Policy" src="https://github.com/user-attachments/assets/c8ee6bb1-5130-4c14-8576-ee4b32907a83" />

- Click on Integration Policy (for me, system 4)

<img width="1200" height="391" alt="3_click_on_integration_policy" src="https://github.com/user-attachments/assets/7951ee13-aca9-418e-8191-e01628b166c0" />


- We will need to access logs
- Linux authorization logs are critical for monitoring security events like failed login attempts, sudo usage, and SSH activity.
- The /var/log/auth.log file (on Debian/Ubuntu) or /var/log/secure (on RHEL/CentOS) is a critical Linux log that records all authorization-related events, including successful/failed login attempts, SSH connections, sudo usage, and user management actions. It is essential for monitoring security and detecting unauthorized access.

<img width="910" height="274" alt="4_Confirming_we_are_logging_for_authlog" src="https://github.com/user-attachments/assets/2d93c696-8b22-4ee1-bdf7-88d20cb9fd3c" />

_Adding a new Fleet Agent_
- Choose our newly added Linux Policy
- Choose Enroll in Fleet (Enroll in Elastic Agent in Fleet to automatically deploy updates and centrally manage the agent

<img width="455" height="754" alt="5_add_new_fleet_policy" src="https://github.com/user-attachments/assets/729ab80c-1f36-495a-931b-6bfd6502cf5c" />

- Select the Linux x86_64 option from the dropdown
- Copy the command (copu icon on upper-right of command)

<img width="451" height="690" alt="6_add_new_fleet_policy_chooseLinux_x86_64" src="https://github.com/user-attachments/assets/9918834f-29e9-4273-808d-be3d92a34e8f" />

- Head over to Powershell, type and enter 'CD' to back out of the var/log directorypaste command and hit enter.
- Paste the command and hit enter

<img width="971" height="399" alt="7_ElasticAgent_Policy_install" src="https://github.com/user-attachments/assets/42bde1c5-6231-47b6-b9aa-589f257e3e3f" />

- Hit Y for Yes to install as service

<img width="947" height="411" alt="9_ElasticAgent_Policy_instal_hityesl" src="https://github.com/user-attachments/assets/f656702c-c8a6-45ce-944a-15759b95d3d2" />

- During install, we get an Error: fail to execute request to fleet-server: x509

<img width="962" height="414" alt="10_error_ElasticAgent_Policy_instal_hityes" src="https://github.com/user-attachments/assets/6e3395b9-4da8-4c4e-aa84-d65fc05a96ba" />

- Ran again, and it finished installing

<img width="930" height="520" alt="13_installed correctly" src="https://github.com/user-attachments/assets/57a09a8d-0206-449e-a674-4477eb301f83" />
<img width="454" height="731" alt="14_installed correctly" src="https://github.com/user-attachments/assets/7d56cea5-713e-4a12-ba85-a4e04d3d74e4" />

- We confirmed we have logs coming through from our new Linux Ubunto Server

<img width="806" height="772" alt="15_conirmedwehave_data" src="https://github.com/user-attachments/assets/24cdda66-198c-42eb-b6ba-b81250e80596" />

- Clicking on the plus sign next to that server allows us to see logs in the table from our new Linux Server

<img width="806" height="772" alt="16" src="https://github.com/user-attachments/assets/6c4190d3-30c9-4db8-a87c-0880bdd11aeb" />
<img width="970" height="618" alt="17" src="https://github.com/user-attachments/assets/b487f03f-7cd1-48cb-b9cf-391c479c8740" />

- We now have more data and logs coming into our Ubuntu/Linux server

<img width="1894" height="903" alt="18" src="https://github.com/user-attachments/assets/dd06258b-5990-40f3-b5b5-3a92c06400ab" />

## Create SSH Brute Force Alert & Dashboard

<img width="1073" height="900" alt="1_ Review_activitylogs_onLinixServer" src="https://github.com/user-attachments/assets/a216e0a9-26a5-46cf-b82b-579c915e1144" />


_Check Dashboard for failed Athentications_
Now that we have our dashboard up, we want to look for failed authentications for brute force attacks. Some fields qwe are looking for are
- failed attempts
- user
- source ip address

<img width="1911" height="641" alt="2_Failed_attempt_logs" src="https://github.com/user-attachments/assets/0b0ce3e6-6a38-4ec5-ac99-d5ba611616f9" />

_Add Columns to dashboard to look for failed attempts_

To get some valid data regarding attemps made from questionable sources, we want to add columns (list of colums found on left under 'search field name), we want to add 'system.auth.ssh.event', username, source.ip, and the source.geo.country_name. We can now see failed attempts amde from these external sources.

<img width="1909" height="576" alt="Bring them all together now with failed attempts" src="https://github.com/user-attachments/assets/968a2961-6f64-4ad8-88b9-013770250c13" />
<img width="1905" height="750" alt="5_Source_IP_ _Coutnry_Name" src="https://github.com/user-attachments/assets/3b894fab-5669-439b-b25e-19b29805d8a4" />
<img width="1322" height="651" alt="4_usernames_logs" src="https://github.com/user-attachments/assets/4a7f6a99-265e-4ae6-8306-764365ed194c" />
<img width="1304" height="552" alt="3_system_auth_Failed_logs" src="https://github.com/user-attachments/assets/43b42479-0d4c-45f0-ab8b-10c688fc87b2" />

- Let's save our SSH Search Activity

<img width="1349" height="777" alt="7_save_our_SSH_Search_Activity_Search" src="https://github.com/user-attachments/assets/db2ee7f9-42a7-449f-8ad3-3637b66fc01a" />

_Create an Alert for failed SSH Attempts_

Let's now create an alert showing failed ssh attempts within the last 5 minutes, and then test the query.

<img width="1338" height="891" alt="8_Create_an_alert_andtest_quesry" src="https://github.com/user-attachments/assets/d05b44e6-8d72-4405-97ca-133db411618d" />
<img width="617" height="860" alt="9_Test_query" src="https://github.com/user-attachments/assets/0788d3e3-4ca6-4e1d-ae96-fd3099fa8ea9" />

- We have 22 attempts loged in the last 5 minutes for our alert
- Select Next
- We are going to skip the Actions section for now and keep the alert simple.
- Give the Alert rule a name, then select 'Create rule'

<img width="626" height="810" alt="10_Give_rule_name_and_select_create" src="https://github.com/user-attachments/assets/7164c4c4-1b64-4ce7-a3f5-5ceb462508be" />

_Create a Dashboard to Visualize where these attacks are sourcing from_

- Click on hamburger icon on top left of dashboard
- Under Analytics sections, click on maps
- See sections that says 'Filter your data using KQL syntax'
- Go back to our dashboard quickly and copy our query we used:

<img width="1626" height="133" alt="11_Copy_query_we_had_on_dashboard" src="https://github.com/user-attachments/assets/e889ee4d-40b1-4665-8bfe-1f596652c993" />

- bring over to a notepad and combine that query with failed attempts, user, and source ip coulumns, as such:
system.auth.ssh.event : * and agent.name: MYSOCENV-Linux and system.auth.ssh.event: Failed
- Hit enter to validate that it returns with logs
- copy that query
- Head back over to Maps under Analytics section
- Paste that search query we just copied
- content will not show on map yet

<img width="1309" height="601" alt="12_enter_query_in_maps_section" src="https://github.com/user-attachments/assets/fc45d002-c8ff-4b83-b8d2-191fdf5527eb" />

- Let's add a layer by choosing Chlorepleth (shaes areas to compare statistics across boundaries)
- We can choose from administrative boundaries from the Elastic Maps Service, OR, points, lines, and polygons  from Elastic Search. For this lab, we will choose Administrative boundaries and select World Countries.
- Keep the Join Field as defulted
- For Data view, we wanna use what we were using for the dashboard.
- Confirm we have the same dataview from dashboard

<img width="496" height="515" alt="14_DataView" src="https://github.com/user-attachments/assets/f6567de5-f1bf-462d-b7ef-684c205c09fd" />

- Then, select that option as the data view
- Then for join field, we will want to select 'source.geo.country_iso_code'. We don't want to use Country name, as we will likely get following error:

_An error occurred when adding join metrics to layer features

Left field values do not match right field values. Left field: 'ISO 3166-1 alpha-2 code' has values: AD,AE,AF,AG,AI (5 of 500). Right field: 'source.geo.country_name' has values: Romania,The Netherlands,United States,Netherlands,Luxembourg (5 of 23)._

-The Join Field (ISO 3166-1 alpha-2 code) wants the abbreviated name for this.

- Select 'Add and continue' for the map

<img width="1917" height="877" alt="15_Add_and_continue_map" src="https://github.com/user-attachments/assets/4e21de04-de1b-462d-8023-5f5956022100" />

- Review Layer Settings. For this lab, For this instance, we can leave these as defaulted.

<img width="448" height="663" alt="16_layer_Settings" src="https://github.com/user-attachments/assets/40cfe308-5e28-4a47-89b3-7026f26219d6" />

- Click on Save on top right
- Give the map a title, like SSH Network Map, and save to Dashboard.

- In the dashbaord, give it a name, and you can sift through the time to see the countries change it colors. We can see a lot of attacks from China, and definately Romania.

-Give the map a name as well, like "SSH Failed Authentication - Network Map"

<img width="1912" height="637" alt="18_Save_to_dashboard_edit_time_Analyze_where_attacks_are_coming_from" src="https://github.com/user-attachments/assets/ef51c717-f076-4c30-80ab-adb0d03c45bb" />

_Make another map showing Successful authentications_
- Make another map/dashboard by duplicating this one (three dots on upper right). Give it a name like SSH Successful Authentications, and apply.
- Change the query at the very end to read "Accepted" instead of "Failed"

<img width="1314" height="628" alt="Accepted_SSH_Map" src="https://github.com/user-attachments/assets/3d26b504-df5d-45b1-9b58-e8a9c753b490" />

- Select Save and Return

_Comapre our two maps_
- We have thousands of failed attemps coming from various countries and foreign ip's, 
- and we have only 3 successful SSH attempts coming from the USA.
- Select Save to complete the dashboard.

<img width="1443" height="868" alt="Screenshot 2026-04-13 173720" src="https://github.com/user-attachments/assets/b6a13b03-a117-4fae-8445-f521626ca21f" />

**_Congratulations! We now have an alert and an affective dashboard!_**
_We can now be alerted the next time a brute force attack occurs, and we can view where these are sourcing from._

## Remote Desktop Protocol

_RDP was abused in over 90% of cyber attacks_
In this section, we will learn about RDP, why it is used, how attackers abuse it, how to find endpoints with exposed RDP and how you can protect yourself.

### What is it?
- Remote Desktop Protocol is used for communication between Terminal Server and the Terminal Server client. RDP is encapsulated and encrypted within TCP and has a default port of 3389/TCP.
- It can be used remotely to connect to another machine.

### How does it get abused?
- An attacker can gain entry into a network or an environment through an exposed RDP service to a server.
- Once they have access, they can perform credential dumping to gather valid credentials found on that server, then, use those credentials to move laterraly onto other servers or machines withing the same network using RDP.
- They can also conduct data exfiltration, ransomware, or both.

### How to find Exposed RDP Servers
_Shodan_
- Shodan is a specialized search engine that crawls the internet to index devices, servers, and systems—such as webcams, routers, and industrial control systems—rather than webpages.
- As an example, On Shodan, once we type in 'port:3389' onto the search, it gives us data showing that there are over 2,500,000 RDP instances where the default 3389 port is set as open!

<img width="1149" height="676" alt="Shodan_port3389_search" src="https://github.com/user-attachments/assets/a108568e-5671-41cb-afa5-cb539cbee9cf" />

_Censys_
- Censys is a cybersecurity platform that continuously scans and maps the entire internet to identify, monitor, and analyze publicly accessible devices, domains, and certificates.
- We can enter a search query to view exposed RDP sessions and their host IP address by entering "host.services: (port: 3389 and protocol: "RDP") "

<img width="1629" height="871" alt="2_Sensys_Search" src="https://github.com/user-attachments/assets/4ad58634-b5e2-448b-b435-bf627620a278" />

- We can then click onto an individual RDP session. We can see if it is secure or not, and the specefics of the connection. Using this site can help us identify assets with potentially sensitive services that are exposed to the internet. Knowing these can help us reduce the risk of potential compromise.

<img width="1895" height="820" alt="3_Sensys_Search" src="https://github.com/user-attachments/assets/b69a68c4-88e0-481e-8436-dc8d000a2001" />

This result below is potentially an example of a bad RDP instance since the browser is not trusted, and perhaps they are using HTTP for a login page in a network.

<img width="878" height="756" alt="4_Sensys_Search" src="https://github.com/user-attachments/assets/e42a3236-017d-4bce-945a-38150d8e2df1" />

### How to Protect Yourself

1) Turn it off. Often, developer tend to use RDP to remote into servers to review or make code on a server, in SQL, or something else, then fogot to turn off the session when they are done.

2) Use Multifactor Authentication

3) Restrict Access. Implement firewall rules to restrict access to indivuals outside an environment or domain, so onlt the ones inside can RDP.

4) Have better Passwords (15 or more characters, containg upper,lowercase, numbers, and special characters.

5) Don't use default accounts (Disable default accounts coming from systems and make new credentials with complex passwords and other security features)

## Creating Alerts and Dashboards for Windows Server

We will be looking for brute force attacks. The Event ID for a failed login attemopt is 4625. This event is recorded in the security log whenever a user fails to log on to a Windows system, capuring key details such as the username, source IP address, and the reason for the failure.

_Creating Failed logon Dashbaord_
- We can now have our dashboard setup to include just these events by entering 'event.code: 4625'
- We will also add 'source.ip' and 'user.name'

<img width="1218" height="739" alt="4_eventcodeescontaining4625" src="https://github.com/user-attachments/assets/4e47c4c4-ca53-4957-8b3c-a9d1f6da9ff3" />

- Enter save and give it a name

<img width="1952" height="824" alt="5_saverdpfailedactivity" src="https://github.com/user-attachments/assets/c9067a02-e9f1-4d2a-aeb0-a2d344398b9b" />

- We see for this attempt on top, if we expand the message field, and view Logon Type, we have 3, which means it is a network-based authentication.
- Let's test the log by trying to RDP into the Windows server. Let's falsley type in the password so we can get a failed message.

<img width="446" height="493" alt="7_failed_RDP" src="https://github.com/user-attachments/assets/b5f4963c-98d1-4738-9e48-4fbcbb49f451" />

- Once failed, go back to dashbard and refresh and filter down to the last minute. We have our failed attempt showing here. Also, if you expand the message you can confirm it iss a logon type 3.

<img width="1218" height="461" alt="8_Failed_attempt_dashboard" src="https://github.com/user-attachments/assets/20d4a092-2fc2-452a-a5ab-f5613c632357" />

_Create successful logon Dashboard_

- We want to create a search Threshhold Rule from the dashboard, upper left.
- Set the definition like so:

<img width="497" height="833" alt="9_set_definition" src="https://github.com/user-attachments/assets/7ed2c6fd-e433-49eb-96b0-db471626d93d" />

- add name and select 'create rule'  (can skip actions)

<img width="480" height="829" alt="10_create_rule" src="https://github.com/user-attachments/assets/93025016-22de-41e9-94e6-d996b4d97188" />

_Let's create a new rule for our alert!_
-Go to Security under hamburger icon, click Rules.
- Click on add new rule.
- Select Threshold rule

1) Define Rule
- enter in our query we used in previous section: system.auth.ssh.event : * and agent.name: MYSOCENV-WIN and system.auth.ssh.event: Failed and user.name: "root" 
- add user.name: root at the end to include that.
- Group by both user.name and source.ip
- Add source.ip and username fields as required fields.

2) About Rule
- For the name, give it something obvious like "MYSOC-SSH-Brute-Force-Attempt"
- Description can be something like: This rule detects failed authentications towards the account "root"
- Let's make the Severity medium
- can keep Default risk score as 50
- Under Advanced Settings, we could choose to enter any Referance URL's if we see anything online or cybernews saying there are attacks coming from a certain source. We will leave blank for this.
- Select 'Source.ip for the Custom Highlighted field.
- Select continue

3) Schedule Rule
- Can set to run ebery 5 minutes and look-back time can be 5 minutes.

4) Rule actions
- leave as is.
- Create
- We have our rule

<img width="1031" height="754" alt="11_BruteforceRulecreated" src="https://github.com/user-attachments/assets/d0f9b392-3a87-4ecf-833b-0a6df341043a" />

_Now we will create a new rule for our RDP sessions_

- Create new rule with the same template as before
1) Define Rule
- enter our query from before, and then add user.name: Administrator instead: system.auth.ssh.event : * and agent.name: MYSOCENV-WIN and system.auth.ssh.event: Failed and user.name: Administrator
- add user.name: root at the end to include that.
- Group by both user.name and source.ip
- Add source.ip and username fields as required fields.

  <img width="619" height="794" alt="12_RDP_Rulecreated" src="https://github.com/user-attachments/assets/08236d5d-b6c8-44d4-968d-876307588af0" />

2) About Rule
- Add name, include RDP in there to identify it is alerting for RDP sessions.
- Set description
- set severity medium
- leave default risk score

3) Schedule Rule
- Have it run every minute, and have addmitional look-back time be 5 minutes.

- Create and enable rule

<img width="1034" height="753" alt="13_RDP_Rulecreated" src="https://github.com/user-attachments/assets/243545f2-ee5e-41c2-a700-cd39a200c06d" />

_Later, we can test these rules_

<img width="1897" height="651" alt="14_DetectionRules" src="https://github.com/user-attachments/assets/f2c17616-5321-4d20-83c7-f5003a215664" />

- We can now check our alerts and see if we have any coming from our rules. 

_Uh-oh_. We are getting and error from the Security > Rules menu: 
Error: Forbidden     at Fetch.fetchResponse (http://149.28.117.193:5601/80a75d1ae44e/bundles/core/core.entry.js:1:244683)     at async fetch (http://149.28.117.193:5601/80a75d1ae44e/bundles/core/core.entry.js:1:248484)     at async http://149.28.117.193:5601/80a75d1ae44e/bundles/core/core.entry.js:1:242716     at async http://149.28.117.193:5601/80a75d1ae44e/bundles/core/core.entry.js:1:242673

- The "Forbidden" error in Elastic/Kibana is an HTTP 403 response, meaning the system is rejecting the request due to insufficient permissions. It's not a configuration bug in my detection rule itself — it's an authorization/privilege issue preventing the rule from executing or writing alerts.

- So after asking about this issue in the Elasticsearch Forum (I recommend turning to that if you are ever stuck in Elasticsearch or Kibana), this is a bug current to version 9.0 to 9.3.3
- I am current to 9.2, so I will have to update my ElasticSearch system to either version 9.3.4, or version 9.4. I will have to wait until; they are available.
- I will be following this guide to update:
https://www.elastic.co/docs/deploy-manage/upgrade/deployment-or-cluster/self-managed

<img width="774" height="777" alt="16_Update_ElasticsearchClusterGuide" src="https://github.com/user-attachments/assets/e7db4026-3614-41c4-9597-4bfeb7909f35" />

## Create a Dashboard for RDP Activity

- Let's Select our saved Dashboard for RDP failed Activity

<img width="1333" height="826" alt="1_Review_Our_dashboard_selectsavedone" src="https://github.com/user-attachments/assets/925bdbf4-52b2-4cb9-aa84-802760e81e91" />

- Copy that event code: 4625 in the search, and enter it in our notepad from before. Let's make a query to include that event id with our agent name.

<img width="508" height="199" alt="2_copy_code" src="https://github.com/user-attachments/assets/1baed7a8-c5ac-4306-832a-9858b9001778" />

- Copy that code, go to maps, and past in that query
- Add a layer and select Chlorapleth
- Enter in the information like so:

  <img width="1338" height="911" alt="3_Make_map" src="https://github.com/user-attachments/assets/788f4554-b1b3-4d62-bb4f-de0c6f3f405d" />

_There are 34,115 failed attemps coming from Romnia in the last 7 days.... wow!_

- Click on Continue, then save on top right (we want to save to Library).
- We wan to add this new map to our existing Dashboard (MYSOC-Authentication-Activity) where we made the other two maps (SSH failed and successfuly activity)
- If you cannot add it from there, you may need to go to that Dashboard first, select add, select Library, and choose our new map from there.
- feel free to toggle and adjust the maps to fit neatly in the dashboard.

### Creating a 4th map to show successful RDP conneciton attempts

_If we look up what even ID shows a successful login to a computer, a simple internet search can tell us._

- Windows Event ID 4624 signifies a successful logon to a computer. Generated in the Security log, this event records essential forensic data including the username, domain, logon type, and source IP address. It is crucial for monitoring unauthorized access, lateral movement, and insider threats.
- Let's go back to our Discover screen in Elastic, and add event.code: 4624 to the search

<img width="1332" height="597" alt="4" src="https://github.com/user-attachments/assets/d6a75c82-ab47-47e3-8da0-08c9739d2fa9" />

- Notice our logon Type is 5, which is a service startup logon.

<img width="1054" height="518" alt="5" src="https://github.com/user-attachments/assets/785ddfc5-91de-481c-87db-e312d13809c4" />
<img width="614" height="719" alt="6" src="https://github.com/user-attachments/assets/fc73ae73-3d16-4504-8654-cc52f9950636" />

- We will need to use that field, winlog.event_data.LogonType

<img width="638" height="598" alt="7_windlogeventlogontype" src="https://github.com/user-attachments/assets/eabc4c12-5bfd-4b7c-b372-882f989eeb53" />

- Referring to our table, we will want codes 7 and 10. The quwey will be:
- event.code: 4624 and (winlog.event_data.LogonType: 10 or winlog.event_data.LogonType: 7)


<img width="1070" height="493" alt="Screenshot 2026-04-16 174252" src="https://github.com/user-attachments/assets/587a5f43-6d96-434e-8a45-0b72c7a8cfad" />

- Save this Discover as 'RDP Sucessful Activity'

_Add the new map in our Dashboard_

Back in our Dashboard, Click on the 3 dots 
on our RDP failed Authentication map, and select Duplicate.
- Edit the duplicate map
- Update our query on the top by pasting our newer query
- See the map update with data
- Select Save and return
- Change the name to something like 'RDP Successful Authentications' and select apply.

<img width="1333" height="866" alt="Screenshot 2026-04-16 175041" src="https://github.com/user-attachments/assets/e3ce1e4a-1b67-4509-ab2c-e317f09ad94f" />

- We have our Dashboard
- If we toggle to the last 15 minutes, we should not have much, if not anything for failed attempts. Except for me, I have one failed RDP authentication. We do see some activity for failed RDP connections, and 36 failed SSH connections from Romania. 

<img width="1609" height="731" alt="Screenshot 2026-04-16 175525" src="https://github.com/user-attachments/assets/93f8e6c6-cbff-45cf-ae61-755b28e14a99" />

_Improving our visuals for the dashboard_
- On Dash board, select add, and select Visualization.
- Add our first query used, system.auth.ssh.event: * and agent.name: MYSOCENV-Linux and system.auth.ssh.event: Failed
- Add a timestamp, source.ip
- on top right, select table in dropdown instead of bar chart
- We have our source ip column and timestamp column. Make the timestamp appear first.
- include user.name and sorce.geo.country_name
- Afer looking at table, the timestamp just create more noise than needed, so let's remove that from our rows in the table in the right to remove it.
- click on the user.name and make the number of values to be 10, and, take off the group remaining values as 'other' option.
- Do the same for Source.ip as well.

<img width="344" height="763" alt="Screenshot 2026-04-16 181955" src="https://github.com/user-attachments/assets/1ece63a0-0dc6-4957-a136-0b56aa187eba" />

- set our Count of records as Sort Decending
- Our dashbopard is setup. Now let's Save and return.

<img width="1912" height="887" alt="Screenshot 2026-04-16 182236" src="https://github.com/user-attachments/assets/daadefa9-fc3f-4b1b-82ac-bd4095eef3c5" />

- In our Dashboard, click on settings for the new view, and give it a name like 'SSH Failed Activity [Table]'. Click Apply.
- Let's put that table between our SSH Maps (failed and successful)

<img width="1904" height="865" alt="Screenshot 2026-04-16 182825" src="https://github.com/user-attachments/assets/12352c9d-1ac7-4c05-9bfa-27dbb5f64aee" />

- Now, let's add another table for our Successes.
- Duplicate the table
- Change the name to 'SSH Successful Authentications [Table] '
- Adjust the query for this new table by clicking on it's settings.
- Select Edit next to the query shown. Yopu might have to also click on 'Edit in Lense'
- In query on top bar, change from Failed to Accepted at the end.

<img width="1330" height="543" alt="Screenshot 2026-04-16 183813" src="https://github.com/user-attachments/assets/7fc0d681-fad4-41d3-9e88-24fd8164215e" />

- Click on Save and Return. We have our tables for SSH authentications

<img width="1400" height="859" alt="Screenshot 2026-04-16 184104" src="https://github.com/user-attachments/assets/69fee723-a8c8-4aa3-b855-d21ff339aa30" />

_Now creating tables for RDP authentications_

- Duplicate one of the tables
- Give it a name like RDP failed Authentications
- Edit the query, and paste the query in the search bar FROM our notepad (event.code: 4625 and agent.name: MYSOCENV-WIN).
- Save and Return.


_Now making RDP successes_
 Duplicate the last table
- Give it a name like RDP failed Authentications
- Edit the query, and paste the query in the search bar FROM our notepad (event.code: 4625 and agent.name: MYSOCENV-WIN). Update.
- Save and Return.

_All Tables have the username, source ip, country of origin, and the number of authentication attempts._

- Select Save on top right to save our Dashboard.

**_Congragulations! We now have a Dashboard that shows global maps with their corresponding tables of failed and successful Authentications from their source of origin, with their source ip address and number of records shown._**

<img width="1909" height="906" alt="Screenshot 2026-04-16 185250" src="https://github.com/user-attachments/assets/46a24749-4e55-43cf-b79f-b65e42a8953f" />
