# IT & Security Journey
U.S. Navy veteran transitioning into Cybersecurity. This repository will document my hands-on work which consists of homelab builds, detection engineering, offensive security, and networking/cloud skills that support it. Security is the primary focus, with Networking and Cloud being supporting foundations. 

Background: CompTIA Security+, Network+| B.S. IT Management (WGU)| Navy vet (USS WASP LHD-1)

## What's in here:

### Security: 
Homelab built on UTM/QEMU (Mac) with Windows Server 2019 AD DC, OPNSense firewall, Ubuntu SOC-Lab, and Kali Linux

- /security/homelab-ad-detection: ad SETUP (OUs, GPOs, password policy), OPNSense firewall rules, Wazuh SIEM deployment and log forwarding, CVE discovery/remediation

- /security/python-security-scripts: log parsing, IOC lookups, and triage automation scripts

### Networking:

- /networking/ccna-labs: Packet Tracer topologies (OSPF, VLANs, router-on-a-stick), subnetting practice

### Cloud:

- /cloud/azure: AZ-900 study notes, Azure fundamentals labs

Currently Working On:

Closing the full attack-detect-respond loop: simulating an attack from Kali against the homelab, confirming Wazuh detects it, and documenting the analyst response which involves recon, enumeration, and vulnerability testing phases are complete, detection/response piece in progress.

Note: This repo is a working log, and not a finished product. It updates as I build.
