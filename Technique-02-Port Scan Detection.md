# Technique #2: Port Scan Detection
## MITRE ATT&CK Technique: T1046 - Network Service Discovery
### Lab Environment: Self-hosted homelab
### Date: Aug.12,2026



### Objective
Simulate a port scan against a lab target and build a working Wazuh detection for it. Unlike SSH Brute-force, Wazuh has no built-in rule for this, so this technique involved writing a custom rule and decoder from scratch.

### Summary
For this one, I used Nmap on Kali to scan my SOC-Lab-01 machine and check all 65,535 ports to see which ones were open. My target had UFW (Ubuntu’s firewall) turned on, only allowing SSH and web traffic through, with logging turned on so it would record anything it blocked.

I checked the target’s firewall log first, and could see it recording tons of blocked connection attempts, one for basically every port Nmap touched. So the target was logging everything correctly.

Then I checked Wazuh to see if it caught the scan. This is where things got complicated. At first, nothing was showing up as an alert at all. After a lot of digging, I found out my Wazuh manager service had actually crashed hours earlier without me realizing it, so nothing new could get through until I restarted it. Lesson learned: always double check your SIEM itself is actually running before troubleshooting anything downstream of it.

Once I had that fixed, Wazuh was receiving the firewall logs, but still wasn’t flagging them as anything of importance. Turns out, unlike SSH brute force, Wazuh doesn’t come with a built-in rule for detecting port scans. I had to build the detection myself from scratch.

I wrote two custom rules:

- One rule that recognizes a single blocked connection from the firewall log
- A second rule that watches for that first rule firing 8 or more times within 120 seconds, since that pattern (lots of blocked connections in a short window) is what an actual port scan looks like

Getting this working took a few tries. My first attempt tried to also check that the traffic was coming from the same IP and hitting different ports each time, but that requires Wazuh to already have those pulled out as separate labeled fields, which it wasn’t doing. I tried building a custom decoder to fix that, but ran into repeated syntax errors and conflicts with Wazuh’s built-in decoders. In the end, I simplified the rule to just count how often blocked connections happened in a short window, without requiring the source IP to be separately verified, since I was working with one attacking machine anyway. 

After that fix, I re-ran the scan and Wazuh finally flagged it correctly as a possible port scan.

What I learned: Wazuh doesn’t automatically know how to detect every kind of attack out of the box. Some detections, like this one, have to be built completely from scratch, including deciding what “suspicious” actually looks like in the logs. I also learned to always check that the SIEM itself is healthy before assuming a detection problem is the issue.
