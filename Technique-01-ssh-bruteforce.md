# Technique #1: SSH Brute Force Attack
## MITRE ATT&CK Technique: T1110 - Brute Force
### Date: Aug.11, 2026
### Lab Environment: Self-hosted homelab

## Objective
Simulate a real-world SSH Brute-force attack against a lab target and evaluate whether the Wazuh SIEM correctly detects and escalates the activity to a high-severity alert.

## Summary

For this test, I used Kali Linux to attack my SOC-Lab-01 machine (Ubuntu Linux) with a tool called Hydra. Hydra tries a bunch of different passwords really fast until it finds the right one, or runs out of passwords to try.

I ran the attack against my own local account using a huge password list called rockyou.txt (Contains about 14 million passwords in it), which I let run for a few minutes.

First, I checked the target machine’s log file (auth.log) and could see all the failed login attempts being recorded, which is what I expected.

Then I checked my Wazuh SIEM to see if it noticed the attack. At first, it caught the individual failed logins, but it didn’t flag it as an actual “brute force attack.” The reason being: Wazuh needs to see 8 failed logins within 120 seconds from the same IP address before it calls it a brute-force attack. My first test wasn’t quite fast enough to hit that number.

So I ran the attack again for longer, and this time Wazuh caught it right away and flagged it as a high-severity alert, correctly labeled as a brute-force attack (MITRE ATT&CK technique T1110).

What I learned: Wazuh doesn’t flag every failed login as an attack — it needs to see a certain amount of failed attempts in a short time window to consider it a “brute force” pattern. This is good to know because if a real attacker tried logging in slowly (spacing out their attempts), they might be able to sneak past this rule without triggering an alert. That’s something worth testing and thinking about for anyone setting up detection rules.

### **Attack**
**Command:** hydra -l charles -P /usr/share/wordlists/rockyou.txt ssh://192.168.64.2 -t 4

- Username: single known local account (charles)
-	Password list: rockyou.txt (~14 million entries)
-	Threads: 4 concurrent connections (-t 4)
-	Duration: ~7 minutes, generating several hundred login attempts
