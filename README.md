# Inter-VLAN-troubleshooting P2
<img width="522" height="366" alt="image" src="https://github.com/user-attachments/assets/01d6726b-60a9-4a23-941f-e508577faf98" />

Scenarios for  enterprise inter VLAN support


## 🎫 Incident Ticket #INC-00851
**Priority:** `P2 — High` **Reported by:** `Marcus L., Sales ` **Assigned to:** `Network Support` **Time logged:** `02:48`
<img width="500" height="500" alt="image" src="https://github.com/user-attachments/assets/f7db55e1-382c-4076-b515-5469f829de1c" />

**<ins>Ticket Key Points:</ins>**
    
- HR can reach IT, but IT can't reach HR
- Both departments getting IP addresses - DHCP is fine?
 
<br>

**<ins>Step 1: Layer 1</ins>**
    
- I started troubleshooting systematically at Layer 1.
- Checked cabling on every connection, to make sure right cables were used.
- The next step was the interfaces, to check for possible **down interfaces**. I useed `show ip interface brief` on the two swtiches, as well as the layer 3 switch.
- The end device  and D1 connected interfaces where up and connected. The **SVIs** for D1 where also up.

<img width="797" height="120" alt="image" src="https://github.com/user-attachments/assets/47db9b83-8186-4407-b1ba-6286cfdc5590" />



**<ins>Step 2: Layer 2 & L3</ins>**
- I then went to check the VLANs. I typed `show vlan` on **SW-HR**, i saw 3 interfaces assigned to vlan 5 (HR vlan). And the trunk interface Gig 0/1 was not assigned to any vlans.
- I typed show vlan on D1, and D1 knew both vlans, so it wasn't an issue here.
Incident ticket says, when HR tries to ping, they get nothing. Could they have the wrong ip address?

- So i went to check DHCP pool for IP addresses to see if the Pool addresses matched the end device Ips
<img width="677" height="356" alt="image" src="https://github.com/user-attachments/assets/94a4cfed-e88f-41c0-b342-8e39e774bc5f" />

The vlan corresponding to HR was not leased, but the IT vlan was...
- THe first step was to elimnate any hardware issues, I went to the PCs themselves and reset the DHCP by clicking static then clicking back to DHCP
<img width="549" height="409" alt="image" src="https://github.com/user-attachments/assets/29d4b2e9-7356-4190-9150-0c1b49b8e35c" />
I waited for a while, but it did not update.
- I then thought that it's definitely a default gatewat issue, if the PC is not getting the DHCP address and i can's see any L2 issues, it must be this
- I typed `show running config` on D1, and unfortunately, the default gateway was set correctly. I was stuck!
- I then researched and fed all my findings into search, to get back possible causes. The only one that matched my config was **subnet misconfiguration**
- I typed the command `show running-config |section interface vlan` to check the subnet of the **SVIs**, and I saw that the HR vlan has a /25 subnet mask instead of a /24.
- This explained everything, and my assumption that they had the wrong addresses was correct, as the /25 chnages the network completey
- To resolve this, I typed on D1 in global configuration mode `interface vlan5`, followed by the ip ddress and hthe correct subnet
- After commiting my config, I went back to the computers to reset their DHCP, and their addresses were now correct.
<img width="386" height="135" alt="image" src="https://github.com/user-attachments/assets/41fa6aa3-fab3-46ae-a3dc-4d056f04d1c8" />
<img width="574" height="106" alt="image" src="https://github.com/user-attachments/assets/f33922fa-0395-436a-95a2-4425d4748ac1" />

<img width="565" height="211" alt="image" src="https://github.com/user-attachments/assets/15ccc39a-10d7-4d41-a7ae-cbd026577ba8" />


**<ins>Ticket Key Points Review:</ins>**
    
- ~~Finance gets IP addresses, but IPs are strange, don't belong to expected subnet~~ Ip issue has been resolved, dhcp pooling works due to corrected VLAN mismatch, finance ips have the correct IPs.
- ~~Finance pings to OPs work sometimes, but inconsistent~~ inter vlan connectivity works, fully functional pings are displayed in the screenshots.
  

