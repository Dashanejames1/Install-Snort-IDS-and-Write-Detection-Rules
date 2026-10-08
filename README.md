# Install-Snort-IDS-and-Write-Detection-Rules


**Author:** Dashane James  
**Lab Environment:** VMware Workstation | Kali Linux | Metasploitable 2

**Purpose:** [Install Snort 3, write a custom detection rule for ICMP traffic, and confirm it alerts on live traffic.]
**Status:** 🔵 Completed

---

## 📋 Overview

[This repository documents installing Snort 3 on Kali Linux and using it as an intrusion detection system in an isolated VMware lab. I first ran Snort as a packet sniffer to confirm it could see traffic on the lab interface, then wrote a custom rule to alert on ICMP echo requests sent from Metasploitable 2 to Kali. Pinging Kali from the target generated five alerts for five packets, confirming the rule fired on live traffic. The lab closes with a comparison of signature-based and anomaly-based detection.]

---

## 🧪 Lab Environment

| Component | Details |
|---|---|
| Hypervisor | VMware Workstation (Host-Only Network) |
| IDS/monitoring Machine | Kali Linux 2026.1 — `192.168.79.129` |
| Traffic Source | [Metasploitable 2] — `192.168.79.130` |
| Network Type | Host-Only (isolated, no internet exposure) |
| Host OS | Windows 11 — ASUS Vivobook 14 |

> ⚠️ **Note:** All activity was performed in a controlled, isolated lab environment against deliberately vulnerable machines. No unauthorized access to live networks was performed.

---

## 🛠️ Tools Used

- **Snort 3 (3.12.2.0)** — Open-source intrusion detection system, used first as a packet sniffer and then with a custom rule file to generate alerts
- **ping** — Generated the ICMP traffic used to test visibility (Kali → Metasploitable) and to trigger the rule (Metasploitable → Kali)
- **nano** — Used to create the custom rule file `~/local.rules`

---


## 🔬 Tasks / Assessments Performed

### 1. [Install Snort and run in packet sniffer mode first.]

Before writing any rules, I installed Snort and ran it as a plain packet sniffer to confirm it could see traffic on the lab interface. I pinged Metasploitable from Kali while Snort listened on `eth0`.

# Commands used
[sudo apt install snort - y]
[sudo snort -i eth0 -A console -q]


# Output

Snort 3.12.2.0 captured 273 packets on eth0 over 2 minutes 26 seconds with zero drops. 255 packets (93.4%) were ICMP, matching the ping test from Kali to 192.168.79.130. The remaining traffic was ARP (16) and UDP (2) background noise. This confirms Snort is seeing traffic on the lab interface and is ready for detection rules.

<img width="323" height="255" alt="Screenshot 2026-09-29 223543" src="https://github.com/user-attachments/assets/16bc749e-2bfd-4e87-b508-7a3769f381a4" />

*Installing snort 3.12.2.0 on Kali via apt.*

<img width="325" height="52" alt="Screenshot 2026-09-29 225951" src="https://github.com/user-attachments/assets/77bdd274-cc89-425e-a09e-c0bb780c92c4" />

*First attempt to run in packet sniffer mode failed. (-A console is Snort 2 Syntax; Snort 3 uses -A alert_fast instead,.)*


<img width="652" height="254" alt="Screenshot 2026-09-29 231007" src="https://github.com/user-attachments/assets/9092b1ed-09aa-4144-b759-5864d23460f8" />

*Snort listening on eth0 (left) while pinging the metasploitable 2 target at 192.168.79.130. (right)*


<img width="324" height="256" alt="Screenshot 2026-09-29 231040" src="https://github.com/user-attachments/assets/3ec303bb-7eaa-45e6-ad26-c30d5c7dcad5" />

Packet statistics after stopping Snort. 273 packets captured with 0 drops; 255(93.4%) were ICMP  from the Ping test. (1/2)

<img width="323" height="177" alt="Screenshot 2026-09-29 231110" src="https://github.com/user-attachments/assets/3181f229-183b-4579-b966-453dc1124508" />

Packet statistics after stopping Snort. 273 packets captured with 0 drops; 255(93.4%) were ICMP  from the Ping test. (2/2)




### 2. [Write a rule to detect ICMP from metasploitable to kali.]
[With visibility confirmed, I wrote a custom rule that alerts whenever Metasploitable pings Kali. The rule lives in its own file so it stays separate from Snort's default configuration.]

# Commands used

```bash
# Create the rule file
nano ~/local.rules
```

**Rule**

```
alert icmp 192.168.79.130 any -> 192.168.79.129 any (msg:"ICMP from Metasploitable to Kali"; itype:8; sid:1000001; rev:1;)
```

| Part | Meaning |
|---|---|
| `alert icmp` | Generate an alert for ICMP traffic |
| `192.168.79.130 any -> 192.168.79.129 any` | Only traffic from Metasploitable to Kali |
| `msg:"..."` | Text shown in the alert |
| `itype:8` | Match ICMP type 8 (echo request) only |
| `sid:1000001` | Unique rule ID; custom rules start at 1,000,000 |
| `rev:1` | Rule revision number |

# Output

<img width="321" height="32" alt="Screenshot 2026-10-05 124833" src="https://github.com/user-attachments/assets/49aa59b6-6824-4aab-8670-ce7f512852fb" />

Running nano command to create a rule.

<img width="323" height="260" alt="Screenshot 2026-10-05 124734" src="https://github.com/user-attachments/assets/4b09c249-bc3d-408c-bfd3-cbfc12360154" />

Rule created to detect ICMP from the target to the host (1/2)

<img width="323" height="256" alt="Screenshot 2026-10-05 124807" src="https://github.com/user-attachments/assets/f8d6c4b7-a19f-42e7-8e28-0fe8e31b071f" />

Rule created to detect ICMP from the target to the host (2/2)
* itype:8 matches only ping requests, and custom rule IDs (sid) start at 1,000,000 *
* 
**Findings:**
  
The rule is deliberately narrow. It matches one direction, one source, one destination, and one ICMP type, so it should fire once per ping request and ignore the replies.



### 3. [Test by pinging from Metasploitable and watching Snort alert.]

To test the rule, I started Snort with the rule file loaded and sent five pings from Metasploitable to Kali.


# Command used

sudo snort -i eth0 -v -c /etc/snort/snort.conf

sudo snort -c /etc/snort/snort.lua -R ~/local.rules -i eth0 -A alert_fast

ping -c 5 192.168.179.129

```bash
# First attempt (failed): snort.conf is the Snort 2 config format
sudo snort -i eth0 -v -c /etc/snort/snort.conf

# Working command (on Kali): Snort 3 config, custom rules, fast alerts
sudo snort -c /etc/snort/snort.lua -R ~/local.rules -i eth0 -A alert_fast

# On Metasploitable: send 5 pings to Kali
ping -c 5 192.168.79.129
```


# Output

Metasploitable sent 5 packets and received 5 replies with 0% packet loss. Snort generated 5 "ICMP from Metasploitable to Kali" alerts, each showing source 192.168.79.130, destination 192.168.79.129, a timestamp, and rule ID `1:1000001:1`.

<img width="362" height="119" alt="Screenshot 2026-10-06 121341" src="https://github.com/user-attachments/assets/1c3c3b33-4a80-4dce-a5a9-b81cd342e50e" />

Pinging Kali (192.168.79.129) from Metasploitable to trigger the Snort ICMP detection rule.

<img width="416" height="390" alt="Screenshot 2026-10-06 122712" src="https://github.com/user-attachments/assets/e82745b8-c7be-4cfa-83df-a1f83c174460" />

Alerts firing + Packet Statistics

<img width="224" height="398" alt="Screenshot 2026-10-06 122742" src="https://github.com/user-attachments/assets/4a646988-7d1e-49d1-9815-00458db05bb1" />

Module Statistics

<img width="419" height="395" alt="Screenshot 2026-10-06 122829" src="https://github.com/user-attachments/assets/120b91e0-9c9b-47f1-a037-d075f911c4bb" />

Stream and AppID statistics: 12 ICMP sessions tracked by `stream_icmp` and 267 total flows monitored.

2
```

**Findings:**

Five pings produced exactly five alerts, so the rule works as written. Snort saw 385 ICMP packets during the session but alerted on only the 5 that matched the rule's source, destination, and type, which shows the rule is filtering rather than alerting on all ICMP.



### 4. Write the difference between signature-based vs anomaly-based detection.


### Output

Signature-based detection (used by Snort) works by matching network traffic against a database of known attack patterns — similar to how antivirus works. It's fast, reliable, and generates few false positives, but it can only detect threats it has a signature for it. Zero-day attacks and new malware will pass through undetected since no signature exists yet.

Anomaly-based detection works differently — it learns what 'normal' network behavior looks like and alerts on anything that deviates from that baseline, even if no known signature exists. This makes it effective against zero-days and novel attacks, but it generates significantly more false positives since any unusual but legitimate activity can trigger an alert.

The key difference is signature-based = low false positives on known threats, misses unknowns. Anomaly-based = catches unknowns, more noise. In practice, most enterprise SOC environments use both together for maximum coverage.

.


** Final Findings:**


Module Statistics — shows which Snort detection modules were active and how many searches they performed. Proves the detection engine was actually working, not just running idle.

Summary Statistics — shows the overall session totals including how many alerts were generated. This is the most important closing number — it will show exactly how many ICMP alerts your rule fired during the test.



Port / Service / Vulnerability / NSE Script / Severity / Notes

21/tcp (FTP) - ftp-vsftpd-backdoor (CVE-2011-2523)	🔴 Critical	Live exploitation confirmed. The vulnerable vsFTPd 2.3.4 backdoor allowed remote command execution and root shell access. (CVSS 9.8)

22/tcp (SSH) - Weak SSH cryptographic configuration - 🟡 Medium	Weak or outdated cryptographic algorithms were identified, potentially reducing the security of encrypted communications.

23/tcp (Telnet) - Telnet service detected	🟠 High	Telnet transmits credentials in plaintext, making usernames and passwords susceptible to interception.

80/tcp (HTTP) - Slowloris Denial-of-Service	🟡 Medium	The web server appears susceptible to the Slowloris DoS attack, which can exhaust server connections and deny service to legitimate users.

80/tcp (HTTP) - HTTP TRACE method enabled	🟢 Low	TRACE is enabled, which may aid reconnaissance or cross-site tracing attacks but does not directly compromise the server.

80/tcp (HTTP) - SQL Injection vulnerability	🟠 High	SQL injection was detected and could allow attackers to retrieve, modify, or delete database information.

111/tcp (rpcbind) - RPC information disclosure	🟢 Low	rpcbind exposes information about available RPC services, assisting attacker reconnaissance.

139/tcp (NetBIOS/SMB) - SMB enumeration	🟡 Medium	SMB information disclosure allows attackers to gather host and share information useful for later attacks.

445/tcp (SMB) - smb-vuln-ms10-061 🟢 Not Vulnerable	NSE script returned false, indicating the system is not vulnerable to MS10-061.

445/tcp (SMB) - smb-vuln-ms10-054 🟢 Not Vulnerable	NSE script returned false, indicating the target is not affected by MS10-054.

445/tcp (SMB) - smb-vuln-regsvc-dos	⚪ Inconclusive	The script failed to complete successfully, so the vulnerability status could not be determined.

3306/tcp (MySQL) - MySQL information disclosure	🟡 Medium	Database version and service information were exposed, which may help attackers identify known exploits.

443/tcp (HTTPS) - Logjam (Weak Diffie-Hellman Parameters) 🟡 Medium	Weak Diffie-Hellman parameters reduce TLS security and could allow encrypted communications to be weakened under certain attack scenarios

**Findings:** 

The table above consolidates all of the findings reported during the Nmap --script vuln scan and categorizes each result by severity. The most significant finding was the vsFTPd 2.3.4 backdoor (CVE-2011-2523) on port 21, which was successfully exploited to obtain root-level access, making it the only Critical vulnerability confirmed through live exploitation. Several additional weaknesses were identified, including SQL injection, Telnet running without encryption, Slowloris denial-of-service susceptibility, and weak SSH cryptographic settings, all of which increase the attack surface of the target system. The SMB vulnerability checks also demonstrated that vulnerability scans can produce different outcomes: some scripts confirmed the system was not vulnerable, while others returned inconclusive results because the checks could not be completed. Overall, the scan illustrates the importance of reviewing every NSE script result individually, as findings may represent confirmed vulnerabilities, informational issues, successful mitigations, or inconclusive tests requiring additional investigation.

---
## 📊 Results Summary

| Test | Packets Analyzed | Alerts | Result |
|---|---|---|---|
| Packet sniffer mode (Task 1) | 273 | n/a | 0 drops; 255 ICMP packets seen |
| Custom ICMP rule (Task 3) | 1,568 | 5 | 5 pings sent, 5 alerts generated and logged |

---

## 🗺️ MITRE ATT&CK Mapping

| Behavior Detected | ATT&CK Tactic | Technique ID | Technique Name |
|---|---|---|---|
| ICMP echo requests from one host to another | Discovery | T1018 | Remote System Discovery |

---

## 📚 CySA+ Exam Relevance

This lab maps to the following CompTIA CySA+ (CS0-003) exam domains:

| Domain | Coverage |
|---|---|
| Security Operations (33%) | Deploying an IDS, writing a detection rule, and analyzing alerts and network traffic statistics |
| Incident Response (20%) | Detection and analysis: confirming that suspicious traffic generates an alert that identifies source and destination |
| Reporting & Communication (17%) | Documenting the setup, rule logic, and test results in a structured, readable format |

---

## 🔑 Technical Notes

> **Snort 2 vs. Snort 3:** Most tutorials online are written for Snort 2. Kali installs Snort 3, where the syntax is different.

| Snort 2 | Snort 3 |
|---|---|
| `-A console` | `-A alert_fast` |
| `/etc/snort/snort.conf` | `/etc/snort/snort.lua` |

```bash
# Run Snort 3 with the default config plus a custom rule file
#   -c  main configuration file
#   -R  additional rules file
#   -i  interface to listen on
#   -A  alert output mode
sudo snort -c /etc/snort/snort.lua -R ~/local.rules -i eth0 -A alert_fast
```

- `itype:8` limits the rule to echo requests. Without it, the rule would match any ICMP type from the source to the destination.
- Custom rule SIDs start at 1,000,000 so they don't collide with official rule IDs.
- Stopping Snort with `Ctrl+C` prints the packet, module, and summary statistics shown in the screenshots.

---

## 📌 About This Project

This is the first defensive lab in my cybersecurity portfolio. The earlier repositories focused on scanning and capturing traffic from the attacker's side; this one moves to the analyst's side by detecting traffic and alerting on it, which is the work I'm aiming for as a cybersecurity analyst.

**Related repositories:**
- Nmap-Host-Discovery-and-Lab-Baseline — Established a baseline of the lab network using ping sweeps and host discovery
- Nmap-Scan-Types-SYN-vs-TCP-vs-UDP — Compared SYN, TCP connect, and UDP scan types for speed, stealth, and reliability
- Service-Version-Detection-and-OS-Fingerprinting — Identified exact software versions on Metasploitable and researched the CVEs tied to them
- Wireshark-Capture-and-Analyze-Traffic — Captured and analyzed live packet traffic, including plaintext credential exposure over Telnet
- TCPDump-CLI-Packet-Capture — Captured traffic from the command line and saved it to a `.pcap` file for analysis in Wireshark

---

## 👤 Author

**Dashane James**  
Senior Field Service Technician → Cybersecurity Analyst  
📍 Yonkers, NY  
🎓 B.S. Information Technology — SUNY Canton  
🏆 CompTIA Security+ | CySA+ (In Progress)  
🔗 [GitHub](https://github.com/Dashanejames1) | [Zero Trust Cyber Security Brand](https://www.instagram.com/zerotrust_cybersecurity)

---

*This repository is part of an active portfolio demonstrating hands-on cybersecurity skills. All lab work performed in isolated environments for educational purposes.*
