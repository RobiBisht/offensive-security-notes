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
