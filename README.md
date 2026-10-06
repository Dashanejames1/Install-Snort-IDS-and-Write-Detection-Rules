# Install-Snort-IDS-and-Write-Detection-Rules


**Author:** Dashane James  
**Lab Environment:** VMware Workstation | Kali Linux | Metasploitable 2

**Purpose:** [The goal of this repository is to install Snort and write a custom rule to detect Nmap scans.]
**Status:** 🔵 Completed

---

## 📋 Overview

[This repository documents ]

---

## 🧪 Lab Environment

| Component | Details |
|---|---|
| Hypervisor | VMware Workstation (Host-Only Network) |
| Attacker Machine | Kali Linux 2026.1 — `192.168.79.129` |
| Target Machine | [Metasploitable 2] — `192.168.79.130` |
| Network Type | Host-Only (isolated, no internet exposure) |
| Host OS | Windows 11 — ASUS Vivobook 14 |

> ⚠️ **Note:** All activity was performed in a controlled, isolated lab environment against deliberately vulnerable machines. No unauthorized access to live networks was performed.

---

## 🛠️ Tools Used

- **Nmap** — Used for both standard scanning and running NSE (Nmap Scripting Engine) scripts to detect and, in one case, actively exploit vulnerabilities
- **NSE (Nmap Scripting Engine)** — Ran default (`-sC`), targeted (`ftp-anon`, `smb-vuln*`), and broad (`vuln`) script categories against the target
- **NVD (National Vulnerability Database)** — Referenced to research CVE details, CVSS scores, and attack vectors for confirmed vulnerabilities

---

## 📁 Repository Structure

```
[Repo-Name]/
├── README.md
├── [folder-1]/
│   ├── [file1.txt]        # Description of file
│   └── [file2.md]         # Description of file
├── [folder-2]/
│   ├── [file3.md]         # Description of file
│   └── [file4.txt]        # Description of file
└── reports/
    └── [final-report.md]  # Full assessment report
```

---

## 🔬 Tasks / Assessments Performed

### 1. [Install Snort and run in packet sniffer mode first.]
For this task ...

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
[]

# Commands used
nano ~/local.rules
alert icmp 192.168.79.130 any -> 192.168.79.129 any (msg:"ICMP from Metasploitable to Kali"; itype:8; sid:1000001; rev:1;)


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




### 3. [Test by pinging from Metasploitable and watching Snort alert.]
[] 


# Command used

sudo snort -i eth0 -v -c /etc/snort/snort.conf

sudo snort -c /etc/snort/snort.lua -R ~/local.rules -i eth0 -A alert_fast

ping -c 5 192.168.179.129


# Output

First, Pinging Kali (192.168.79.129) from Metasploitable to trigger the Snort ICMP detection rule — 5 packets transmitted, 5 received, 0% packet loss. These 5 ping packets are exactly what generated the 5 'ICMP from Metasploitable to Kali' alerts visible in the Snort console, confirming the detection rule fired on real traffic between the two lab machines

Snort custom ICMP rule firing in real time — 5 alerts generated showing 'ICMP from Metasploitable to Kali' with source IP 192.168.79.130 → destination 192.168.79.129, timestamps, and rule ID 1:1000001:1. Packet statistics confirm 1,568 total packets analyzed with 385 ICMP packets captured (24.5% of total traffic).

Snort module statistics showing the detection engine processed 1,568 packets, generated exactly 5 alerts (matching the 5 ICMP alerts seen in the console), and logged all 5. The ips_actions section confirms 5 alert actions were taken — one for each ping detected from Metasploitable. The port_scan module also tracked 1,485 packets across 28 trackers, showing Snort was actively analyzing traffic beyond just ICMP.

Snort session tracking statistics — 12 ICMP sessions tracked via stream_icmp, 267 total network flows monitored, and application identification running across all traffic. This confirms Snort was performing deep packet inspection and session tracking throughout the test, not just simple packet counting.

<img width="362" height="119" alt="Screenshot 2026-10-06 121341" src="https://github.com/user-attachments/assets/1c3c3b33-4a80-4dce-a5a9-b81cd342e50e" />

Pinging Kali (192.168.79.129) from Metasploitable to trigger the Snort ICMP detection rule.

<img width="416" height="390" alt="Screenshot 2026-10-06 122712" src="https://github.com/user-attachments/assets/e82745b8-c7be-4cfa-83df-a1f83c174460" />

Alerts firing + Packet Statistics

<img width="224" height="398" alt="Screenshot 2026-10-06 122742" src="https://github.com/user-attachments/assets/4a646988-7d1e-49d1-9815-00458db05bb1" />

Module Statistics

<img width="419" height="395" alt="Screenshot 2026-10-06 122829" src="https://github.com/user-attachments/assets/120b91e0-9c9b-47f1-a037-d075f911c4bb" />

Stream and Appid Statistics

2
```

**Findings:**





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

## 📊 Key Findings Summary

| Port/Service | Tool Used | Risk Level | Notes |

| 21/tcp FTP | Nmap (`vuln`, `ftp-anon`) | 🔴 Critical | vsFTPd 2.3.4 backdoor (CVE-2011-2523) — live exploitation confirmed, root access achieved |
| 23/tcp Telnet | Nmap (`vuln`) | 🟠 High | Transmits credentials in plaintext, vulnerable to interception |
| 80/tcp HTTP | Nmap (`vuln`) | 🟠 High | SQL injection vulnerability detected |
| 80/tcp HTTP | Nmap (`vuln`) | 🟡 Medium | Susceptible to Slowloris denial-of-service attack |
| 445/tcp SMB | Nmap (`-sC`, `smb-vuln*`) | 🟡 Medium | Message signing disabled; guest authentication allowed |
| 22/tcp SSH | Nmap (`vuln`) | 🟡 Medium | Weak/outdated cryptographic configuration identified |
| 3306/tcp MySQL | Nmap (`vuln`) | 🟡 Medium | Database version and service info exposed |

**Risk Levels:** 🔴 Critical | 🟠 High | 🟡 Medium | 🟢 Low


---

## 🗺️ MITRE ATT&CK Mapping

| Action Performed | ATT&CK Tactic | Technique ID | Technique Name |
|---|---|---|---|
| Default and targeted script scanning (-sC, ftp-anon, smb-vuln*) | Reconnaissance | T1595 | Active Scanning |
| Anonymous FTP login confirmed | Initial Access | T1078 | Valid Accounts |
| vsFTPd backdoor exploitation (root shell via `id` command) | Execution | T1059 | Command and Scripting Interpreter |
| Root-level access confirmed post-exploitation | Privilege Escalation | T1068 | Exploitation for Privilege Escalation |

---

## 🛡️ Defensive Recommendations

Based on findings, the following remediations would be recommended in a real environment:

1. **vsFTPd 2.3.4 backdoor (Critical)** — Immediately upgrade or replace the FTP service; this version is known to contain an intentional backdoor and should never be used in production.
2. **Anonymous FTP login allowed** — Disable anonymous access entirely; require authenticated accounts for any FTP access.
3. **Telnet enabled (plaintext credentials)** — Disable Telnet and replace with SSH for all remote administration.
4. **SMB message signing disabled** — Enable SMB message signing to prevent tampering and man-in-the-middle attacks on SMB traffic.
5. **SQL injection on port 80** — Implement input validation/parameterized queries on the web application, and consider a WAF as an additional layer of defense.

---

## 📚 CySA+ Exam Relevance

This lab directly maps to the following CompTIA CySA+ (CS0-003) exam domains:

| Domain | Coverage |

| Security Operations (33%) | Hands-on use of Nmap and NSE scripts to perform reconnaissance and identify live services and misconfigurations |
| Vulnerability Management (30%) | Running vulnerability-check scripts (`smb-vuln*`, `vuln`), interpreting results, and cross-referencing CVE/CVSS data from NVD |
| Incident Response (20%) | Recognizing confirmed exploitation (root access via vsFTPd backdoor) as evidence of active compromise |
| Reporting & Communication (17%) | Documenting findings by severity in a structured, readable format for technical and non-technical audiences |

---

## 🔑 Technical Notes

> # Note: not all NSE vulnerability scripts behave the same way — most only detect and report (true/false), while a small number (like ftp-vsftpd-backdoor) actually attempt live exploitation. Always check script documentation or output carefully rather than assuming uniform behavior.
>
> "Always add the -n flag to Nmap scans in this VMware environment to prevent DNS resolution hangs."]
> 
> -sP and -sT contradict eachother (Can't be used together because -sP means just do a ping/host-discovery sweep, skip ports entirely but -sT tells Nmap to do a full TCP connect port scan. These commands contradict eachother.)

```bash

# Any important commands or workarounds
[-sC]
[-Pn]

# Any important commands or workarounds

# Task 1: Run default scripts (-sC) against key ports
sudo nmap -sC -Pn --disable-arp-ping -n -p 21,22,80,445 192.168.79.130

# Task 2: Check FTP for anonymous login specifically
sudo nmap --script ftp-anon -Pn --disable-arp-ping -n -p 21 192.168.79.130

# Task 3: Run all SMB vulnerability scripts against port 445
sudo nmap --script smb-vuln* -Pn --disable-arp-ping -n -p 445 192.168.79.130

# Task 4: Run the full vuln script category across all open ports
sudo nmap --script vuln -Pn --disable-arp-ping -n 192.168.79.130

# Workaround: -p requires a value directly after it — omitting the port number
# (e.g. "-p 192.168.79.130") causes Nmap to misinterpret the target IP as a
# port specification. To scan all ports instead of a specific one, remove
# the -p flag entirely rather than leaving it empty.

# Workaround: always use -n in this lab to prevent DNS resolution hangs,
# since the isolated Host-Only network has no real DNS server.

# Note: --script <name>* (wildcard) runs every NSE script matching that
# name pattern — useful for running a whole category (e.g. smb-vuln*)
# in a single command instead of specifying each script individually.
```

---

## 📌 About This Project

[1-2 sentences about how this fits into your overall portfolio and career goals.]

This repository is part of my broader cybersecurity portfolio demonstrating practical, hands-on vulnerability assessment skills from initial scanning through confirmed exploitation as I work toward a career as a cybersecurity analyst.

This repository is important because here we have built up to the point of having live proof of the actual exploitation by an automated Nmap script for the CVE-2011-2523. In a past NVD task I researched this exact vulnerability with a CVSS score of 9.8 now this has come full circle and we found out how this exact vulnerability is exploited and root access was acheived. 


**Related repositories:**
- Nmap-Host-Discovery-and-Lab-Baseline — Established a baseline of the lab network using ping sweeps and host discovery, documenting the target environment before deeper scanning began
- Nmap-Scan-Types-SYN-vs-TCP-vs-UDP — Compared SYN, TCP connect, and UDP scan types against the target, examining differences in speed, stealth, and reliability
- Service-Version-Detection-and-OS-Fingerprinting — Identified exact software versions on Metasploitable and researched real CVEs tied to them, including the vsFTPd backdoor later exploited in this repository
- Wireshark-Capture-and-Analyze-Traffic — Captured and analyzed live packet traffic, including plaintext credential exposure over Telnet
- TCPDump-CLI-Packet-Capture — Used TCPDump from the command line to capture traffic (including a live Telnet session showing plaintext credential exposure), and demonstrated saving captures to a `.pcap` file for later analysis in Wireshark

---

## 👤 Author

**Dashane James**  
Senior Field Service Technician → Cybersecurity Analyst  
📍 Yonkers, NY  
🎓 B.S. Information Technology — SUNY Canton  
🏆 CompTIA Security+ | CySA+ (In Progress)  
🔗 [GitHub](https://github.com/Dashanejames1) | [Zero Trust Cyber Security Brand](https://www.instagram.com/zerotrust_cybersecurity)

---

*This repository is part of an active portfolio demonstrating hands-on cybersecurity skills. All lab work performed in isolated environments for educational purposes.*0
