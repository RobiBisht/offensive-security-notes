# Port Scanning with Nmap

[Back to networking index](README.md)

Nmap (Network Mapper) is the standard tool for discovering hosts, open ports, running services, operating systems and known weaknesses. It is the first tool most ethical hackers run during reconnaissance. I only scan my own lab machines, or targets I have written permission to test. `scanme.nmap.org` is Nmap's official test host, and I send it only a few requests.

## Basic syntax

```
nmap [scan type] [options] <target>
```

Ways to give a target:

```
nmap 192.168.1.10                              # single IP
nmap scanme.nmap.org                           # hostname
nmap 192.168.1.0/24                            # whole subnet
nmap 192.168.1.1-50                            # range
nmap -iL targets.txt                           # targets from a file
nmap 192.168.1.0/24 --exclude 192.168.1.1      # skip one address
```

## Host discovery

```
nmap -sn 192.168.1.0/24      # ping sweep only, no port scan (list live hosts)
nmap -Pn 192.168.1.10        # skip discovery, assume the host is up (when ICMP is blocked)
nmap -PR 192.168.1.0/24      # ARP discovery (default and most reliable on a LAN)
nmap -sL 192.168.1.0/29      # just list targets, send nothing
```

## Specifying ports

```
nmap 192.168.1.10                 # default: top 1000 TCP ports
nmap -p 22,80,443 192.168.1.10    # specific ports
nmap -p 1-1024 192.168.1.10       # range
nmap -p- 192.168.1.10             # all 65535 ports
nmap -F 192.168.1.10              # fast: top 100 ports
nmap --top-ports 20 192.168.1.10
sudo nmap -sU -sS -p U:53,161,T:21-25,80 192.168.1.10   # mixed UDP and TCP
```

For a mixed UDP and TCP list, I need to ask for both scan types (`-sU` and `-sS`).

## Scan types

### SYN scan: `-sS` (the default as root, also called "stealth" or "half-open")

Nmap sends a SYN. If it gets SYN+ACK, the port is open, and Nmap then sends RST instead of ACK, so the handshake is never completed. It is fast and is less likely to be logged by the application.

```
Open port:      Nmap --SYN-->  Target
                Nmap <-SYN/ACK- Target
                Nmap --RST-->  Target     (connection never completes)

Closed port:    Nmap --SYN-->  Target
                Nmap <---RST--- Target

Filtered port:  Nmap --SYN-->  (no reply, or ICMP unreachable)
```

```
sudo nmap -sS 192.168.1.10
```

### TCP connect scan: `-sT` (the default without root)

It uses the operating system's full `connect()` call and completes the three-way handshake, so it is slower and gets logged more easily.

```
nmap -sT 192.168.1.10
```

### UDP scan: `-sU`

It is slow, because UDP services often don't reply. No reply gives `open|filtered`, and an ICMP port-unreachable gives `closed`. Always limit the port range.

```
sudo nmap -sU --top-ports 20 192.168.1.10
sudo nmap -sU -p 53,67,69,123,161 192.168.1.10
```

### Other scan types (know they exist)

| Option | Name | Idea |
|---|---|---|
| `-sA` | ACK scan | Maps firewall rules: reports filtered or unfiltered |
| `-sF`, `-sN`, `-sX` | FIN, Null, Xmas | Odd flag combinations. Closed ports reply RST and open ports stay silent (works on RFC-compliant systems, not Windows). |
| `-sW` | Window scan | Like ACK, but inspects the TCP window size |
| `-sI` | Idle (zombie) scan | Scans via a third host, hiding my IP |
| `-sn` | Ping scan | Host discovery only |

## Service and version detection: `-sV`

Open port numbers only **suggest** a service. `-sV` connects to each open port, reads the banner and sends probes to identify the real software and version. Versions let me search for known vulnerabilities (CVEs).

```
sudo nmap -sV 192.168.1.10
sudo nmap -sV --version-intensity 9 192.168.1.10   # try harder (0 to 9)
sudo nmap -sV --version-light 192.168.1.10         # faster, less accurate
```

Example output (Metasploitable 2, trimmed):

```
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4
22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
23/tcp   open  telnet      Linux telnetd
25/tcp   open  smtp        Postfix smtpd
80/tcp   open  http        Apache httpd 2.2.8 ((Ubuntu) DAV/2)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X
445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X
3306/tcp open  mysql       MySQL 5.0.51a-3ubuntu5
```

How to read it: `vsftpd 2.3.4` is a famous backdoored release, and Apache 2.2.8 and OpenSSH 4.7 are very old. These are leads for the next phase, such as running `searchsploit vsftpd 2.3.4` or checking the CVE databases.

## OS detection: `-O`

Nmap sends specially crafted probes and compares how the TCP/IP stack responds (TTL, window size, options, flags) with a fingerprint database. It needs root and at least one open and one closed port to be accurate.

```
sudo nmap -O 192.168.1.10
sudo nmap -O --osscan-guess 192.168.1.10   # show best guesses
sudo nmap -A 192.168.1.10                  # aggressive: -sV -O -sC --traceroute
```

Sample output:

```
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:linux_kernel:2.6
OS details: Linux 2.6.9 - 2.6.33
Network Distance: 1 hop
```

Results are estimates, and firewalls, VMs and NAT can distort them.

## Nmap Scripting Engine (NSE): `--script`

NSE runs Lua scripts for discovery, brute force, vulnerability checks and more. Scripts live in `/usr/share/nmap/scripts/`. They are grouped into categories: `default`, `safe`, `discovery`, `version`, `vuln`, `auth`, `brute`, `exploit`, `intrusive`, `malware`. Categories like `brute`, `exploit` and `intrusive` can disrupt or damage a target, so I only run them on my own lab machines.

```
nmap -sC 192.168.1.10                              # run default scripts (same as --script=default)
nmap --script=vuln 192.168.1.10                    # known-vulnerability checks
nmap --script=http-title,http-headers -p 80 192.168.1.10
nmap --script=ftp-anon -p 21 192.168.1.10          # is anonymous FTP allowed?
nmap --script=smb-os-discovery -p 445 192.168.1.10
nmap --script=ssh-auth-methods -p 22 192.168.1.10
nmap --script=dns-zone-transfer -p 53 --script-args dns-zone-transfer.domain=example.com ns1.example.com
nmap --script "http-*" -p 80 192.168.1.10          # wildcard
nmap --script-help ftp-anon                        # read the documentation
ls /usr/share/nmap/scripts | grep smb              # find scripts
sudo nmap --script-updatedb                        # refresh the script index
```

Example output:

```
PORT   STATE SERVICE
21/tcp open  ftp
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxr-xr-x   2 ftp   ftp   4096 Mar 01  2024 pub
```

## Output formats: `-oN`, `-oX`, `-oG`, `-oA`

| Option | Format | Best for |
|---|---|---|
| `-oN file.txt` | Normal text | Reading and reports |
| `-oX file.xml` | XML | Importing into tools (Metasploit, Zenmap, parsers) |
| `-oG file.gnmap` | Grepable | Quick filtering with grep or awk (deprecated but handy) |
| `-oA basename` | All three at once | Best habit: creates `.nmap`, `.xml` and `.gnmap` |
| `-oS file` | Script kiddie | Just for fun |

Create the output folder first, then scan and filter the results:

```
mkdir scans
sudo nmap -sV -oA scans/metasploitable 192.168.56.101
grep "open" scans/metasploitable.gnmap

# list only hosts with port 22 open
grep "22/open" scans/metasploitable.gnmap | cut -d' ' -f2

# convert XML to an HTML report (install first: sudo apt install xsltproc)
xsltproc scans/metasploitable.xml -o scans/report.html
```

## Timing, verbosity and evasion basics

| Option | Purpose |
|---|---|
| `-T0` to `-T5` | Timing template: 0 paranoid (very slow), 3 normal (default), 4 aggressive, 5 insane. Use `-T4` in labs. |
| `-v` / `-vv` | Verbose: show open ports as they are found |
| `-d` | Debug output |
| `--reason` | Explain why each port has its state |
| `--open` | Show only open ports |
| `--packet-trace` | Show every packet sent and received |
| `-D RND:5` | Decoy scan, mixes my IP among fake sources |
| `-f` | Fragment packets (older firewalls) |
| `--source-port 53` | Spoof the source port (may bypass poorly written rules) |
| `--min-rate 1000` | Send at least 1000 packets per second |

Evasion options are for understanding how firewalls and IDS behave in my own lab. I don't use them against systems I don't have permission to test.

## Practical lab: scan my own network step by step

**Setup:** In VirtualBox, run Kali and Metasploitable 2 on a Host-Only or Internal network (for example `192.168.56.0/24`) so they are isolated from the Internet. Find Metasploitable's IP by logging in (the default credentials for that lab VM are `msfadmin` / `msfadmin`) and running `ifconfig`.

1. **Discover live hosts:** `sudo nmap -sn 192.168.56.0/24`, then note the target IP (say `192.168.56.101`).
2. **Quick scan:** `sudo nmap -F 192.168.56.101`
3. **All ports and service versions:** `sudo nmap -p- -sV -T4 -oA scans/meta 192.168.56.101`
4. **OS guess:** `sudo nmap -O 192.168.56.101`
5. **Default scripts and a vulnerability check:** `sudo nmap -sC --script=vuln -p 21,22,80,445 192.168.56.101`
6. **UDP top ports:** `sudo nmap -sU --top-ports 20 192.168.56.101`
7. **Compare scan types:** run `sudo nmap -sT` and `sudo nmap -sS` with Wireshark open. In the SYN scan, I will see RST replacing the final ACK.
8. **Document:** open `scans/meta.nmap`, then write one line per open port: service, version, risk and recommended fix.

## Mini report template

| Port | Service / Version | Risk | Recommendation |
|---|---|---|---|
| 21/tcp | vsftpd 2.3.4 | Backdoored version; anonymous login possible | Upgrade or replace; disable anonymous access; use SFTP |
| 23/tcp | Linux telnetd | Clear-text credentials | Disable Telnet; use SSH |
| 445/tcp | Samba 3.x | Old SMB, multiple known CVEs | Patch; restrict by firewall |
| 3306/tcp | MySQL 5.0.51a | Database exposed on the network | Bind to localhost; use a firewall |

## Nmap cheat sheet

| Task | Command |
|---|---|
| Live hosts on LAN | `sudo nmap -sn 192.168.1.0/24` |
| Quick top ports | `nmap -F <ip>` |
| Full TCP port scan | `sudo nmap -p- -T4 <ip>` |
| Stealth SYN scan | `sudo nmap -sS <ip>` |
| Versions | `sudo nmap -sV <ip>` |
| OS detection | `sudo nmap -O <ip>` |
| Aggressive all-in-one | `sudo nmap -A -T4 <ip>` |
| Default scripts | `nmap -sC <ip>` |
| Vulnerability scripts | `nmap --script vuln <ip>` |
| UDP scan | `sudo nmap -sU --top-ports 20 <ip>` |
| No ping (firewalled host) | `nmap -Pn <ip>` |
| Save everything | `nmap -oA name <ip>` |
| Typical first scan in practice | `sudo nmap -sC -sV -T4 -oA first <ip>` |