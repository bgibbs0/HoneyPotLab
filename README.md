<h1>Azure Honeypot Lab</h1>

<h2>Description</h2>
This project consists of spinning up a virtual machine in Microsoft Azure that will be used as a honeypot. The honeypot VM will be connected to Log Analytics Workspace and Azure Sentinel (SIEM). These connections will populate the failed RDP (Remote Desktop) logs into Microsoft Sentinel, allowing me to plot the failed connection attempts on a world map. This lab uses Josh Madakor's <a href=https://youtu.be/RoZeVbbZ0o0">SIEM tutorial video</a> as guidance.
<br/>

<h2>Languages Used</h2>
- Powershell: Used to extract Windows Security logs</br>  
- Kusto Query Language (KQL): Used to parse raw data from Windows Security logs</br>

<h2>Utilities Used</h2>
- Microsoft Azure: Used to deploy honeypot virtual machine</br>
- Microsoft Sentinel: Azure Cloud SIEM that populated the honeypots security alerts</br>
- Log Analytics Workspace: Azure service that stores the extracted log data</br>
- ipgeolocation.io: API that provided IP geolocation data from security logs</br>

<h2>Glossary</h2>
API - An application programming interface is a way for two or more computer programs to communicate with each other. </br>
</br>
Azure - cloud computing platform run by Microsoft, which offers access, management, and development of applications and services through global data centers </br>
</br>
Brute Force Attack - a hacking method that uses trial and error to crack passwords, login credentials, and encryption keys</br>
</br>
Honeypot - network-attached systems intended to mimic likely targets of cyber attacks, such as vulnerable networks. </br>
</br>
Microsoft Sentinel - a cloud-native security information and event management (SIEM) platform that uses built-in AI to help analyze large volumes of data across an enterprise. </br>
</br>
Remote Desktop - proprietary protocol developed by Microsoft Corporation which provides a user with a graphical interface to connect to another computer over a network connection. </br>
</br>
Virtual Machine (VM) - a digital version of a physical computer that can run in a cloud. </br>


<h2>Lab Walkthrough</h2>

To begin the lab, I first deployed a virtual machine  in Azure. The virtual machine was named "honeypotvm" and deployed in the West US 3 region. It was imaged with Windows 10 Pro, version 22H2.
</br>

<p align="center">
Azure Virtual Machine: <br/>
<img src="https://i.imgur.com/fRF2hfr.png" 
  </p>
</br>

I then created my admin login with username "brianadmin" and allowed RDP (port 3389) to remotley connect to the virtual machine.
</br>

<p align="center">
Azure Virtual Machine: <br/>
<img src="https://i.imgur.com/2kUeMs3.png"
  </p>
</br>

I created a custom network security group that acts as the virtual machines firewall. The inbound securty rule is set to allow TCP, UDP and ICMP requests to reach the virtual machine. This inbound rule will allow the vm to be found by the rest of the world.
</br>
<p align="center">
Azure Virtual Machine: <br/>
<img src="https://i.imgur.com/aWEysC0.png"
  </p>
</br>

I then navigated to "Microsoft Defender for Cloud | Environment settings" and toggled All Events. This configuration will allow Microsoft Defender on the VM to store all Windows Security events. These events will display failed logon attempts. 
</br>
<p align="center">
Azure Virtual Machine: <br/>
<img src="https://i.imgur.com/cJcQpYL.png"
  </p>
</br>

The VM was provisioned a public IP address of 20.120.88.213. This allowed me connect to the VM through Remote Desktop using my "brianadmin" credentials. 
</br>
<p align="center">
Azure Virtual Machine: <br/>
<img src="https://i.imgur.com/o8sRVCC.png"
  </p>
</br>

While the virtual machine was spinning up for the first time, I navigated to "Log Analytics Workspace > log-honeypot | Virutal machines" to establish a connection between Log Analytics Workspace and the honeypot VM. This connection installs a monitoring client on the VM that will export the logs back to Log Analytics Workspace for further analysis. 
</br>
<p align="center">
Log Analytics Workspace: <br/>
<img src="https://i.imgur.com/n1fQGH2.png"
  </p>
</br>

I then navigated to Microsoft Sentinel and configured a connection to "log-honeypot" where Log Analytics Workspace will be storing the logs from the VM. With this connection established, Microsoft Sentinel is able to contextualize the data for easier interpretation.
</br>
<p align="center">
Microsoft Sentinel: <br/>
<img src="https://i.imgur.com/aDHmQfh.png"
  </p>
</br>

With Log Analytics Workspace and Microsoft Sentinel set up, I swtiched back to the Remote Desktop session. On the honeypot VM, I disabled the local Windows Defender Firewall to allow ICMP echo request to reach the vm. This enables the VM to be globally discoverd by ping scans. 
</br>
<p align="center">
Azure Virtual Machine: <br/>
<img src="https://i.imgur.com/72Ce4OP.png"
  </p>
  <p align="center">
<img src="https://i.imgur.com/BxSdwEt.png"
  </p>
     <p align="center">
<img src="https://i.imgur.com/AKIAK2w.png"
  </p>
</br>

Now with the firewall disabled and the VM discoverable, I ran the Powershell script included in [Josh Madakor's tutorial](https://github.com/joshmadakor1/Sentinel-Lab/blob/main/Custom_Security_Log_Exporter.ps1). This Powershell script filters out the failed RDP attempts and then exports the logs to ipgeolocation.io. ipgeolocation.io is a API that is able to provide country, city, state, province, local, latitude and longitude, country calling code, time zone and much more information all based on the IP address it's given. </br>
The script then writes the data provided by ipgeolocation.io to a log file located at _C:\ProgramData\failed_rdp.log_ on the VM. </br>
</br>
With the script running I noticed that the honeypot VM was already logging failed RDP attempts from an entity in Guangxi, China.
</br>
<p align="center">
Failed RDP Attempts: <br/>
<img src="https://i.imgur.com/et5Dadr.png"
  </p>
</br>

The entity in Guangxi, China is conducting a bruteforce attack. This is evident in the different usernames they were trying.</br>
Some of the attempted usersnames are as follows:</br>
- _admin_ </br>
- _Administrator_ </br>
- _lenovo_ </br>
- _WIN-CQ312NK70UD\mazq_ </br>
</br>
<p align="center">
Failed RDP Attempts: <br/>
<img src="https://i.imgur.com/jLK7JjC.png"
  </p>
</br>

They even tried logging in with Hanzi characters as captured below. 
</br>
<p align="center">
Failed RDP Attempts: <br/>
<img src="https://i.imgur.com/0xmFXTP.png"
  </p>
</br>

With the Powershell script successfully reaching out to the ipgeolocation.io API and logging the data, I returned back to Log Analytics Workspace to set up a custom table. This table will look for the log file generated by the script located at _C:\ProgramData\failed_rdp.log_ on the honeypot VM. The custom table was named **FAILED_RDP_HP**. </br>
</br>
<p align="center">
Custom Table: <br/>
<img src="https://i.imgur.com/998CXj5.png"
  </p>
  <p align="center">
<img src="https://i.imgur.com/2dUWFrA.png"
  </p>
    <p align="center">
<img src="https://i.imgur.com/byTghRd.png"
  </p>
  
</br>

After about 30 minutes, the logs from the _failed_rdp.log_ file were imported into the **FAILED_RDP_HP** custom table. 
</br>
<p align="center">
FAILED_RDP_HP: <br/>
<img src="https://i.imgur.com/eVz5li2.png"
  </p>
</br>

I then ran a KQL query to parse the RawData column. The query sorted the data into: _Latitude, Longitude, DestinationHost, Username, Sourcehost, State, Country, Label, and Timestamp_. 
</br>
<p align="center">
FAILED_RDP_HP: <br/>
<img src="https://i.imgur.com/FETkygR.png"
  </p>
</br>

After parsing the data in the **FAILED_RDP_HP table**, I switched back to Microsoft Sentinel and createed a workbook. This workbook allowed me to plot the data from the table onto a world map and get a visual representation of where the failed RDP attempts came from. To format the workbook, I ran another KQL query to sort the data. 
</br>
<p align="center">
Sentinel Workbook: <br/>
<img src="https://i.imgur.com/dYtNb5Z.png"
  </p>
  <p align="center">
<img src="https://i.imgur.com/cYKLkpN.png"
  </p>
</br>

With the data sorted, I was now able to plot the data onto a world map. I named the Sentinel Workbook **FAILED_RDP_WORDLMAP**</br>
As seen below, there were 351 failed RDP attempts from Guangxi, China, 1 failed attempt from Central Federal District, Russia and 1 failed attempt from Colorado, United States. All attempts coming in after 30 minutes of deploying the honeypot VM.
</br>
<p align="center">
FAILED_RDP_WORLDMAP: <br/>
<img src="https://i.imgur.com/csRiQQD.png"
  </p>
</br>

I left the honeypot running for roughly 24 hours to observe what other countries would try to log into the VM. The results are shown below:
</br>
<p align="center">
FAILED_RDP_WORLDMAP: <br/>
<img src="https://i.imgur.com/57xvR5u.png"
  </p>
  
</br>
</br>

The total number of failed RDP attempts by location are as follows:</br>
- Gaborone, Botswana - 2.93k </br>
- Maharashtra, India - 2.68k </br>
- Guangxi, China - 351 </br>
- California, United States - 296 </br>
- Amman, Jordan - 88 </br>
- Nevada, United States - 55 </br>
- Goias, Brazil - 30 </br>
- Virginia, United States - 19 </br>
- England, United Kingdom - 14 </br>
- Ankara, Turkey - 12 </br>
- Colorado, United States - 2 </br>
- Belize District, Belize - 2 </br>
- Central Federal District, Russia - 1 </br>
- North Holland, Netherlands - 2 </br>
- Sao Paulo, Brazil - 1 </br>
- Oregon, United States - 1 </br>
- Northwestern Federal District, Russia - 1 </br>
</br>

<h2>Analysis</h2>

With the conclusion of the lab, we can see that it doesn't take very long for a device to be found on the internet, especially if it is not properly hardened. The honeypot vm recieved a total of 6496 failed RDP connection attempts within the 24 hours that it was powered on. </br>
</br>
Of those 6496 attempts, 5712 used administrator as the username. </br>
</br>
It is crucial that organizations properly harden their devices by limiting inbound communication from external sources. As well as disabling default accounts like administrator. An attacker is eventually going to get in if they have enough time and resources. 


