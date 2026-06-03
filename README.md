# Inter-VLAN-troubleshooting P2
<img width="522" height="366" alt="image" src="https://github.com/user-attachments/assets/01d6726b-60a9-4a23-941f-e508577faf98" />

Scenarios for enterprise inter-VLAN support

## 🎫 Incident Ticket #INC-00851
**Priority:** `P2 — High` **Reported by:** `Marcus L., Sales` **Assigned to:** `Network Support` **Time logged:** `02:48`
<img width="500" height="500" alt="image" src="https://github.com/user-attachments/assets/f7db55e1-382c-4076-b515-5469f829de1c" />

**<ins>Ticket Key Points:</ins>**
    
- HR can reach IT, but IT can't reach HR
- Both departments are getting IP addresses — DHCP is fine?
 
<br>

**<ins>Step 1: Layer 1</ins>**
    
- I started troubleshooting systematically at Layer 1.
- Checked cabling on every connection to make sure the right cables were used.
- The next step was the interfaces, to check for possible **down interfaces**. I used `show ip interface brief` on the two switches, as well as the Layer 3 switch.
- The end device and D1 connected interfaces were up and connected. The **SVIs** for D1 were also up.

<img width="797" height="120" alt="image" src="https://github.com/user-attachments/assets/47db9b83-8186-4407-b1ba-6286cfdc5590" />

**<ins>Step 2: Layer 2 & L3</ins>**
- I then went to check the VLANs. I typed `show vlan` on **SW-HR**; I saw 3 interfaces assigned to VLAN 5 (HR VLAN), and the trunk interface Gig 0/1 was not assigned to any VLANs.
- I typed `show vlan` on D1, and D1 knew both VLANs, so it wasn't an issue here.
The incident ticket states that when HR tries to ping, they get nothing. Could they have the wrong IP address?
- So I went to check the DHCP pool to see if the pool addresses matched the end device IPs.

<img width="677" height="356" alt="image" src="https://github.com/user-attachments/assets/94a4cfed-e88f-41c0-b342-8e39e774bc5f" />

The VLAN corresponding to HR was not leased, but the IT VLAN was.
- The first step was to eliminate any hardware issues. I went to the PCs themselves and reset the DHCP by clicking static, then clicking back to DHCP.
  
<img width="549" height="409" alt="image" src="https://github.com/user-attachments/assets/29d4b2e9-7356-4190-9150-0c1b49b8e35c" />

I waited for a while, but it did not update.
- I then thought that it was definitely a default gateway issue — if the PC is not getting the DHCP address and I can't see any L2 issues, it must be this.
- I typed `show running config` on D1, and unfortunately, the default gateway was set correctly. I was stuck!
- I then researched and fed all my findings into a search to get back possible causes. The only one that matched my config was **subnet misconfiguration**.
- I typed the command `show running-config |section interface vlan` to check the subnet of the **SVIs**, and I saw that the HR VLAN had a /25 subnet mask instead of a /24.

<img width="386" height="135" alt="image" src="https://github.com/user-attachments/assets/41fa6aa3-fab3-46ae-a3dc-4d056f04d1c8" />

- This explained everything, and my assumption that they had the wrong addresses was correct, as the /25 changes the network completely.
- To resolve this, I typed on D1 in global configuration mode `interface vlan5`, followed by the IP address and the correct subnet.
- After committing my config, I went back to the computers to reset their DHCP, and their addresses were now correct.

<img width="574" height="106" alt="image" src="https://github.com/user-attachments/assets/f33922fa-0395-436a-95a2-4425d4748ac1" />
<img width="565" height="211" alt="image" src="https://github.com/user-attachments/assets/15ccc39a-10d7-4d41-a7ae-cbd026577ba8" />

**<ins>Ticket Key Points Review:</ins>**
    
- ~~HR can reach IT, but IT can't reach HR~~ HR could not be reached as IT had the wrong Ip addresses, the subnet was incorrect so IT were pinging the right address but HR didn't have it.
- ~~- Both departments are getting IP addresses — DHCP is fine?~~ DHCP was not fine for HR. They had IP addresses, but not the right ones.
