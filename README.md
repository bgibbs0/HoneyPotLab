<h1>Azure Honeypot Lab</h1>

<h2>Description</h2>
This project consists of spinning up a virtual machine in Microsoft Azure that will be used as a honeypot. The honeypot VM will be connected to Log Analytics Workspace and Azure Sentinel (SIEM). These connections will populate the failed RDP (Remote Desktop) logs into Microsoft Sentinel, allowing us to plot the failed connection attempts on a world map. This lab uses Josh Madakor's <a href=https://youtu.be/RoZeVbbZ0o0">SIEM tutorial video</a> as guidance.
<br/>

<h2>Languages Used</h2>
- Powershell: Used to extract Windows Security logs</br>  
- Kusto Query Language (KQL): Used to parse raw data from Windows Security logs</br>

<h2>Utilities Used</h2>
- Microsoft Azure: Used to deploy honeypot virtual machine</br>
- Microsoft Sentinel: Azure Cloud SIEM that populated the honeypots security alerts</br>
- Log Analytics Workspace: Azure service that stores the extracted log data</br>
- ipgeolocation.io: API that provided IP geolocation data from security logs</br>

<h2>Lab Walkthrough</h2>

To begin the lab, I first deployed a virtual machine in Azure. The virtual machine was named "honeypotvm" and deployed in the West US 3 region. It was imaged with 
Windows 10 Pro, version 22H2.
</br>

<p align="center">
Azure Virtual Machine: <br/>
<img src="https://i.imgur.com/fRF2hfr.png" 
  </p>
</br>

I then created my admin login with username "brianadmin" and allowed RDP (3389) to remotley connect to the virtual machine.
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

I then navigated to "Microsoft Defender for Cloud | Environment settings" and toggled All Events. This configuration will allow Microsoft Defender on the vm to store all Windows Security events.
</br>
<p align="center">
Azure Virtual Machine: <br/>
<img src="https://i.imgur.com/cJcQpYL.png"
  </p>
</br>

With the virutal machine deployed, I was able to connect through a Remote Desktop session. The vm was provisioned a public IP address of 20.120.88.213. I logged in with my username brianadmin. 
</br>
<p align="center">
Azure Virtual Machine: <br/>
<img src="https://i.imgur.com/o8sRVCC.png"
  </p>
</br>

While the virtual machine was spinning up for the first time, I navigated to "Log Analytics Workspace > log-honeypot | Virutal machines" to set up a connection between Log Analytics Workspace and the honeypot vm. This connection installs a monitoring client on the vm that will export the logs back to Log Analytics Workspace. 
</br>
<p align="center">
Log Analytics Workspace: <br/>
<img src="https://i.imgur.com/n1fQGH2.png"
  </p>
</br>

I then navigated to Microsoft Sentinel and configured a connection to "log-honeypot" where Log Analytics Workspace will be storing the logs from the vm. 
</br>
<p align="center">
Microsoft Sentinel: <br/>
<img src="https://i.imgur.com/aDHmQfh.png"
  </p>
</br>

With Log Analytics Workspace and Microsoft Sentinel set up, I swtiched back to the Remote Desktop session. On the honeypot vm, I disabled the local Windows Defender Firewall to allow ICMP echo request to reach the vm. This enables the vm to be globally discoverd by a ping scan. 
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

Now with the firewall disabled and the vm discoverable, I ran the Powershell script included in [Josh Madakor's tutorial](https://github.com/joshmadakor1/Sentinel-Lab/blob/main/Custom_Security_Log_Exporter.ps1). This powershell script filters out the failed RDP attempts and then exports the logs to ipgeolocation.io. ipgeolocation.io is a API that is able to provide country, city, state, province, local, latitude and longitude, country calling code, time zone and much more information all based on the IP address it's given. </br>
The script then writes the data provided by ipgeolocation.io to a log file located at "C:\ProgramData\failed_rdp.log" on the vm. </br>
</br>
With the script running we can that the honeypot vm was already getting failed RDP attempts from an entity in Guangxi, China.
</br>
<p align="center">
Failed RDP Attempts: <br/>
<img src="https://i.imgur.com/et5Dadr.png"
  </p>
</br>

The entity in Guangxi, China is conducting a bruteforce login attempt. This is evident in the different usernames they were trying.</br>
Some of the attempted usersnames are as follows:</br>
- admin</br>
- Administrator</br>
- lenovo</br>
- WIN-CQ312NK70UD\mazq</br>
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

Now that the Powershell script was successfully reaching out to the ipgeolocation.io API and logging the data. I returned back to Log Analytics Workspace to set up a custom table. This table will look for the log file generated by the script located at C:\ProgramData\failed_rdp.log on the honeypot vm. The custom table was named FAILED_RDP_HP and will be constantly updated with new data written to the failed_rdp.log file. </br>
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

After about 30 minutes, the data from the failed_rdp.log file was imported into the FAILED_RDP_HP custom table. The data that is needing to be extracted is located in the RawData column. 
</br>
<p align="center">
FAILED_RDP_HP: <br/>
<img src="https://i.imgur.com/eVz5li2.png"
  </p>
</br>

I now run a KQL query to parse the RawData column. The query will sort the data into: Latitude, Longitude, DestinationHost, Username, Sourcehost, State, Country, Label, and Timestamp. 
</br>
<p align="center">
FAILED_RDP_HP: <br/>
<img src="https://i.imgur.com/FETkygR.png"
  </p>
</br>

After parsing the data in the FAILED_RDP_HP table, I switch to Microsoft Sentinel and create a workbook. This workbook allows me to plot the data from the table onto a world map and get a visual representation of where the failed RDP attempts are coming from. To create the workbook, I run another KQL query to sort the data. 
</br>
<p align="center">
Sentinel Workbook: <br/>
<img src="https://i.imgur.com/dYtNb5Z.png"
  </p>
  <p align="center">
<img src="https://i.imgur.com/cYKLkpN.png"
  </p>
</br>

With the data sorted, I am now able to plot the data onto a world map. I named the Sentinel Workbook "FAILED_RDP_WORDLMAP"</br>
As seen below, there are 351 failed RDP attempts from Guangxi, China, 1 failed attempt from Central Federal District, Russia and 1 failed attempt from Colorado, United States. All attempts coming in after 30 minutes of deploying the honeypot vm.
</br>
<p align="center">
FAILED_RDP_WORLDMAP: <br/>
<img src="https://i.imgur.com/csRiQQD.png"
  </p>
</br>

I left the honeypot running for roughly 24 hours to observe what other countries would try to log into the vm. The results are shown below:
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


