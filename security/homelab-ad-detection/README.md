# Homelab: Active Directory, Firewall Hardening & SIEM Detection

## Overview
Self-directed enterprise lab built on UTM/QEMU (MacBook Air, Apple Silicon), simulating a small business network: domain controller, firewall, and a SIEM collecting logs from both.

**Environment:**
- Windows Server 2019 — Active Directory Domain Controller (`homelab.local`)
- OPNsense — perimeter firewall
- Ubuntu Server 26.04 LTS — Wazuh SIEM (192.168.64.7)
- Kali Linux — attacker/recon VM

## Active Directory
- Promoted Windows Server 2019 to a domain controller for `homelab.local`
- Built an "Employees" OU and created test user accounts
- Authored a domain-wide password policy GPO
- **Key lesson:** password policy GPOs only take effect when linked at the domain root — an OU-level link silently fails to apply. Diagnosed and corrected this by re-linking at `homelab.local`.

## OPNsense Firewall
- Verified and tested a Block Telnet (port 23) rule — confirmed via a live telnet attempt from the Ubuntu VM and cross-checked in firewall logs, learning the practical difference between "filtered" and "closed" port states
- Restricted the admin web GUI to a single trusted host (source-IP rule allowing only the Ubuntu VM on port 443); validated the block using `curl` from a separate VM outside the allow-list
- Configured DHCP and NAT port-forwarding rules to simulate real-world network services

## Vulnerability Discovery & Remediation
- Ran reconnaissance from Kali: `nmap -sV` and `nmap --script vuln -sV` against the firewall
- Discovered a **CVSS 10.0 DNS cache-poisoning vulnerability** in Unbound 1.24.2 (bundled with OPNsense)
- Independently verified the finding against the vendor's security advisory before acting on it
- Applied the firmware update (77 packages, OPNsense → 26.1.11, Unbound → 1.25.1)
- Confirmed remediation with a follow-up scan
- Documented the full recon → verify → remediate → confirm workflow as a written incident report

## Wazuh SIEM Deployment
- Installed Wazuh on a dedicated Ubuntu Server VM
- Troubleshot two infrastructure issues during setup:
  - RAM shortfall from ARM64 virtualization overhead — resolved by increasing allocation to 5120MB
  - Disk space failure from Ubuntu's guided LVM partitioning under-allocating space — resolved via `lvextend` and `resize2fs`
- Configured OPNsense to forward firewall logs to Wazuh (syslog, UDP 514)
- Diagnosed a log-forwarding failure caused by OPNsense's syslog-ng using a different process name (`filterlog`) than the default config expected (`firewall`) — corrected the destination filter directly in OPNsense's syslog-ng config

## Skills demonstrated
Active Directory administration · GPO troubleshooting · Firewall rule design & validation · Vulnerability scanning (Nmap/NSE) · CVE research & remediation · SIEM deployment · Log forwarding & syslog troubleshooting · Linux system administration

## Status
Foundation is complete and stable. Next phase: closing the full attack-detect-respond loop by simulating an attack from Kali and confirming Wazuh generates a corresponding alert (in progress).
