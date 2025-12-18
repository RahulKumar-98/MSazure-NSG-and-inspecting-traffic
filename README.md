<p align="center">
<img src="https://i.imgur.com/Ua7udoS.png" alt="Traffic Examination"/>
</p>

<h1>Network Security Groups (NSGs) and Inspecting Traffic Between Azure Virtual Machines</h1>
In this tutorial, we observe various network traffic to and from Azure Virtual Machines with Wireshark as well as experiment with Network Security Groups. <br />


<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Various Command-Line Tools
- Various Network Protocols (SSH, RDH, DNS, HTTP/S, ICMP)
- Wireshark (Protocol Analyzer)

<h2>Operating Systems Used </h2>

- Windows 10 (22H2)
- Ubuntu Server 22.04


<h2>Actions and Observations</h2>

<img width="1546" height="301" alt="1" src="https://github.com/user-attachments/assets/5dc6db1a-67fa-430e-867e-b8d0d99b3de2" />

<p>
In this procedure, we will configure Network Security Groups and observing network traffic within Microsoft Azure. To get started, we will create 2 virtual machines. One Virtual machine will be running Windows 10, and the other will be running Linux. When creating the VMs, we will ensure both virtual machines are under the same network, region, resource group and contain at least 2 Vcpus for the size. (Reference the following for virtual machine set-up:  -- <a href="https://github.com/RahulKumar-98/MSAzure-activedirectory">On-premises Active Directory Deployed in the Cloud (Azure).
</p>
<br />

<p>
  <img width="863" height="718" alt="2" src="https://github.com/user-attachments/assets/68ad5c69-2f9f-4367-bdfe-7cacbf68be36" />

</p>
<p>
Next we will log into the windows virtual machine and install  -- <a href="https://www.wireshark.org">Wireshark. 
</p>
<p> First, we will copy the link and install Wireshark in our virtual machine (we will be installing the Windows x64 bit installer). This application allows us to capture live packets of data being transferred between Azure's virtual environment and the network, primarily through ports and protocols.</p>
<br />

</p>
<p>
  
  <img width="939" height="670" alt="3" src="https://github.com/user-attachments/assets/278b8eff-63ce-4bdf-870a-69f1caa43614" />

Once Wireshark has been installed, we will observe ICMP traffic to examine ICMP traffic coming from and coming in to our virtual machine. This will will allow us to inspect connectivity between devices when using "ping". To do so, click on "Ethernet" -> enter in "ICMP" in the search bar within wireshark.
</p>
<br />


<p>
  <img width="942" height="748" alt="4" src="https://github.com/user-attachments/assets/e6d3ebe1-edf1-4229-8ce7-910ccf72b89e" />

Next, we will copy the Linux VM's private IP address from Azure and attempt to ping it from our Windows VM. This allows us to see the ICMP traffic in Wireshark. When filtering for ICMP traffic in wireshark, we notice request and reply packets being sent and received from both source and destination addresses. Wireshark captures source and destination MAC addresses, data payload, and other protocol relevant information.
</p>
<br />

<p>
<img width="654" height="265" alt="5" src="https://github.com/user-attachments/assets/0b66877e-ebd7-45f0-b383-3a051ce2fb87" />

</p>
<p>
To block specific traffic from being communicated within the network, we can configure security rules in Azure. In this example, we will block ICMP traffic from the Linux computer. To do this, we will first ping the Linux VM non-stop, by using the command "Ping-t (private IP of Linux VM) from our Windows VM.
</p>
<br />

<p>
<img width="1962" height="1183" alt="5" src="https://github.com/user-attachments/assets/fda35fd0-745c-4e09-ad9f-92be53e9f1b1" />

</p>
<p>
Navigate to Azure, Click on Linux VM under virtual machines -> Network Settings -> Scroll down to the Network Security Group portion and click "Create Port Rule" -> Click "Add In-bound security rules" -> Under Source Port Ranges, enter "*" since ICMP does not have a port -> select ICMP protocol -> Set Priority to "290" -> Click add.
</p>
<br />

<p>
<img width="1259" height="1143" alt="6" src="https://github.com/user-attachments/assets/e0b3ab0e-0783-48b9-a4f7-c8279c3a7843" />

</p>
<p>
Back in the Windows VM, we can observe the request's from the Window's VM to the Linux VM outgoing, but no reply being sent back. Additionally, powershell shows a "Request timed out" message as the ping to the Linux VM is not responsive anymore. In Wireshark, we can see "No  response seen" message under the Internet Control Message Protocol, and the Echo Pings are primarily showing requests now, thus indicating the ICMP traffic between the two machines has been blocked per out in-bound NSG settings in our Linux-VM. We can delete this protocol rule to enable the ICMP traffic again, if we wish to do so. 
</p>
<br />

<p>
<img width="1244" height="1228" alt="7" src="https://github.com/user-attachments/assets/54d710cf-5ce5-414d-9ebc-a145e456acef" />

</p>
<p>
Continuing our packet capture analysis in Wireshark, we will now observe Secure Shell Protocol (SSH) traffic. This will require us to connect to the Linux VM on our Window's VM through powershell. To do this, filter on SSH traffic in Wireshark -> Open Powershell -> enter "SSH (username of Linux VM)@ Private IP of Linux VM". Key Note: When using SSH and performing a keystroke in powershell, the packet is shown and captured in wireshark, on a "per-keystroke" level. To exit SSH, type "exit" in Powershell.
</p>
<br />

<p>
<img width="864" height="965" alt="8" src="https://github.com/user-attachments/assets/d350bd43-ad9a-467e-b807-0f09b10b04c6" />

</p>
<p>
To observe DHCP traffic in Wireshark, we will type "ipconfig/ release" to release the DHCP lease containing our current IP address, from there we will type "ipconfig/ renew" to renew our IP address. This allows us to refresh and assign our computer a new IP from the DHCP server. Next we will enter DHCP in the Wireshark search box. Observing the above DHCP traffic, we can see the Ipconfig release command being communicated to the DHCP server, which had released the current IP address of our Windows machine, resulting in an IP address if 0.0.0.0."Ipconfig /renew" had then been executed, therefore, a broadcast was sent to the DHCP server "255.255.255.255", which had resulted in the DHCP server sending my computer an offer. Which, my computer (Windows-vm) had requested a new IP thereafter. DHCP acknowledged the request as shown in the packet capture on Wireshark.</p>
<br />

</p>
<br />

  
  <img width="870" height="773" alt="10" src="https://github.com/user-attachments/assets/4fe9f2c0-4cee-48e2-9b0a-0f386341229c" />

Now we can move onto observing DNS traffic. To do this, we will filter/ enter "DNS" in the search box. Next we can perform an "nslookup (website)" command in powershell to find the IP address for a specific IP address. In this example, we used "Google.com". We can observe this activity in wireshark and see the back-end communication happening on the DNS server (TCP/UDP port 53).
</p>
<br />

<p>
  <img width="998" height="236" alt="11" src="https://github.com/user-attachments/assets/fb77e6ad-21a2-4f17-89ee-b6d5030cd699" />

</p>
<p>
Lastly, to analyze Remote Desktop traffic on our Azure VM, we simply type in the Wireshark search box "tcp.port == 3389", this focuses the packet capture primarily on RDP Protocol, which displays on-going live packet capture. This is because we are connected to Azure's virtual machine, which is constantly streaming images from the virtual machine on to our local computer. This concludes this procedure on Network Security Groups and inspecting traffic within wireshark.
</p>
<br />
