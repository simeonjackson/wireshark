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

![Screenshot 2025-02-26 001438](https://github.com/user-attachments/assets/18fb7cdf-97b5-45fe-969c-1ea7c4c54160)


<p>
Opening up Wireshark with its default settings gives a packet capture file (pcap) showing enumerable IP packets and activity going across the network. In this lab I am going to filter down the traffic to a more digestible, easy to read size showing common protocols that you would come across analyzing traffic.
</p>

![Screenshot 2025-02-26 002123](https://github.com/user-attachments/assets/f6459587-1348-4cfd-b044-319004cafa69)


<p>
The first protocol I looked at was ICMP which is commonly used to "ping" devices to report errors, troubleshoot and communicate diagnostic information. Using the Windows VM I sent messages to the private IP of the Linux VM and used Wireshark to see this activity.
</p>

![Screenshot 2025-02-26 003424](https://github.com/user-attachments/assets/b2c3dcbb-14d8-4abd-9c9b-21544126ee4f)


<p>
PowerShell shows four successful ping attempts and the Wireshark data confirms this with found pairs of request and reply messages. Diving deeper into Wireshark you can see more information on the packets we just captured such as the length of the packet, the source and destination IP and MAC addresses, and even what was sent in the payload.
</p>

![Screenshot 2025-02-26 004422](https://github.com/user-attachments/assets/0393bfce-b60c-4fe5-b440-5e6640b69a08)


<p>
Next, I sent a continuous ping to the Linux VM using "ping 10.0.0.5 -t". Instead of sending for echo request, now the Windows VM is sending continuous requests. Which is shown on Wireshark and PowerShell below.
</p>

![Screenshot 2025-02-26 005021](https://github.com/user-attachments/assets/9c302333-dfe0-4f5a-a0fa-75538241ac81)


<p>
In a "simulation" of an administrator setting Firewall rules I created a rule to block inbound ICMP traffic to the Linux VM using Azure's Network Security Groups. This could be done by an organization to reduce attack surface by blocking ping sweeps or ping flood which attackers may use to identify active hosts or initiate DoS attacks.

Going into the network settings of the Linux VM I created an inbound security rule  blocking all ICMP traffic from any source. The port field is left blank because ICMP does not use a port. I set the priority to be 290 which is the lowest on the default list. NSG security rules start from lowest priority and work their way down the list so this rule has the highest priority.
</p>

![Screenshot 2025-02-26 010621](https://github.com/user-attachments/assets/ec557ba6-9abb-4e36-b4a3-e94bd95babf7)


<p>
Looking back in PowerShell and Wireshark, you can see that our ping requests have now started to time out meaning that the new security rule is in effect.
</p>

![Screenshot 2025-02-26 010954](https://github.com/user-attachments/assets/e4cd30eb-028a-48d9-a3fb-ff25500171ea)


<p>
Next I filtered Wireshark to observe only SSH traffic.  SSH (Secure Shell) is used to make a secure connection from one computer to another computer via the command line. In this case I created a SSH connection from my Windows VM to the Linux VM.
</p>

![Screenshot 2025-02-26 170638](https://github.com/user-attachments/assets/0e9664b2-73a0-478d-93ff-78483c77920d)


<p>
I did this in PowerShell using the private IP address of my Linux VM and the "ssh" command as follows:

	ssh simeonlab@10.0.0.5

Note that Wireshark now shows the SSH traffic that we just generated.
</p>

![Screenshot 2025-02-26 170856](https://github.com/user-attachments/assets/1d5fe2dd-16a6-4b2a-a067-90250f813f7c)


<p>
After completing my login to the Linux VM I have created a secure connection from my Windows VM to my Linux VM. Any command that I put into PowerShell will show up on my Wireshark network capture. Since I used SSH instead of Telnet (which is not secure and sends plaintext) the traffic that Wireshark picks up is encrypted.
</p>

![Screenshot 2025-02-26 171910](https://github.com/user-attachments/assets/b05f1c04-8cef-4d83-9337-275503eb6944)


<p>
Next I looked at DHCP traffic. DHCP is used to assign IP addresses to devices when they join the network. DHCP uses UDP ports 67 and 68.
</p>

![Screenshot 2025-02-26 172115](https://github.com/user-attachments/assets/2d9fa67a-a27d-4042-bf92-628972d0346c)


<p>
Using the ipconfig release and renew commands my Windows machine will forget its private IP address and then send a broadcast message to the DHCP server requesting a new IP address. All of this traffic can be seen and read in Wireshark which shows the DHCP release, discover, offer, and request packets.
</p>

![Screenshot 2025-02-26 172751](https://github.com/user-attachments/assets/8dee6842-70be-4deb-815e-003960fce6f7)


<p>
DHCP Release: Our Windows VM releases its current IP address
  
DHCP Discover: Our device now with (0.0.0.0 as IP) sends a broadcast to find DHCP server

DHCP Offer: The DHCP server (server IP) proposes an IP address

DHCP Request: Our VM accepts the proposed IP

DHCP ACK: The DHCP server acknowledges the new IP assignment


I filtered for DNS traffic on Wireshark which after using a command like nslookup will generate traffic. DNS uses port 53 on both UDP and TCP.
</p>

![Screenshot 2025-02-26 172948](https://github.com/user-attachments/assets/2164c8de-a347-44cd-a7a1-c6528ecbddc4)


<p>
The last port I looked at on Wireshark was TCP 3389 which the Remote Desktop Protocol (RDP) uses. Note the non-stop traffic being captured. This is because I am connected to my VM via RDP.
</p>

![Screenshot 2025-02-26 173133](https://github.com/user-attachments/assets/6eb47685-8637-4fbb-b377-cebe356a9c39)


<p>
Thanks for reading my basic overview of how Wireshark can be used to filter and analyze network traffic for some common protocols and ports.
</p>
