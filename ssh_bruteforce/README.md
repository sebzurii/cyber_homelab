Objective

Simulate an SSH brute-force attack against the victim machine and identify it through system log analysis.

Lab setup

- Attacker: Kali Linux, IP `192.168.64.2`
- Victim: Ubuntu Server, IP `192.168.64.3`, running OpenSSH
- Both VMs isolated on a private virtual network with no exposure to the host's physical LAN

Steps taken:

1. Confirmed SSH was reachable from the attacker machine:
   ```
   nmap -p 22 192.168.64.3
   ```

2. Ran a brute-force attack against SSH using Hydra with a common password wordlist:
   ```
   hydra -l sebastianzurita -P /usr/share/wordlists/rockyou.txt.gz ssh://192.168.64.3 -t 4
   ```
<img width="2416" height="1574" alt="kali_attacker" src="https://github.com/user-attachments/assets/f2bb55ee-ff43-40c1-b283-e3e2434f6418" />

3. On the victim machine, checked the authentication log for evidence of the attack:
   ```
   sudo grep "Failed password" /var/log/auth.log | tail -50
   ```
<img width="2438" height="1596" alt="ubuntu_victim" src="https://github.com/user-attachments/assets/8afda573-73f2-4c92-a8a3-102b714e46d3" />

Troubleshooting encountered

The first attempt at this attack failed with `[ERROR] all children were disabled due to too many connection errors`, even though a manual SSH connection to the same host worked fine.

Diagnosis: watching `/var/log/auth.log` live while Hydra ran (`sudo tail -f /var/log/auth.log`) showed the real cause — OpenSSH's default `MaxAuthTries` setting (6) was force-disconnecting Hydra's connections after 6 failed attempts per session. Hydra interpreted these disconnects as generic connection errors rather than retrying, and gave up.

Fix: edited `/etc/ssh/sshd_config` on the victim machine and raised the limit:
```
MaxAuthTries 64
```
then restarted the service:
```
sudo systemctl restart ssh
```

After this change, Hydra ran normally and began generating a steady stream of failed-login attempts, confirmed both in Hydra's own status output and in the victim's auth log.

Why this mattered: this was a useful reminder that a "broken attack" and a "working defence" can look identical from the attacker's side, the server was correctly doing its job by cutting off repeated failures, and the fix required understanding why the block was happening rather than just retrying blindly.

Evidence:

Sample log output from `/var/log/auth.log` during the attack:
```
Failed password for sebastianzurita from 192.168.64.2 port 53052 ssh2
Failed password for sebastianzurita from 192.168.64.2 port 53052 ssh2
Failed password for sebastianzurita from 192.168.64.2 port 53052 ssh2
Failed password for sebastianzurita from 192.168.64.2 port 53052 ssh2
```

Hydra status output confirming an active, sustained attack:
```
[STATUS] 79.00 tries/min, 79 tries in 00:01h, 4 active
[STATUS] 60.33 tries/min, 181 tries in 00:03h, 1 active
[STATUS] 37.43 tries/min, 262 tries in 00:07h, 1 active
[STATUS] 27.73 tries/min, 416 tries in 00:15h, 1 active
```

Notable pattern: a high volume of failed login attempts from a single source IP (`192.168.64.2`) in a short window, the core signature of a brute-force attempt.

Detection logic

A simple detection rule for this behaviour:

Alert if: more than 5 failed SSH login attempts are observed from the same source IP within a 60-second window.

This could be implemented as:
- A `fail2ban` jail rule (practical, immediate blocking)
- A SIEM correlation rule (e.g., in Wazuh or Elastic) for environments monitoring multiple hosts

Remediation

- Fail2ban — automatically ban IPs after n failed attempts within a time window
- Disable password authentication — require SSH key-based auth only
- Rate limiting — restrict connection attempts at the firewall level
- Keep `MaxAuthTries` at its secure default — in this lab it was deliberately raised to let the attack simulation run; in a real environment, the default (or lower) value is itself a defensive control and should not be increased.


Key takeaway:

This exercise demonstrated the full detection lifecycle: an attack occurring, evidence being generated in logs, a human (or rule) recognising the pattern, and a concrete remediation step being identified. It also reinforced an important lesson, a built-in security control (`MaxAuthTries`) was the actual reason the first attack attempt "failed," which is itself worth documenting, since misreading a defence as a bug is a realistic scenario in real troubleshooting work.
