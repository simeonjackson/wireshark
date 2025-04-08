<h1>Network Analysis with Wireshark</h1>
This tutorial outlines the basics of how Wireshark can be used to analyze network traffic looking at some very common protocols.  <br />


<h2>Environments and Technologies Used</h2>

- Wireshark
- Microsoft Azure
- Remote Desktop
- PowerShell

<h2>Operating Systems Used </h2>

- Windows 11
- Ubuntu Linux

<h2>Deployment and Configuration Steps</h2>

<p>
I created a basic network and used Wireshark to analyze traffic flow between two VMs.

In order to do this I created a resource group with two VMs (WindowsVM and LinuxVM) and placed them on the same virtual network (Wireshark-vnet) and subnet.
</p>

![Screenshot 2025-02-24 015152](https://github.com/user-attachments/assets/e8f0537d-5916-468e-80d7-7c5ad0946553)


<p>
Using Remote Desktop Protocol (RDP) I will connect to the Windows VM which has Wireshark installed onto the desktop.
</p>
<p>
<img src="https://i.imgur.com/DJmEXEB.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>

<p>
Opening up Wireshark with its default settings gives a packet capture file (pcap) showing enumerable IP packets and activity going across the network. In this lab I am going to filter down the traffic to a more digestible, easy to read size showing common protocols that you would come across analyzing traffic.
</p>
<p>
<img src="https://i.imgur.com/DJmEXEB.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>

<p>
The first protocol I looked at was ICMP which is commonly used to "ping" devices to report errors, troubleshoot and communicate diagnostic information. Using the Windows VM I sent messages to the private IP of the Linux VM and used Wireshark to see this activity.
</p>
<p>
<img src="https://i.imgur.com/DJmEXEB.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>

<p>
PowerShell shows four successful ping attempts and the Wireshark data confirms this with found pairs of request and reply messages. Diving deeper into Wireshark you can see more information on the packets we just captured such as the length of the packet, the source and destination IP and MAC addresses, and even what was sent in the payload.
</p>
<p>
<img src="https://i.imgur.com/DJmEXEB.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>

<p>
Next, I sent a continuous ping to the Linux VM using "ping 10.0.0.5 -t". Instead of sending for echo request, now the Windows VM is sending continuous requests. Which is shown on Wireshark and PowerShell below.
</p>
<p>
<img src="https://i.imgur.com/DJmEXEB.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>


<p>
In a "simulation" of an administrator setting Firewall rules I created a rule to block inbound ICMP traffic to the Linux VM using Azure's Network Security Groups. This could be done by an organization to reduce attack surface by blocking ping sweeps or ping flood which attackers may use to identify active hosts or initiate DoS attacks.

Going into the network settings of the Linux VM I created an inbound security rule  blocking all ICMP traffic from any source. The port field is left blank because ICMP does not use a port. I set the priority to be 290 which is the lowest on the default list. NSG security rules start from lowest priority and work their way down the list so this rule has the highest priority.
</p>
<p>
<img src="https://i.imgur.com/DJmEXEB.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>

<p>
Looking back in PowerShell and Wireshark, you can see that our ping requests have now started to time out meaning that the new security rule is in effect.
</p>
<p>
<img src="https://i.imgur.com/DJmEXEB.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>

<p>
Next I filtered Wireshark to observe only SSH traffic.  SSH (Secure Shell) is used to make a secure connection from one computer to another computer via the command line. In this case I created a SSH connection from my Windows VM to the Linux VM.
</p>
<p>
<img src="https://i.imgur.com/DJmEXEB.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>

<p>
I did this in PowerShell using the private IP address of my Linux VM and the "ssh" command as follows:

	ssh simeonlab@10.0.0.5

Note that Wireshark now shows the SSH traffic that we just generated.
</p>
<p>
<img src="https://i.imgur.com/DJmEXEB.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>

<p>
After completing my login to the Linux VM I have created a secure connection from my Windows VM to my Linux VM. Any command that I put into PowerShell will show up on my Wireshark network capture. Since I used SSH instead of Telnet (which is not secure and sends plaintext) the traffic that Wireshark picks up is encrypted.
</p>
<p>
<img src="https://i.imgur.com/DJmEXEB.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>

<p>
Next I looked at DHCP traffic. DHCP is used to assign IP addresses to devices when they join the network. DHCP uses UDP ports 67 and 68.
</p>
<p>
<img src="https://i.imgur.com/DJmEXEB.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>

<p>
Using the ipconfig release and renew commands my Windows machine will forget its private IP address and then send a broadcast message to the DHCP server requesting a new IP address. All of this traffic can be seen and read in Wireshark which shows the DHCP release, discover, offer, and request packets.
</p>
<p>
<img src="https://i.imgur.com/DJmEXEB.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>

<p>
DHCP Release: Our Windows VM releases its current IP address
  
DHCP Discover: Our device now with (0.0.0.0 as IP) sends a broadcast to find DHCP server

DHCP Offer: The DHCP server (server IP) proposes an IP address

DHCP Request: Our VM accepts the proposed IP

DHCP ACK: The DHCP server acknowledges the new IP assignment


I filtered for DNS traffic on Wireshark which after using a command like nslookup will generate traffic. DNS uses port 53 on both UDP and TCP.
</p>
<p>
<img src="https://i.imgur.com/DJmEXEB.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>

<p>
The last port I looked at on Wireshark was TCP 3389 which the Remote Desktop Protocol (RDP) uses. Note the non-stop traffic being captured. This is because I am connected to my VM via RDP.
</p>
<p>
<img src="https://i.imgur.com/DJmEXEB.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>

<p>
Thanks for reading my basic overview of how Wireshark can be used to filter and analyze network traffic for some common protocols and ports.
</p>
