# Networking Basics

My notes on networking for security. Practiced in my Kali and Ubuntu VMs.

## 1. Models and layers

### 1.1 OSI model (7 layers)

| Sr no. | Layer | Job | PDU | Examples / Devices |
|---|---|---|---|---|
| 7 | Application | Interface for apps and network services | Data | HTTP, HTTPS, FTP, SMTP, DNS, SSH |
| 6 | Presentation | Encoding, encryption, compression | Data | TLS/SSL, JPEG, ASCII, UTF-8 |
| 5 | Session | Opens, manages and closes sessions | Data | NetBIOS, RPC, SQL sessions |
| 4 | Transport | End-to-end delivery, ports, reliability | Segment / Datagram | TCP, UDP |
| 3 | Network | Logical addressing and routing | Packet | IP, ICMP, routers |
| 2 | Data Link | Hop-to-hop delivery using MAC addresses | Frame | Ethernet, Wi-Fi, ARP, switches |
| 1 | Physical | Bits on the wire or radio | Bits | Cables, hubs, NIC, radio signals |

### 1.2 TCP/IP model (4 layers)

| TCP/IP layer | Matches OSI | Protocols |
|---|---|---|
| Application | 5, 6, 7 | HTTP, HTTPS, DNS, DHCP, FTP, SSH, SMTP, Telnet |
| Transport | 4 | TCP, UDP |
| Internet | 3 | IP, ICMP, ARP (sits between layers 2 and 3) |
| Network Access (Link) | 1, 2 | Ethernet, Wi-Fi, MAC addressing |

### 1.3 Encapsulation: what happens when you open a website

When you type `http://example.com`, my computer first uses DNS to find the server's IP address, then completes a TCP handshake, and only then sends the HTTP request. The data travels down the layers on my machine, and each layer wraps it with its own header. This is **encapsulation**. The receiver unwraps it going up, which is **de-encapsulation**.

```
Application : GET /index.html HTTP/1.1                                 -> DATA
Transport   : [TCP hdr: src port 51522, dst port 80] + DATA            -> SEGMENT
Network     : [IP hdr: src 192.168.1.10, dst 203.0.113.10] + SEGMENT   -> PACKET
Data Link   : [Eth hdr: src MAC, dst MAC (gateway)] + PACKET + [FCS]   -> FRAME
Physical    : 10101101000110...                                        -> BITS
```

The destination MAC is the **gateway's** MAC, not the website's. MAC addresses change at every hop, while IP addresses stay the same from end to end, except when NAT rewrites the private source address on my router.

### 1.4 Worked examples

#### Example 1: Map real tools to layers

| Tool / event | Layer | Why |
|---|---|---|
| `ping` | 3 (Network) | Uses ICMP, which rides on IP |
| `nmap -sS` | 4 (Transport) | Crafts TCP SYN packets to ports |
| Wireshark filter `eth.addr` | 2 (Data Link) | Filters on MAC addresses |
| Burp Suite / SQL injection | 7 (Application) | Manipulates HTTP requests |
| A cut Ethernet cable | 1 (Physical) | Signal path is broken |
| HTTPS encryption | 6 (Presentation) | TLS encrypts and encodes data (the textbook OSI mapping) |

#### Example 2: Troubleshoot using layers (bottom-up)

- **Layer 1:** Is the cable plugged in? Is the Wi-Fi connected? Is the link light on?
- **Layer 2:** Does the device have a MAC visible in `arp -a`? Is it on the right VLAN?
- **Layer 3:** Do I have an IP? Can I ping `127.0.0.1`, then the gateway, then `8.8.8.8`?
- **Layer 4:** Is the port open? Use `nc -vz host 443` or `nmap -p 443 host`.
- **Layer 7:** Does the website itself respond? Use `curl -I https://example.com`.

#### Practical 1: See the layers on my own machine

Windows:

```
ipconfig /all          # Layer 3 (IP) and Layer 2 (MAC = Physical Address)
ping 8.8.8.8           # Layer 3 test (ICMP)
tracert google.com     # Layer 3 path
```

Linux / Kali:

```
ip addr                      # IP + MAC
ip route                     # default gateway
ping -c 4 8.8.8.8
traceroute google.com
curl -I http://example.com   # Layer 7 response headers
```

**Task:** Run `ping 8.8.8.8` while Wireshark captures. Expand one packet and identify the Ethernet, IP and ICMP headers. This is the layer stack in action.

**What I saw:** (write what you found, with a short screenshot caption)

## 2. IP addressing

An IP address is the logical address of a device on a network, like a postal address. It has a **network part** (which street) and a **host part** (which house).

### 2.1 IPv4 basics

- 32 bits, written as four octets in dotted decimal: `192.168.1.10`
- Each octet is 0 to 255 (8 bits), which gives about 4.3 billion addresses in total
- The subnet mask says which bits are network and which are host: `255.255.255.0` = `/24`

Binary view of `192.168.1.10`:

```
192      168      1        10
11000000.10101000.00000001.00001010
```

### 2.2 Classes (historical)

| Class | First octet | Default mask | Use |
|---|---|---|---|
| A | 1 to 126 | 255.0.0.0 (/8) | Very large networks |
| B | 128 to 191 | 255.255.0.0 (/16) | Medium networks |
| C | 192 to 223 | 255.255.255.0 (/24) | Small networks |
| D | 224 to 239 | n/a | Multicast |
| E | 240 to 255 | n/a | Experimental |

Today networks use **CIDR** (Classless Inter-Domain Routing, for example `/26`), but exam questions still ask about classes. The first octet 127 is skipped in Class A because it is reserved for loopback.

### 2.3 Private, public and special addresses

| Range | CIDR | Meaning |
|---|---|---|
| 10.0.0.0 to 10.255.255.255 | 10.0.0.0/8 | Private (RFC 1918) |
| 172.16.0.0 to 172.31.255.255 | 172.16.0.0/12 | Private (RFC 1918) |
| 192.168.0.0 to 192.168.255.255 | 192.168.0.0/16 | Private (RFC 1918) |
| 127.0.0.0 to 127.255.255.255 | 127.0.0.0/8 | Loopback (localhost) |
| 169.254.0.0 to 169.254.255.255 | 169.254.0.0/16 | APIPA: self-assigned when DHCP fails |
| 224.0.0.0 to 239.255.255.255 | 224.0.0.0/4 | Multicast |
| 255.255.255.255 | /32 | Limited broadcast |

Private addresses are not routed on the Internet. My router uses **NAT** (covered in the NAT section below) to share one public address among many private devices.

### 2.4 IPv6 in brief

- 128 bits in hexadecimal: `2001:0db8:85a3:0000:0000:8a2e:0370:7334`
- Shorten it by dropping leading zeros and replacing one run of zero groups with `::`, which gives `2001:db8:85a3::8a2e:370:7334`
- `::1` is loopback, `fe80::/10` is link-local, and `fc00::/7` is unique local (private)
- No broadcast and no ARP in IPv6. It uses multicast and **NDP** (Neighbor Discovery Protocol)

### 2.5 Examples

#### Example 1: Public or private?

| Address | Answer |
|---|---|
| 172.20.5.9 | Private (inside 172.16 to 172.31) |
| 172.32.0.1 | Public (just outside the private range) |
| 192.169.1.1 | Public (private is 192.168 only) |
| 169.254.12.8 | APIPA: the machine could not reach a DHCP server |
| 127.0.0.5 | Loopback (the whole 127/8 block loops back) |

#### Example 2: Find the class and default mask

- `10.4.7.2`: Class A, mask 255.0.0.0
- `150.30.1.1`: Class B, mask 255.255.0.0
- `200.5.5.5`: Class C, mask 255.255.255.0

#### Practical 2: Inspect addresses

Windows:

```
ipconfig
```

Linux / Kali:

```
ip -br addr
hostname -I
```

My public IP (shows the NAT router's address, not my private one):

```
curl ifconfig.me
```

**Task:** Compare my private IP (`ip addr`) with my public IP (`curl ifconfig.me`). They differ because of NAT.

**What I saw:** (write what you found, in your own words)


## 3. Subnetting

Subnetting splits one large network into smaller ones. Benefits: less broadcast traffic, better organisation, and security, because I can separate servers, staff and guests and apply firewall rules between them.

### 3.1 The core formulas

| Item | Formula |
|---|---|
| Host bits (h) | 32 minus the prefix length |
| Usable hosts per subnet | `2^h - 2` (minus the network and broadcast addresses) |
| Number of subnets | `2^s`, where s = borrowed bits |
| Block size (increment) | 256 minus the interesting octet of the mask |
| Network address | All host bits = 0 |
| Broadcast address | All host bits = 1 |
| Usable range | Network + 1 up to Broadcast - 1 |

The `2^h - 2` rule doesn't apply to `/31` (used for point-to-point links) and `/32` (a single host).

### 3.2 Cheat table (last octet)

| CIDR | Mask | Block size | Usable hosts |
|---|---|---|---|
| /24 | 255.255.255.0 | 256 | 254 |
| /25 | 255.255.255.128 | 128 | 126 |
| /26 | 255.255.255.192 | 64 | 62 |
| /27 | 255.255.255.224 | 32 | 30 |
| /28 | 255.255.255.240 | 16 | 14 |
| /29 | 255.255.255.248 | 8 | 6 |
| /30 | 255.255.255.252 | 4 | 2 |

### 3.3 Step-by-step examples

#### Example 1: Analyse 192.168.10.37/27

1. `/27` gives mask 255.255.255.224, and block size = 256 - 224 = 32.
2. Subnet blocks in the last octet: 0, 32, 64, 96, and so on.
3. 37 falls in the block starting at 32.

```
Network address : 192.168.10.32
First host      : 192.168.10.33
Last host       : 192.168.10.62
Broadcast       : 192.168.10.63
Usable hosts    : 2^5 - 2 = 30
```

#### Example 2: Split 192.168.1.0/24 into 4 equal subnets

I need 4 subnets, so I borrow 2 bits. The new prefix is `/26` (block size 64, 62 hosts each).

| Subnet | Network | Usable range | Broadcast |
|---|---|---|---|
| 1 | 192.168.1.0/26 | 192.168.1.1 to .62 | 192.168.1.63 |
| 2 | 192.168.1.64/26 | 192.168.1.65 to .126 | 192.168.1.127 |
| 3 | 192.168.1.128/26 | 192.168.1.129 to .190 | 192.168.1.191 |
| 4 | 192.168.1.192/26 | 192.168.1.193 to .254 | 192.168.1.255 |

#### Example 3: How many hosts? Pick the right prefix

- Need 100 hosts: `2^7 - 2 = 126`, so 7 host bits, which gives `/25`.
- Need 20 hosts: `2^5 - 2 = 30`, so 5 host bits, which gives `/27`.
- Need 2 hosts (a router link): 2 host bits, which gives `/30`.

#### Example 4: Are two hosts on the same subnet?

Host A = `10.1.1.130/26` and Host B = `10.1.1.190/26`. The block size is 64, so A is in the 128 to 191 block and B is in the 128 to 191 block. They are on the **same subnet**.

If B were `10.1.1.200`, it would be in the 192 to 255 block, a different subnet, so traffic would have to go through a router.

#### Example 5: VLSM (different sizes) for 192.168.5.0/24

Requirements: Servers 50 hosts, Staff 25 hosts, Guests 10 hosts, Router link 2 hosts. **Always allocate the largest segment first.**

| Segment | Needs | Prefix | Network | Range |
|---|---|---|---|---|
| Servers | 50 | /26 (62 hosts) | 192.168.5.0/26 | .1 to .62 |
| Staff | 25 | /27 (30 hosts) | 192.168.5.64/27 | .65 to .94 |
| Guests | 10 | /28 (14 hosts) | 192.168.5.96/28 | .97 to .110 |
| Router link | 2 | /30 (2 hosts) | 192.168.5.112/30 | .113 to .114 |

#### Practical 3: Verify with tools

Python (run it in Kali, Ubuntu or Git Bash):

```
python3 - <<'PY'
import ipaddress
n = ipaddress.ip_network('192.168.10.37/27', strict=False)
print(n.network_address, n.broadcast_address, n.netmask, n.num_addresses-2)
print(list(ipaddress.ip_network('192.168.1.0/24').subnets(new_prefix=26)))
PY
```

Linux: `ipcalc` (install it first with `sudo apt install ipcalc`):

```
ipcalc 192.168.10.37/27
```

**Task:** Subnet `172.16.0.0/16` into subnets of 500 hosts each.

**Answer:** 500 hosts need 9 host bits (`2^9 - 2 = 510`), so the prefix is `/23`. The subnets are `172.16.0.0/23`, `172.16.2.0/23`, `172.16.4.0/23`, and so on.

#### My own practice problems

(Add 3 subnetting problems I solved by hand, with the working, and check each with `ipcalc` or Python.)


## 4. Core protocols

Example IP addresses below are examples. In my own lab I replace them with the addresses of my VMs and my gateway. I only scan or test networks that I own or have permission to test.

### 4.1 MAC addresses

A MAC (Media Access Control) address is the hardware address of a network interface. It works at Layer 2 and is used to deliver frames between devices on the same local network.

#### Format and structure

- 48 bits written as 6 bytes in hex: `00:1A:2B:3C:4D:5E` (Linux and Mac use colons, Windows uses dashes: `00-1A-2B-3C-4D-5E`)
- First 3 bytes = **OUI** (Organizationally Unique Identifier), which identifies the manufacturer. Last 3 bytes = device-specific.
- `FF:FF:FF:FF:FF:FF` is the broadcast MAC. Every device on the LAN accepts it.
- If the 2nd hex digit of the first byte is 2, 6, A or E, the address is **locally administered** (often randomised or spoofed).

#### MAC vs IP

| Feature | MAC address | IP address |
|---|---|---|
| Layer | 2 (Data Link) | 3 (Network) |
| Scope | Local network only | End to end across networks |
| Assigned by | Manufacturer (can be changed in software) | DHCP or an administrator |
| Changes per hop? | Yes, rewritten at every router | No (except with NAT) |
| Length | 48 bits | 32 bits (IPv4) or 128 bits (IPv6) |

#### Example 1: Identify a vendor

MAC `B8:27:EB:12:34:56`: the OUI `B8:27:EB` belongs to the Raspberry Pi Foundation. This is how scanners such as Nmap report "Raspberry Pi" next to a device. (Vendors own many OUIs, so a device can have a different one.)

#### Example 2: How a switch uses MACs

A switch keeps a **CAM table** that maps MAC to port. It learns by reading the source MAC of each incoming frame. If the destination MAC is unknown, it **floods** the frame out of every port except the one it arrived on.

```
Port 1 -> AA:AA:AA:AA:AA:01   (PC1)
Port 2 -> AA:AA:AA:AA:AA:02   (PC2)
Port 3 -> AA:AA:AA:AA:AA:03   (Server)

PC1 sends to the Server  =>  the switch forwards it only out of Port 3.
```

#### Practical 4: View and change my MAC (only on my own lab VM)

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

### 4.2 ARP (Address Resolution Protocol)

IP says **who** I want to reach, and Ethernet needs a **MAC** to deliver the frame. ARP maps an IPv4 address to a MAC address on the local network.

#### How ARP works

1. PC A (`192.168.1.10`) wants to talk to `192.168.1.20` but doesn't know its MAC.
2. A checks its ARP cache. If there is no entry, it broadcasts: "Who has 192.168.1.20? Tell 192.168.1.10" (destination MAC `FF:FF:FF:FF:FF:FF`).
3. Only the owner of `192.168.1.20` replies (unicast): "192.168.1.20 is at BB:BB:BB:BB:BB:20".
4. A stores the result in its ARP cache and sends the frame.

To reach a host on a different network, A ARPs for the **default gateway's** MAC and sends the packet there.

#### Example

```
Frame 1  ARP Request  Who has 192.168.1.1? Tell 192.168.1.10
Frame 2  ARP Reply    192.168.1.1 is at c4:e9:84:5a:11:22
```

#### Practical 5: Work with the ARP cache

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

#### Practical 6: Watch ARP in Wireshark

- Start a capture on my Wi-Fi or Ethernet interface, with the display filter `arp`.
- Run `arp -d *` (Windows, as admin) and then `ping 192.168.1.1`.
- I should see one broadcast request and one unicast reply.

#### ARP spoofing (ARP poisoning): the attack

ARP has no authentication. Any host can send a fake reply, even without a request, and others will believe it. An attacker tells the victim "the gateway is at **my** MAC" and tells the gateway "the victim is at **my** MAC". Now all traffic flows through the attacker, which is a **man-in-the-middle (MITM)**.

```
Normal:   Victim <------------------> Gateway
Attack:   Victim <--> Attacker <--> Gateway
```

Tell-tale sign in `arp -a`: two IPs with the **same MAC address**.

- **Detection:** duplicate MACs in `arp -a`, Wireshark's Expert Info warning about a duplicate IP address, and tools such as `arpwatch`.
- **Defence:** Dynamic ARP Inspection (DAI) on managed switches, static ARP entries for critical hosts, port security, and using HTTPS or SSH so intercepted data stays encrypted.

### 4.3 DNS (Domain Name System)

DNS is the Internet's phonebook: it converts names (`google.com`) into IP addresses. It mostly uses **UDP port 53**, and **TCP port 53** for large responses and zone transfers.

#### The DNS hierarchy

```
                    .   (root servers)
            ________|________
           com      org      in      (TLD servers)
            |
        google.com                   (authoritative server)
            |
     www.google.com                  (host record)
```

#### How resolution works

1. I type `www.example.com`. The OS checks its local cache and the hosts file.
2. If the answer isn't there, it asks the **recursive resolver** (my ISP, my router, or `8.8.8.8`).
3. The resolver asks a **root server** and gets the address of the `.com` TLD server.
4. It asks the **TLD server** and gets the address of example.com's authoritative server.
5. It asks the **authoritative server** and gets the final IP.
6. The resolver returns the answer to me and caches it for the record's TTL.

#### Common record types

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

#### Practical 7: DNS commands

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

#### Security view of DNS

- **DNS enumeration:** attackers list subdomains (`dnsrecon`, `sublist3r`, `gobuster dns`) to find forgotten servers such as `dev.example.com`. Passive lookups are fine, but brute-forcing a domain I don't own needs permission.
- **Zone transfer (AXFR):** `dig axfr example.com @ns1.example.com`. A misconfigured server hands over its entire zone, and properly configured servers refuse. I only try this against servers I own or have permission to test.
- **DNS spoofing and cache poisoning:** forged answers send victims to fake sites. Defences: DNSSEC, DNS over HTTPS or TLS.
- **DNS tunnelling:** data hidden inside DNS queries to bypass firewalls. Look for very long or high-volume TXT queries.

#### Practical 8: See DNS in Wireshark

- Capture with the display filter `dns`, then run `nslookup example.com`.
- Open the query packet and note UDP with destination port 53. Open the response and look at the Answers section.

### 4.4 DHCP (Dynamic Host Configuration Protocol)

DHCP automatically gives devices an IP address, subnet mask, default gateway, DNS servers and a lease time. It uses UDP: **server port 67, client port 68**.

#### The DORA process

| Step | Message | Direction | What happens |
|---|---|---|---|
| 1 | Discover | Client to broadcast | "Is there any DHCP server? I need an IP." (source `0.0.0.0`, destination `255.255.255.255`) |
| 2 | Offer | Server to client | "You can have 192.168.1.50 for 24 hours." |
| 3 | Request | Client to broadcast | "I accept 192.168.1.50 from this server." |
| 4 | Acknowledge | Server to client | "Confirmed. The lease starts now." |

At 50% of the lease time the client tries to renew. If no server answers at all, Windows falls back to an APIPA address (`169.254.x.x`).

#### Practical 9: DHCP commands

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

#### DHCP attacks and defences

- **Rogue DHCP server:** an attacker answers faster and hands out a malicious gateway or DNS, which leads to MITM. Defence: **DHCP snooping** on switches, trusting only the real server's port.
- **DHCP starvation:** a flood of fake Discover messages exhausts the address pool. Defence: port security.

### 4.5 NAT (Network Address Translation)

NAT lets many devices with private IPs share one public IP. The router rewrites addresses as packets pass through. It also saved IPv4 from running out and hides internal addressing from the Internet.

#### Types of NAT

| Type | How it works | Use |
|---|---|---|
| Static NAT | One private IP to one fixed public IP | Publishing a server, such as a web server |
| Dynamic NAT | Private IPs mapped from a pool of public IPs | Rare today |
| PAT / NAT overload | Many private IPs to **one** public IP, told apart by port numbers | Home and office routers (the most common) |

#### PAT worked example

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

#### Port forwarding

Because NAT blocks unsolicited inbound traffic, hosting a service at home needs a rule such as "anything arriving at public port 8080 goes to 192.168.1.50:80". This is **port forwarding** (static PAT).

#### Practical 10: Observe NAT

```
ip addr | grep inet       # private IP, for example 192.168.1.10
curl ifconfig.me          # public IP, which is different
```

Run Wireshark on my PC and again on a VM behind another NAT. The source IP and port that the destination sees differ from what I sent.

**VirtualBox lab:** set a VM's network to **NAT**. It sits behind VirtualBox's NAT, gets `10.0.2.15`, can reach the Internet, but cannot be reached from outside. Switch to **Bridged** to place it directly on my LAN.

### 4.6 TCP, UDP and ICMP

#### TCP (Transmission Control Protocol)

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

This is the foundation of Nmap's port scanning (see the Nmap section).

**Other reliability features**

- **Sequence and ack numbers:** ordering and loss detection
- **Window size:** flow control, meaning how much data may be in flight
- **Retransmission:** lost segments are sent again after a timeout

#### UDP (User Datagram Protocol)

UDP is **connectionless and unreliable**: no handshake, no acknowledgements, no ordering. It is fast and light (an 8-byte header), which is ideal when speed matters more than perfection.

| Feature | TCP | UDP |
|---|---|---|
| Connection | Handshake first | None |
| Reliability | Guaranteed, ordered | Best effort |
| Speed and overhead | Slower, 20+ byte header | Faster, 8-byte header |
| Use cases | Web, email, file transfer, SSH | DNS, DHCP, VoIP, video streaming, games, TFTP, SNMP |
| Scanning | Easy: clear SYN, ACK and RST replies | Hard and slow: silence may mean open **or** filtered |

#### Practical 11: Try TCP and UDP with netcat

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

#### ICMP (Internet Control Message Protocol)

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

#### Practical 12: ICMP experiments

```
ping -c 4 8.8.8.8              # Linux (Windows: ping -n 4)
ping -c 4 -s 1000 8.8.8.8      # bigger payload
ping 127.0.0.1                 # test my own TCP/IP stack
sudo hping3 -1 192.168.1.1     # craft ICMP myself (my own lab only)
sudo nmap -sn 192.168.1.0/24   # ping sweep to find live hosts (my own network only)
```

A reply with TTL 64 usually means Linux or Mac, 128 usually Windows, and 255 usually network gear. This is a rough OS hint, because the TTL drops by one per hop.


