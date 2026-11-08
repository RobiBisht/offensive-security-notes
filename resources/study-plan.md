# 5-Month Offensive Security Study Plan: Week-by-Week with Topics, Resources and Hours

This is the detailed version of the 5-month plan. For every week it tells you exactly what to learn, where to learn it, how many hours to spend, what to practice and what to produce. It assumes about 2 hours a day (about 14 hours a week) and that you already know Python basics.

## How to Use This Plan

**Weekly time budget (14 hours):**

- Learn: 5 hours (videos, reading, notes)
- Practice: 7 hours (labs, rooms, machines)
- Build and document: 2 hours (Python tools, write-ups, notes pushed to GitHub)

**Platforms used (all have free options):**

- TryHackMe (THM): guided rooms, best for beginners
- Hack The Box Academy (HTB Academy): deep modules, free tier available
- PortSwigger Web Security Academy: best free web hacking labs
- OverTheWire: Linux and wargames
- YouTube: Professor Messer, IppSec, Rana Khalil, John Hammond, The Cyber Mentor
- Reference sites: HackTricks, The Hacker Recipes, GTFOBins, LOLBAS, PayloadsAllTheThings

**Rules:**

- Only attack your own lab and legal platforms
- Try each challenge for at least 30 to 60 minutes before looking at a hint
- Write a short note or write-up after every week and push it to your GitHub notes repo
- If you fall behind, add a week to that phase instead of skipping topics

## MONTH 1: Foundations and Core Tools

### Week 1: Lab Setup and Linux

**Topics to learn:**

- Virtualization: what a VM is, snapshots, NAT vs host-only networks
- Linux file system, users, permissions (chmod, chown, SUID), processes, services, package managers
- Essential commands: ls, cd, cat, grep, find, awk, sed, curl, wget, ssh, scp, pipes and redirection

**Where to learn:**

- VirtualBox or VMware Workstation Player (install Kali Linux and Ubuntu from their official sites)
- TryHackMe: Linux Fundamentals 1, 2 and 3
- OverTheWire Bandit: levels 0 to 20
- Book (free PDF): The Linux Command Line by William Shotts

**Hours:** Setup 2h, learning 4h, practice 6h, notes 2h.

**Practice:** Do all Bandit levels 0 to 20. Install Metasploitable 2 as a vulnerable target and confirm Kali can ping it on a host-only network.

**Deliverable:** A GitHub notes repo created, with a `linux/` folder containing your command cheat sheet.

### Week 2: Networking in Practice

**Topics to learn:**

- OSI and TCP/IP models, IP addressing, subnetting, MAC, ARP, DNS, DHCP, NAT
- TCP three-way handshake, UDP, ICMP, common ports (21, 22, 23, 25, 53, 80, 110, 139, 443, 445, 3306, 3389)
- Packet analysis with Wireshark
- Port scanning with Nmap: SYN scan, service and version detection (-sV), OS detection (-O), scripts (--script), output formats

**Where to learn:**

- Professor Messer free Network+ videos (watch the networking fundamentals and ports sections; skip the rest)
- TryHackMe: Pre Security (networking modules), Network Fundamentals, Wireshark: The Basics, Nmap
- Nmap official reference guide (nmap.org/book)

**Hours:** Learning 5h, practice 7h, notes 2h.

**Practice:** Capture a TCP handshake, a DNS lookup, an HTTP login and an FTP login in Wireshark. Scan Metasploitable 2 with five different Nmap scan types and compare results.

**Deliverable:** A `networking/` folder in your notes repo with an Nmap cheat sheet and your Wireshark screenshots with explanations.

### Week 3: Windows, Security Concepts and Methodology

**Topics to learn:**

- Windows basics: registry, services, users and groups, NTFS permissions, event logs, PowerShell basics
- Security fundamentals: CIA triad, threat vs vulnerability vs risk, hashing, symmetric vs asymmetric encryption, TLS and certificates
- Pentest methodology: scoping, reconnaissance, scanning, exploitation, post-exploitation, reporting
- Frameworks: OWASP Top 10 overview, MITRE ATT&CK overview
- Legal basics: authorization, scope, the IT Act 2000 (India)

**Where to learn:**

- TryHackMe: Windows Fundamentals 1 to 3, Active Directory Basics, Introductory Researching, Principles of Security
- Professor Messer Security+ videos (threats, cryptography and PKI sections)
- attack.mitre.org (read the matrix overview)

**Hours:** Learning 6h, practice 5h, Python tool 3h.

**Practice:** Set up a Windows evaluation VM, create users and groups, view event logs and run basic PowerShell commands.

**Python tool:** Build a multithreaded port scanner with banner grabbing (socket, threading, argparse). Push it to its own GitHub repo with a README and disclaimer.

### Week 4: Recon, OSINT and First Machines

**Topics to learn:**

- Passive recon: whois, DNS enumeration (dig, nslookup), certificate transparency (crt.sh), Google dorks, Shodan, theHarvester
- Active recon: subdomain and directory enumeration (Gobuster, ffuf)
- How to read vulnerability scan output (Nessus Essentials or OpenVAS overview)
- Note-taking structure for engagements

**Where to learn:**

- TryHackMe: Passive Reconnaissance, Active Reconnaissance, Content Discovery, OhSINT
- Start the TryHackMe Jr Penetration Tester path (introduction modules)

**Hours:** Learning 4h, practice 8h, Python tool 2h.

**Practice:** Complete 3 to 4 easy THM rooms. Run recon on your own lab web server.

**Python tool:** Subdomain enumerator or directory brute-forcer.

**Month 1 checkpoint:** You can scan a network with Nmap, read a packet capture, navigate Linux comfortably, and have two small tools and a notes repo on GitHub.

## MONTH 2: Web Application Hacking

### Week 5: HTTP, Burp Suite and Injection

**Topics to learn:**

- How HTTP works: methods, headers, cookies, status codes, sessions, same-origin policy
- Burp Suite Community: proxy, repeater, intruder, decoder
- SQL injection: in-band, UNION-based, blind, and how to prevent it
- Basic SQL (SELECT, WHERE, UNION, comments)

**Where to learn:**

- TryHackMe: Web Fundamentals, Burp Suite: The Basics, SQL Injection
- PortSwigger Web Security Academy: SQL injection (read the theory, then do all labs)
- SQLBolt (free interactive SQL practice)

**Hours:** Learning 4h, practice 8h, notes 2h.

**Practice:** Install DVWA and OWASP Juice Shop (Docker or local). Complete the PortSwigger SQL injection labs by hand first; use sqlmap only afterwards.

### Week 6: XSS, Authentication and Access Control

**Topics to learn:**

- Cross-site scripting: reflected, stored, DOM-based, and the role of JavaScript
- Authentication flaws: weak passwords, brute force, password reset logic, 2FA bypass
- Broken access control and IDOR
- Basic JavaScript for reading and writing payloads

**Where to learn:**

- PortSwigger: XSS, Authentication, Access control labs
- freeCodeCamp or MDN JavaScript basics
- Rana Khalil's PortSwigger walkthroughs on YouTube (after you attempt a lab yourself)

**Hours:** Learning 4h, practice 8h, notes 2h.

**Practice:** Finish all apprentice and practitioner labs in these topics.

### Week 7: SSRF, File Attacks, CSRF and Logic Flaws

**Topics to learn:**

- Server-side request forgery (SSRF)
- File upload vulnerabilities, path traversal, file inclusion (LFI and RFI)
- OS command injection
- CSRF and CORS misconfigurations
- Business logic flaws

**Where to learn:**

- PortSwigger: SSRF, File upload, Path traversal, OS command injection, CSRF, CORS, Business logic labs
- TryHackMe: OWASP Top 10 room, File Inclusion, Upload Vulnerabilities

**Hours:** Learning 4h, practice 8h, notes 2h.

**Practice:** Solve 10 OWASP Juice Shop challenges and write short notes on each vulnerability class: what it is, how to find it, how to fix it.

### Week 8: APIs, JWT and a Web Project

**Topics to learn:**

- API testing and the OWASP API Top 10
- JWT attacks and token handling
- Writing a basic security report

**Where to learn:**

- PortSwigger: API testing, JWT, Business logic labs
- OWASP API Security Top 10 (owasp.org)
- Sample reports from TCM Security and HackerOne Hacktivity for report style

**Hours:** Learning 3h, practice 5h, project 6h.

**Project:** Build or extend a Python web vulnerability scanner (security header checks, simple SQLi and XSS probes) and test it against DVWA. Write a sample pentest report on DVWA or Juice Shop as a PDF.

**Month 2 checkpoint:** 40+ PortSwigger labs done, a working scanner on GitHub and one written web pentest report.

## MONTH 3: Network Pentesting, Exploitation and Privilege Escalation

### Week 9: Service Enumeration and Exploitation

**Topics to learn:**

- Enumerating SMB, FTP, SSH, SMTP, SNMP, NFS and database services
- Using Metasploit: modules, payloads, handlers, Meterpreter
- Finding exploits with searchsploit and exploiting manually
- Password attacks: Hydra, John the Ripper, Hashcat, wordlists

**Where to learn:**

- TryHackMe: Network Services 1 and 2, Metasploit: Introduction, Metasploit: Exploitation, Hydra, John the Ripper, Hashcat
- HTB Academy: Network Enumeration with Nmap, Footprinting, Attacking Common Services

**Hours:** Learning 4h, practice 8h, notes 2h.

**Practice:** Exploit Metasploitable 2 services (vsftpd, Samba, Tomcat) first with Metasploit, then manually.

### Week 10: Linux Privilege Escalation

**Topics to learn:**

- SUID/SGID binaries, sudo misconfigurations, cron jobs, writable files, capabilities, PATH hijacking, NFS no\_root\_squash, kernel exploits
- Shells: reverse and bind shells, stabilizing a shell, file transfer techniques

**Where to learn:**

- TryHackMe: Linux PrivEsc, Common Linux Privesc
- HTB Academy: Linux Privilege Escalation
- GTFOBins, HackTricks (Linux privilege escalation section)
- Tool: LinPEAS (use after learning the manual checks)

**Hours:** Learning 3h, practice 9h, notes 2h.

**Practice:** Root 4 easy machines on THM or HTB (retired).

### Week 11: Windows Privilege Escalation

**Topics to learn:**

- Unquoted service paths, weak service permissions, token impersonation, AlwaysInstallElevated, stored credentials, scheduled tasks, DLL hijacking

**Where to learn:**

- TryHackMe: Windows PrivEsc, Windows Privilege Escalation
- HTB Academy: Windows Privilege Escalation
- LOLBAS project, HackTricks (Windows local privilege escalation), PayloadsAllTheThings
- Tools: WinPEAS, PowerUp, Seatbelt

**Hours:** Learning 3h, practice 9h, notes 2h.

**Practice:** Complete the THM Windows PrivEsc room and 2 Windows easy machines.

### Week 12: Pivoting and Consolidation

**Topics to learn:**

- Port forwarding, SSH tunneling, SOCKS proxies with proxychains, chisel
- Working across multiple networks (pivoting from one machine to an internal one)

**Where to learn:**

- TryHackMe: Pivoting rooms (Wreath network is a good full practice)
- HTB Academy: Pivoting, Tunneling and Port Forwarding
- IppSec YouTube videos for how experienced testers think

**Hours:** Learning 3h, practice 9h, write-ups 2h.

**Practice:** Do 4 to 5 medium-level machines end to end with no hints for the first hour.

**Month 3 checkpoint:** You can take an easy machine from recon to root without a walkthrough and have 8+ written write-ups.

## MONTH 4: Active Directory and Windows Attacks

### Week 13: AD Theory and Lab

**Topics to learn:**

- Domains, forests, OUs, GPOs, trusts, domain controllers
- Kerberos and NTLM authentication in detail
- Security groups and delegation concepts

**Where to learn:**

- TryHackMe: Active Directory Basics, Windows Internals
- Microsoft Learn: Active Directory Domain Services overview
- The Hacker Recipes (AD sections)

**Hours:** Learning 6h, lab setup 6h, notes 2h.

**Practice:** Build a home AD lab: Windows Server evaluation (domain controller) plus two Windows 10/11 clients. Create users, groups and a GPO. GOAD (Game of Active Directory, on GitHub) is an advanced alternative.

### Week 14: AD Enumeration and Initial Access

**Topics to learn:**

- Enumeration with BloodHound, PowerView and ldapsearch
- LLMNR/NBT-NS poisoning with Responder
- Password spraying (carefully, to avoid lockouts)
- Kerberoasting and AS-REP roasting

**Where to learn:**

- TryHackMe: Attacktive Directory, Breaching Active Directory, Enumerating Active Directory
- HTB Academy: Active Directory Enumeration and Attacks (first modules)
- ired.team, adsecurity.org

**Hours:** Learning 4h, practice 8h, notes 2h.

**Practice:** Perform each attack in your own AD lab and note what the logs show.

### Week 15: Lateral Movement and Domain Compromise

**Topics to learn:**

- Pass-the-hash, pass-the-ticket, credential dumping, DCSync
- Impacket tools: secretsdump, psexec, wmiexec
- Common AD misconfigurations: ACL abuse, delegation, AD CS basics

**Where to learn:**

- HTB Academy: Active Directory Enumeration and Attacks (later modules)
- TryHackMe: Post-Exploitation Basics, Exploiting Active Directory
- Impacket GitHub documentation

**Hours:** Learning 3h, practice 9h, notes 2h.

### Week 16: Full Attack Chain

**Topics to learn:**

- Connecting everything: foothold, privilege escalation, lateral movement, domain admin, persistence concepts, cleanup
- Writing remediation for each AD finding

**Where to learn:**

- TryHackMe: Holo, Wreath or Throwback networks
- HTB Academy AD module skills assessment

**Hours:** Practice 10h, documentation 4h.

**Deliverable:** A written, step-by-step attack chain on your AD lab with remediation advice, published on GitHub.

**Month 4 checkpoint:** You can explain and demonstrate a full AD compromise and have it documented.

## MONTH 5: Specialization, Reporting and Job Readiness

### Weeks 17 and 18: One Specialization (Pick One)

**Option A: Cloud security**

- Learn: IAM, S3 and storage misconfigurations, security groups, metadata service abuse, basics of Kubernetes and containers
- Where: flAWS.cloud, flAWS2, CloudGoat (GitHub), Prowler and ScoutSuite (study their checks), AWS free tier, AWS and Azure official security docs
- Build: a Python (boto3) misconfiguration scanner

**Option B: Bug bounty**

- Learn: recon at scale, automation, writing good reports, program scope
- Where: HackerOne Hacktivity, Bugcrowd University, Intigriti, NahamSec videos
- Do: join a vulnerability disclosure program and try for a valid low-severity report

**Option C: Red team basics**

- Learn: C2 concepts, phishing, evasion basics, operational security
- Where: TryHackMe Red Team path, introductions to Sliver and Havoc

**Hours (both weeks):** Learning 4h, practice 6h, project 4h each week.

### Week 19: Exam Preparation and Mock Engagements

**Topics to learn:**

- Review weak areas from your write-ups
- Exam format and syllabus of your chosen cert (eJPT, PNPT or CPTS)

**Where to learn:**

- INE eJPT learning path or TCM Security PNPT courses, HTB CPTS path
- Timed multi-machine HTB or THM networks for practice

**Hours:** Review 4h, practice 10h.

**Practice:** Do a timed 8 to 12 hour mock pentest on a multi-machine network, then write a proper report.

### Week 20: Portfolio and Applications

**Topics to learn:**

- Resume writing for security roles, LinkedIn optimization, interview preparation (networking questions, web vulnerabilities, scenario questions)

**Where to learn:**

- Your own Portfolio and GitHub Guide document
- TCM Security and public HackerOne reports for report style
- Common pentest interview questions (search, then answer them in your own words)

**Hours:** Portfolio and applications 10h, interview practice 4h.

**Deliverable:** Polished GitHub repos, 8 to 10 write-ups, a sample report, an updated resume and LinkedIn, and at least 10 internship or junior job applications sent.

**Month 5 checkpoint:** First practical cert attempted, three portfolio projects, 10+ write-ups and applications sent.

## Weekly Rhythm (Repeat Every Week)

- Mon, Wed: learn theory (2 hours each)
- Tue, Thu, Fri: hands-on labs (2 hours each)
- Sat: bigger machine or project (3 to 4 hours)
- Sun: notes, write-up and plan next week (1 hour)

## Total Time Summary

| Month | Focus | Approx. hours |
| --- | --- | --- |
| 1 | Foundations and core tools | 56 |
| 2 | Web application hacking | 56 |
| 3 | Network pentesting and privilege escalation | 56 |
| 4 | Active Directory | 56 |
| 5 | Specialization and job readiness | 56 |
| Total |  | about 280 hours |

## Final Tips

- Finish each week's practice before moving on; watching videos alone will not build skill
- Take screenshots and notes as you work, because write-ups are much easier that way
- Ask for help in communities (TryHackMe and HTB Discords, r/AskNetsec) after you have tried yourself
- This plan gets you to solid junior level. OSCP-level skill takes longer, so keep practicing after month 5.
