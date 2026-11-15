# Core Protocols

[Back to networking index](README.md)

Example IP addresses below are examples. In my own lab I replace them with the addresses of my VMs and my gateway. I only scan or test networks that I own or have permission to test.

## MAC addresses

A MAC (Media Access Control) address is the hardware address of a network interface. It works at Layer 2 and is used to deliver frames between devices on the same local network.

### Format and structure

- 48 bits written as 6 bytes in hex: `00:1A:2B:3C:4D:5E` (Linux and Mac use colons, Windows uses dashes: `00-1A-2B-3C-4D-5E`)
- First 3 bytes = **OUI** (Organizationally Unique Identifier), which identifies the manufacturer. Last 3 bytes = device-specific.
- `FF:FF:FF:FF:FF:FF` is the broadcast MAC. Every device on the LAN accepts it.
- If the 2nd hex digit of the first byte is 2, 6, A or E, the address is **locally administered** (often randomised or spoofed).

### MAC vs IP

| Feature | MAC address | IP address |
|---|---|---|
| Layer | 2 (Data Link) | 3 (Network) |
| Scope | Local network only | End to end across networks |
| Assigned by | Manufacturer (can be changed in software) | DHCP or an administrator |
| Changes per hop? | Yes, rewritten at every router | No (except with NAT) |
| Length | 48 bits | 32 bits (IPv4) or 128 bits (IPv6) |

### Example 1: Identify a vendor

MAC `B8:27:EB:12:34:56`: the OUI `B8:27:EB` belongs to the Raspberry Pi Foundation. This is how scanners such as Nmap report "Raspberry Pi" next to a device. (Vendors own many OUIs, so a device can have a different one.)

### Example 2: How a switch uses MACs

A switch keeps a **CAM table** that maps MAC to port. It learns by reading the source MAC of each incoming frame. If the destination MAC is unknown, it **floods** the frame out of every port except the one it arrived on.

```
Port 1 -> AA:AA:AA:AA:AA:01   (PC1)
Port 2 -> AA:AA:AA:AA:AA:02   (PC2)
Port 3 -> AA:AA:AA:AA:AA:03   (Server)

PC1 sends to the Server  =>  the switch forwards it only out of Port 3.
```

### Practical 4: View and change my MAC (only on my own lab VM)

View:

```
ipconfig /all        # Windows: Physical Address
ip link show         # Linux: link/ether
```

Temporarily spoof on Kali (resets on reboot). The interface may not be called `eth0`, so check the name with `ip link show` first:

```
sudo ip link set eth0 down
sudo ip link set eth0 address 02:11:22:33:44:55
sudo ip link set eth0 up
```

Or let `macchanger` pick a random one:

```
sudo macchanger -r eth0
```

## ARP (Address Resolution Protocol)

IP says **who** I want to reach, and Ethernet needs a **MAC** to deliver the frame. ARP maps an IPv4 address to a MAC address on the local network.

### How ARP works

1. PC A (`192.168.1.10`) wants to talk to `192.168.1.20` but doesn't know its MAC.
2. A checks its ARP cache. If there is no entry, it broadcasts: "Who has 192.168.1.20? Tell 192.168.1.10" (destination MAC `FF:FF:FF:FF:FF:FF`).
3. Only the owner of `192.168.1.20` replies (unicast): "192.168.1.20 is at BB:BB:BB:BB:BB:20".
4. A stores the result in its ARP cache and sends the frame.

To reach a host on a different network, A ARPs for the **default gateway's** MAC and sends the packet there.

### Example

```
Frame 1  ARP Request  Who has 192.168.1.1? Tell 192.168.1.10
Frame 2  ARP Reply    192.168.1.1 is at c4:e9:84:5a:11:22
```

### Practical 5: Work with the ARP cache

```
arp -a                  # show the cache (Windows, Linux, Mac)
ip neigh                # Linux alternative
arp -d 192.168.1.1      # delete an entry (needs admin on Windows, sudo on Linux)
ping 192.168.1.1        # forces a fresh ARP exchange
arp -a                  # the entry is back
```

Send my own ARP request (Linux):

```
sudo arping -c 3 192.168.1.1
```

Discover live hosts on my own LAN using ARP:

```
sudo nmap -sn 192.168.1.0/24
sudo netdiscover -r 192.168.1.0/24
```

### Practical 6: Watch ARP in Wireshark

- Start a capture on my Wi-Fi or Ethernet interface, with the display filter `arp`.
- Run `arp -d *` (Windows, as admin) and then `ping 192.168.1.1`.
- I should see one broadcast request and one unicast reply.

### ARP spoofing (ARP poisoning): the attack

ARP has no authentication. Any host can send a fake reply, even without a request, and others will believe it. An attacker tells the victim "the gateway is at **my** MAC" and tells the gateway "the victim is at **my** MAC". Now all traffic flows through the attacker, which is a **man-in-the-middle (MITM)**.

```
Normal:   Victim <------------------> Gateway
Attack:   Victim <--> Attacker <--> Gateway
```

Tell-tale sign in `arp -a`: two IPs with the **same MAC address**.

- **Detection:** duplicate MACs in `arp -a`, Wireshark's Expert Info warning about a duplicate IP address, and tools such as `arpwatch`.
- **Defence:** Dynamic ARP Inspection (DAI) on managed switches, static ARP entries for critical hosts, port security, and using HTTPS or SSH so intercepted data stays encrypted.

## DNS (Domain Name System)

DNS is the Internet's phonebook: it converts names (`google.com`) into IP addresses. It mostly uses **UDP port 53**, and **TCP port 53** for large responses and zone transfers.

### The DNS hierarchy

```
                    .   (root servers)
            ________|________
           com      org      in      (TLD servers)
            |
        google.com                   (authoritative server)
            |
     www.google.com                  (host record)
```

### How resolution works

1. I type `www.example.com`. The OS checks its local cache and the hosts file.
2. If the answer isn't there, it asks the **recursive resolver** (my ISP, my router, or `8.8.8.8`).
3. The resolver asks a **root server** and gets the address of the `.com` TLD server.
4. It asks the **TLD server** and gets the address of example.com's authoritative server.
5. It asks the **authoritative server** and gets the final IP.
6. The resolver returns the answer to me and caches it for the record's TTL.

### Common record types

| Record | Purpose | Example |
|---|---|---|
| A | Name to IPv4 address | `example.com` to `203.0.113.10` |
| AAAA | Name to IPv6 address | `example.com` to `2001:db8::10` |
| CNAME | Alias to another name | `www` to `example.com` |
| MX | Mail server for the domain | `10 mail.example.com` |
| NS | Authoritative name servers | `ns1.example.com` |
| TXT | Free text: SPF, DKIM, verification | `"v=spf1 include:_spf.google.com ~all"` |
| PTR | Reverse lookup: IP to name | `10.113.0.203.in-addr.arpa` |
| SOA | Zone administration info | Serial, refresh, retry |

### Practical 7: DNS commands

```
nslookup example.com                  # basic lookup (Windows and Linux)
nslookup -type=MX gmail.com           # mail servers
nslookup 8.8.8.8                      # reverse lookup

dig example.com                       # full detail (Linux and Kali)
dig example.com A +short
dig example.com MX
dig example.com NS
dig example.com TXT
dig @8.8.8.8 example.com              # ask a specific server
dig +trace example.com                # watch root, then TLD, then authoritative
dig -x 8.8.8.8                        # reverse lookup

host -t ns example.com
ipconfig /displaydns                  # Windows: show the cache
ipconfig /flushdns                    # Windows: clear the cache
```

Sample `dig` output (trimmed, with a documentation address):

```
;; QUESTION SECTION:
;example.com.             IN   A
;; ANSWER SECTION:
example.com.      3600    IN   A    203.0.113.10
;; Query time: 24 msec
;; SERVER: 192.168.1.1#53
```

How to read it: the record type is A, the answer is `203.0.113.10`, it may be cached for 3600 seconds, and the server that answered was `192.168.1.1`.

### Security view of DNS

- **DNS enumeration:** attackers list subdomains (`dnsrecon`, `sublist3r`, `gobuster dns`) to find forgotten servers such as `dev.example.com`. Passive lookups are fine, but brute-forcing a domain I don't own needs permission.
- **Zone transfer (AXFR):** `dig axfr example.com @ns1.example.com`. A misconfigured server hands over its entire zone, and properly configured servers refuse. I only try this against servers I own or have permission to test.
- **DNS spoofing and cache poisoning:** forged answers send victims to fake sites. Defences: DNSSEC, DNS over HTTPS or TLS.
- **DNS tunnelling:** data hidden inside DNS queries to bypass firewalls. Look for very long or high-volume TXT queries.

### Practical 8: See DNS in Wireshark

- Capture with the display filter `dns`, then run `nslookup example.com`.
- Open the query packet and note UDP with destination port 53. Open the response and look at the Answers section.

## DHCP (Dynamic Host Configuration Protocol)

DHCP automatically gives devices an IP address, subnet mask, default gateway, DNS servers and a lease time. It uses UDP: **server port 67, client port 68**.

### The DORA process

| Step | Message | Direction | What happens |
|---|---|---|---|
| 1 | Discover | Client to broadcast | "Is there any DHCP server? I need an IP." (source `0.0.0.0`, destination `255.255.255.255`) |
| 2 | Offer | Server to client | "You can have 192.168.1.50 for 24 hours." |
| 3 | Request | Client to broadcast | "I accept 192.168.1.50 from this server." |
| 4 | Acknowledge | Server to client | "Confirmed. The lease starts now." |

At 50% of the lease time the client tries to renew. If no server answers at all, Windows falls back to an APIPA address (`169.254.x.x`).

### Practical 9: DHCP commands

Windows:

```
ipconfig /all          # see DHCP Server, Lease Obtained, Lease Expires
ipconfig /release      # give up the address
ipconfig /renew        # run DORA again
```

Linux:

```
sudo dhclient -r eth0     # release
sudo dhclient eth0        # request
nmcli device show eth0    # lease details (NetworkManager)
```

Nmap can look for DHCP servers on the LAN:

```
sudo nmap --script broadcast-dhcp-discover
```

**Task:** Capture with the display filter `udp.port == 67 || udp.port == 68` (newer Wireshark also accepts `dhcp`, older versions use `bootp`). Run `ipconfig /release` and then `/renew`, and identify the four DORA packets. Running `/release` drops my connection for a moment, so I do this on a machine I'm happy to disconnect.

### DHCP attacks and defences

- **Rogue DHCP server:** an attacker answers faster and hands out a malicious gateway or DNS, which leads to MITM. Defence: **DHCP snooping** on switches, trusting only the real server's port.
- **DHCP starvation:** a flood of fake Discover messages exhausts the address pool. Defence: port security.

## NAT (Network Address Translation)

NAT lets many devices with private IPs share one public IP. The router rewrites addresses as packets pass through. It also saved IPv4 from running out and hides internal addressing from the Internet.

### Types of NAT

| Type | How it works | Use |
|---|---|---|
| Static NAT | One private IP to one fixed public IP | Publishing a server, such as a web server |
| Dynamic NAT | Private IPs mapped from a pool of public IPs | Rare today |
| PAT / NAT overload | Many private IPs to **one** public IP, told apart by port numbers | Home and office routers (the most common) |

### PAT worked example

```
Inside host           Router (public IP 203.0.113.5)       Outside server
192.168.1.10:51000 -> 203.0.113.5:40001 ------------------> 198.51.100.20:80
192.168.1.11:51000 -> 203.0.113.5:40002 ------------------> 198.51.100.20:80

NAT translation table
Inside local          Inside global        Outside
192.168.1.10:51000    203.0.113.5:40001    198.51.100.20:80
192.168.1.11:51000    203.0.113.5:40002    198.51.100.20:80
```

Replies come back to `203.0.113.5:40001`, and the router looks it up in the table and forwards it to `192.168.1.10:51000`.

### Port forwarding

Because NAT blocks unsolicited inbound traffic, hosting a service at home needs a rule such as "anything arriving at public port 8080 goes to 192.168.1.50:80". This is **port forwarding** (static PAT).

### Practical 10: Observe NAT

```
ip addr | grep inet       # private IP, for example 192.168.1.10
curl ifconfig.me          # public IP, which is different
```

Run Wireshark on my PC and again on a VM behind another NAT. The source IP and port that the destination sees differ from what I sent.

**VirtualBox lab:** set a VM's network to **NAT**. It sits behind VirtualBox's NAT, gets `10.0.2.15`, can reach the Internet, but cannot be reached from outside. Switch to **Bridged** to place it directly on my LAN.

## TCP, UDP and ICMP

### TCP (Transmission Control Protocol)

TCP is **connection-oriented and reliable**: it sets up a connection, numbers every byte, acknowledges receipt, retransmits lost data and controls flow. It is used by HTTP(S), SSH, FTP and SMTP.

**TCP header flags**

| Flag | Meaning |
|---|---|
| SYN | Synchronise: start a connection |
| ACK | Acknowledge received data |
| FIN | Finish: gracefully close |
| RST | Reset: abruptly kill or reject the connection |
| PSH | Push data to the application now |
| URG | The urgent data pointer is valid |

**The three-way handshake**

```
Client                                   Server (listening on port 80)
  | ---- SYN, seq=100 ------------------>  |   1. "I want to talk, my seq starts at 100"
  | <--- SYN+ACK, seq=300, ack=101 -------  |   2. "OK, my seq starts at 300, I got 100"
  | ---- ACK, seq=101, ack=301 ----------->  |   3. "Got yours. Connection established"
  |                                          |
  | ======== data flows both ways ========== |
```

The ack number is always the **next byte expected**, so ack = seq + 1 during the handshake.

**Closing a connection (four-way)**

```
Client -- FIN, ACK --> Server
Client <-- ACK ------- Server
Client <-- FIN, ACK -- Server
Client -- ACK -------> Server      (the client waits in TIME_WAIT, then closes)
```

**What a closed or open port looks like**

| Scenario | Server reply to SYN | Meaning |
|---|---|---|
| Port open | SYN+ACK | A service is listening |
| Port closed | RST (+ACK) | The host is up, nothing is listening |
| Port filtered | No reply, or ICMP unreachable | A firewall is dropping the packet |

This is the foundation of Nmap's port scanning (see [Nmap notes](06-nmap.md)).

**Other reliability features**

- **Sequence and ack numbers:** ordering and loss detection
- **Window size:** flow control, meaning how much data may be in flight
- **Retransmission:** lost segments are sent again after a timeout

### UDP (User Datagram Protocol)

UDP is **connectionless and unreliable**: no handshake, no acknowledgements, no ordering. It is fast and light (an 8-byte header), which is ideal when speed matters more than perfection.

| Feature | TCP | UDP |
|---|---|---|
| Connection | Handshake first | None |
| Reliability | Guaranteed, ordered | Best effort |
| Speed and overhead | Slower, 20+ byte header | Faster, 8-byte header |
| Use cases | Web, email, file transfer, SSH | DNS, DHCP, VoIP, video streaming, games, TFTP, SNMP |
| Scanning | Easy: clear SYN, ACK and RST replies | Hard and slow: silence may mean open **or** filtered |

### Practical 11: Try TCP and UDP with netcat

TCP: run the server in terminal 1 and the client in terminal 2. Text typed in either window appears in the other.

```
nc -lvnp 4444              # terminal 1 (server)
nc 127.0.0.1 4444          # terminal 2 (client)
```

If `-p` gives an error with `-l` on my system, use `nc -lvn 4444` instead.

UDP version:

```
nc -u -lvnp 5555           # terminal 1 (server)
nc -u 127.0.0.1 5555       # terminal 2 (client)
```

Test whether a port is open, using Nmap's official test host (a few requests only):

```
nc -vz scanme.nmap.org 22 80 443
```

See listening ports on my own machine:

```
netstat -ano               # Windows
ss -tulpn                  # Linux (t=tcp u=udp l=listening p=process n=numeric)
```

### ICMP (Internet Control Message Protocol)

ICMP carries error and diagnostic messages for IP. It has no ports. It is a Layer 3 protocol used by `ping` and `traceroute`.

| Type | Code | Name | Where I see it |
|---|---|---|---|
| 0 | 0 | Echo Reply | Reply to ping |
| 8 | 0 | Echo Request | `ping` |
| 3 | 0, 1, 3 | Destination Unreachable (net, host, port) | Closed UDP port, no route |
| 11 | 0 | Time Exceeded (TTL = 0) | `traceroute` |
| 5 | 0 | Redirect | A router suggests a better gateway |

**How traceroute works:** it sends packets with TTL 1, 2, 3, and so on. Each router decrements the TTL, and the router where it hits 0 sends back ICMP Time Exceeded, which reveals its address. Repeating this maps every hop.

```
tracert example.com                       # Windows (ICMP)
traceroute example.com                    # Linux (UDP by default)
traceroute -I example.com                 # Linux using ICMP
sudo traceroute -T -p 443 example.com     # using TCP port 443 (passes more firewalls)
```

### Practical 12: ICMP experiments

```
ping -c 4 8.8.8.8              # Linux (Windows: ping -n 4)
ping -c 4 -s 1000 8.8.8.8      # bigger payload
ping 127.0.0.1                 # test my own TCP/IP stack
sudo hping3 -1 192.168.1.1     # craft ICMP myself (my own lab only)
sudo nmap -sn 192.168.1.0/24   # ping sweep to find live hosts (my own network only)
```

A reply with TTL 64 usually means Linux or Mac, 128 usually Windows, and 255 usually network gear. This is a rough OS hint, because the TTL drops by one per hop.

## Common ports

A port (0 to 65535) identifies a service on a host. An IP address plus a port is a **socket**. Ports 0 to 1023 are **well-known**, 1024 to 49151 are **registered**, and 49152 to 65535 are **dynamic or ephemeral** (my OS picks one as the source port for outgoing connections).

### The ports from my syllabus

| Port | Protocol | Service | Notes and security angle |
|---|---|---|---|
| 21 | TCP | FTP (control) | Clear-text login. Check for anonymous login; use SFTP instead. |
| 22 | TCP | SSH | Encrypted remote shell. Target of brute-force attacks; use keys. |
| 23 | TCP | Telnet | Clear-text remote shell. Obsolete and dangerous. |
| 25 | TCP | SMTP | Sending and relaying email. Check for open relay and user enumeration (`VRFY`). Mail clients usually submit on 587 or 465. |
| 53 | UDP/TCP | DNS | UDP for queries, TCP for zone transfers. |
| 80 | TCP | HTTP | Unencrypted web. Web app attacks (SQLi, XSS). |
| 110 | TCP | POP3 | Clear-text mail retrieval (995 = POP3S). |
| 139 | TCP | NetBIOS Session | Old Windows file sharing (SMB over NetBIOS). |
| 443 | TCP | HTTPS | Web over TLS. |
| 445 | TCP | SMB | Windows file sharing. EternalBlue and WannaCry exploited it. |
| 3306 | TCP | MySQL | Database. Should never be exposed to the Internet. |
| 3389 | TCP | RDP | Windows Remote Desktop. Target of brute-force attacks and BlueKeep. |

### Other ports worth knowing

| Port | Service | Port | Service |
|---|---|---|---|
| 20 | FTP data | 67 / 68 | DHCP (UDP) |
| 69 | TFTP (UDP) | 88 | Kerberos |
| 123 | NTP (UDP) | 143 | IMAP |
| 161 | SNMP (UDP) | 389 / 636 | LDAP / LDAPS |
| 1433 | MS SQL Server | 5432 | PostgreSQL |
| 5900 | VNC | 8080 | Alternate HTTP and proxies |

### Port states (as Nmap reports them)

| State | Meaning |
|---|---|
| `open` | A service is actively accepting connections |
| `closed` | Reachable, but no service is listening (RST received) |
| `filtered` | A firewall or filter blocks the probe, so Nmap can't tell |
| `open\|filtered` | No response; could be open or filtered (common with UDP) |
| `unfiltered` | Reachable, but Nmap can't tell if it is open or closed (ACK scan) |

### Practical 13: See ports on my own computer

```
netstat -ano | findstr LISTENING     # Windows
ss -tulpn                            # Linux
sudo lsof -i -P -n | grep LISTEN     # Linux and Mac
```

Start a quick web server and find it:

```
python3 -m http.server 8000
```

In another terminal:

```
ss -tlpn | grep 8000
curl http://127.0.0.1:8000
```

Look up well-known ports in the services file. On Windows it is `C:\Windows\System32\drivers\etc\services`. On Linux it is `/etc/services`:

```
grep -w 443 /etc/services
```