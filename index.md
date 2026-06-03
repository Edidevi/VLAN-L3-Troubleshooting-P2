---
layout: troubleshooting
title: VLAN L3 Troubleshooting — P2
description: "Enterprise inter-VLAN support scenarios. Real-world incident tickets simulating network faults across a Layer 3 switched environment."
diagram: https://github.com/user-attachments/assets/01d6726b-60a9-4a23-941f-e508577faf98

concepts:
  - Inter-VLAN Routing
  - Subnet Misconfiguration
  - DHCP Troubleshooting
  - Layer 1 to Layer 3 Methodology
  - SVI Verification

environment:
  - "1x Cisco 3560 Layer 3 Switch (D1)"
  - "1x Cisco Catalyst 2960 (SW-HR)"
  - "End Devices — HR & IT PCs"
  - "Cisco Packet Tracer"

incidents:
  - id: "Incident Ticket #INC-00851"
    priority: "P2 — High"
    reported_by: "Marcus L., Sales"
    assigned_to: "Network Support"
    time: "02:48"

    symptoms:
      - "HR can reach IT, but IT can't reach HR"
      - "Both departments are getting IP addresses — DHCP is fine?"

    steps:
      - layer: "Layer 1"
        title: "Physical Layer & Cabling Check"
        findings:
          - "Started troubleshooting systematically at Layer 1."
          - "Checked cabling on every connection to make sure the right cables were used."
          - "Checked for possible <strong>down interfaces</strong> using <code>show ip interface brief</code> on both switches and the Layer 3 switch."
          - "End device and D1 connected interfaces were up and connected. The <strong>SVIs</strong> for D1 were also up."
        images:
          - src: https://github.com/user-attachments/assets/47db9b83-8186-4407-b1ba-6286cfdc5590
            caption: "show ip interface brief — interfaces confirmed up/up on D1 and access switches"

      - layer: "Layer 2"
        title: "VLAN & Trunk Verification"
        findings:
          - "Typed <code>show vlan</code> on <strong>SW-HR</strong> — saw 3 interfaces assigned to VLAN 5 (HR VLAN). The trunk interface Gig 0/1 was not assigned to any VLANs."
          - "Typed <code>show vlan</code> on D1 — D1 knew both VLANs, so it was not an issue here."
          - "The incident ticket states that when HR tries to ping, they get nothing. Suspected the wrong IP address."
          - "Checked the DHCP pool to see if pool addresses matched end device IPs — the HR VLAN was not leased, but the IT VLAN was."
        images:
          - src: https://github.com/user-attachments/assets/94a4cfed-e88f-41c0-b342-8e39e774bc5f
            caption: "DHCP pool — HR VLAN not leased, IT VLAN leased successfully"

      - layer: "Layer 3"
        title: "SVI & Subnet Verification"
        findings:
          - "Attempted to eliminate hardware issues by resetting DHCP on the PCs — switched to static then back to DHCP. Addresses did not update."
          - "Suspected a default gateway issue — typed <code>show running-config</code> on D1, but the default gateway was set correctly."
          - "Researched all findings — the only matching cause was <strong>subnet misconfiguration</strong>."
          - "Typed <code>show running-config | section interface vlan</code> to check the SVI subnets — found the HR VLAN had a <strong>/25 subnet mask</strong> instead of a /24. This changes the network completely, causing HR PCs to receive incorrect addresses."
          - "Resolved by entering <code>interface vlan5</code> on D1 in global configuration mode and correcting the IP address and subnet."
          - "Reset DHCP on the PCs — addresses were now correct."
        images:
          - src: https://github.com/user-attachments/assets/29d4b2e9-7356-4190-9150-0c1b49b8e35c
            caption: "DHCP reset on HR PC — switching to static then back to DHCP"
          - src: https://github.com/user-attachments/assets/41fa6aa3-fab3-46ae-a3dc-4d056f04d1c8
            caption: "SVI subnet check — HR VLAN showing incorrect /25 mask"

      - layer: "Verify"
        title: "Inter-VLAN Connectivity Test"
        findings:
          - "After committing the corrected subnet config, HR PCs received the correct IP addresses."
          - "Performed ping test to confirm inter-VLAN routing was fully restored between HR and IT."
        images:
          - src: https://github.com/user-attachments/assets/f33922fa-0395-436a-95a2-4425d4748ac1
            caption: "HR PC receiving correct DHCP address after subnet fix"
          - src: https://github.com/user-attachments/assets/15ccc39a-10d7-4d41-a7ae-cbd026577ba8
            caption: "Successful inter-VLAN ping results — HR to IT"

    resolution:
      - "HR PCs now receive correct IP addresses — the /25 subnet misconfiguration on the HR SVI was corrected to /24."
      - "IT to HR pings are now fully functional. HR could not be reached as the incorrect subnet meant HR PCs never had the expected addresses."

learnings:
  - "Always start troubleshooting at Layer 1 — rule out physical issues before moving up the stack."
  - "DHCP can appear to be working if devices receive any address — always verify addresses belong to the expected subnet."
  - "A subnet mask misconfiguration on an SVI changes the network entirely, silently breaking connectivity."
  - "DHCP leases may need to be manually refreshed after correcting an SVI subnet."
---
