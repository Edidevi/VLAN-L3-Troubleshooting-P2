# Inter-VLAN-troubleshooting
<img width="522" height="366" alt="image" src="https://github.com/user-attachments/assets/01d6726b-60a9-4a23-941f-e508577faf98" />

Scenarios for  enterprise inter VLAN support


## 🎫 Incident Ticket #INC-00851
**Priority:** `P2 — High` **Reported by:** `James T. IT` **Assigned to:** `Network Support` **Time logged:** `09:15 AM`
<img width="500" height="500" alt="image" src="https://github.com/user-attachments/assets/f7db55e1-382c-4076-b515-5469f829de1c" />

**<ins>Ticket Key Points:</ins>**
    
- It can reach HR, but HR can't reach IT
- Both departments getting IP addresses - DHCP is fine?
 
<br>

**<ins>Step 1: Layer 1 & Vlan mismatch issue</ins>**
    
- I started troubleshooting systematically at Layer 1
- Checked cabling on every connection, to make sure right cables were used 
- Noticed **incorrect cabling** between **SW-Finance**, **SW-OPs** and the **D1**.
- Replaced **crossover cable** with a straight through cable. Having incorrect cabling could cause inconssistent pings due to collisions from to pins transmitting at same time. Maybe the network team made a mistake yesterday?
- The next step was the interfaces, to check for possible **down interfaces**. I useed `show ip interface brief` on the two swtiches, as well as the layer 3 switch.
- I checked the **SW-Finance** first, on which there were no alarming things. Connected interfaces were up/up as expected 
- I then checked **SW-OPs** and i got a native VLAN mismatch message for the trunk between `GigabitEthernet0/1` on **SW-OP** and `FastEthernet0/1` on **D1**
  
<img width="797" height="120" alt="image" src="https://github.com/user-attachments/assets/47db9b83-8186-4407-b1ba-6286cfdc5590" />

- I logged into interface `Fast Ethernet 0/1` from global configuration mode and typed `switchport trunk native Vlan 30` to chnage **D1s** interface vlan to **SW-OPs** interface Vlan, Vlan 30



**<ins>Step 2: Layer 2 & L3</ins>**
- I then resumed my systematic troubleshooting by returning to **SW-OPS** to check the interfaces
- I typed `show ip interface brief` to verify and saw all required interfaces are up
- The next step was to check D1 for the SVIs, using `show ip interface brief` in global config mode.
- The SVIs were up witht he right ip addresses, both up/up.
- The next thing i did was check the finance IP addresses, since they were reporting weird IP addresses.
- There PCs had the signature Class C private ip addresses of 192.168......
- The previos VLAN mismatch could have easily caused this, as the IP information from D1 would not be able to reach the computers
- I reset the DHCP config on the devices by switching to static briefly, so the IP addresses could update.

<img width="350" height="194" alt="image" src="https://github.com/user-attachments/assets/0c2b5618-2467-4df3-9578-c4ea08dc8f45" />

<img width="437" height="205" alt="image" src="https://github.com/user-attachments/assets/b112de77-9605-4ea2-a0ed-fbcd8491b488" />


After solving the IP issue, the last issue to check was reachability, so I decided to ping from one vlan to another, to test inter vlan connectivity.


<img width="598" height="457" alt="image" src="https://github.com/user-attachments/assets/671a3580-beed-48a2-bef4-eac87ecf0f74" />

Yes the first one failed, due to ARP most likely, but the pings worked. 

**<ins>Ticket Key Points Review:</ins>**
    
- ~~Finance gets IP addresses, but IPs are strange, don't belong to expected subnet~~ Ip issue has been resolved, dhcp pooling works due to corrected VLAN mismatch, finance ips have the correct IPs.
- ~~Finance pings to OPs work sometimes, but inconsistent~~ inter vlan connectivity works, fully functional pings are displayed in the screenshots.
  

