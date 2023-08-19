<h1>Honey Pot Lab</h1>

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

Now with the firewall disabled and the vm discoverable, I ran the Powershell script included in Josh Madakor's tutorial. This powershell script filters out the failed RDP attempts and then exports the logs to ipgeolocation.io. ipgeolocation.io is a API that is able to provide country, city, state, province, local, latitude and longitude, country calling code, time zone and much more information all based on the IP address it's given. </br>
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
