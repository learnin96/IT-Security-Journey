# Technique #3: Privilege Escalation Detection (New Admin Account)
## MITRE ATT&CK Technique T1136 - Create Account
### Lab Environment: Self-hosted homelab
### Date: Aug.13,2026


## Objective
Simulate a common attacker move: after gaining access to a machine, create a new hidden admin account to keep access later without needing to re-exploit anything. Also serves as a test to see if Wazuh detects this.

## Summary
For this test, I created a new user account on my SOC-Lab-01 machine called “backdoor_user,” gave it a password, and then added it to the sudo group, which gives it full admin rights. This is a classic real-world persistence technique: attackers do this after breaking in so they have a standing way back into the system.
I ran three commands to do this:

- Created the account
- Set a password for it
- Added it to the sudo group

First, I checked the target’s auth log directly and could see all three actions recorded clearly: the account being created, the password being set, and the account being added to the sudo group.

Then I checked Wazuh, and ran into the exact same issue as last time: nothing was showing up as an alert. I checked the Wazuh manager service, and sure enough, it had crashed again without me noticing. I restarted it and confirmed it was healthy.
Once the manager was back up, I checked whether Wazuh already had a built-in detection for this. I found that Wazuh does have a decoder built in for reading “new user” log lines from useradd, but it did not have a decoder for usermod (the command that adds someone to a group). My first move was to write a custom rule assuming I’d need to build detection from scratch again, referencing a rule ID I hadn’t actually confirmed existed yet.
Before testing it, I went back and checked Wazuh’s default ruleset directly, and found that a rule for this already existed: rule 5902, which matches on new user creation and is already correctly mapped to MITRE T1136. My custom rule turned out to be redundant, so I removed it and confirmed the built-in rule 5902 fired correctly on its own.

Result:

Rule: 5902 (level 8) -> “New user added to the system.”
This confirms Wazuh’s default ruleset does catch new account creation on its own, no custom rule needed.

Key finding:

Unlike port scanning (Technique #2), Wazuh already had a working, correctly MITRE-mapped detection for this technique out of the box. The real issue wasn’t a missing detection - it was that my Wazuh manager service had silently crashed, which would have completely blinded this alert in a real environment without me knowing. This is now the second time this exact issue has come up, which tells me service health monitoring for the SIEM itself is something a real SOC setup would need to actively watch for, not just assume is always running.
I also learned to verify a rule ID actually exists in the ruleset before building on top of it, rather than assuming from memory - I nearly wrote a redundant custom rule referencing a real rule ID before confirming it firsthand.

Next steps:
Test whether this detection also catches a new account being added to other privileged groups (not just sudo), and whether Wazuh distinguishes between an admin manually creating an account versus a suspicious pattern (e.g., account creation immediately followed by login from an unusual location)
