# Networking Practice Solutions

[Back to networking index](README.md)

Solutions for the 15 exercises in [Practice exercises](08-practice-exercises.md). Try each one on my own before reading the answer.

> I practiced on Kali Linux in WSL2 and on Windows. Output such as IP addresses, DNS answers and TTL values depends on my network and changes over time, so my own results may differ from the examples here.

## 1. Encapsulation

**Question:** I open `http://example.com`. From the top, what does each layer do or add, what is the PDU, and whose MAC address is the destination in the frame?

### Solution

First DNS finds the server's address and TCP completes its handshake. Then the data goes down the layers on my machine, and each layer adds its own header.

| Layer | What it does or adds | PDU |
|---|---|---|
| 7 Application | The browser creates the HTTP request (`GET / HTTP/1.1`) | Data |
| 4 Transport | TCP adds the source port (a random high port), the destination port 80 and sequence numbers | Segment |
| 3 Network | IP adds my source IP address and the server's destination IP address | Packet |
| 2 Data Link | Ethernet adds my source MAC, the destination MAC and a trailer (FCS) | Frame |
| 1 Physical | The frame is sent as electrical, light or radio signals | Bits |

The destination MAC is the **default gateway's**, not the server's. A MAC address only works on the local network, and it is rewritten at every hop. IP addresses stay the same from end to end, except where NAT rewrites my private source address.

## 2. Layers and troubleshooting

**Question:** Name the OSI layer for six tools or events, and write a bottom-up checklist for "the website does not load".

### Solution

| Tool or event | Layer | Why |
|---|---|---|
| `ping` | 3 (Network) | It uses ICMP, which runs on IP |
| `nmap -sS` | 4 (Transport) | It sends TCP SYN packets to ports |
| Wireshark filter `eth.addr` | 2 (Data Link) | It filters on MAC addresses |
| SQL injection | 7 (Application) | It manipulates the input of an application |
| TLS encryption | 6 (Presentation) | The textbook OSI mapping for encryption and encoding |
| A cut Ethernet cable | 1 (Physical) | The signal path is broken |

Bottom-up checklist:

| Layer | Check | Command |
|---|---|---|
| 1 | Is the cable or Wi-Fi connected? | `ip -br link` (Linux), `Get-NetAdapter` (Windows) |
| 2 | Do I know the gateway's MAC address? | `ip neigh` or `arp -a` |
| 3 | Do I have an IP address, and can I reach the gateway and the Internet? | `ip a`, then `ping` to `127.0.0.1`, the gateway and `8.8.8.8` |
| 3/7 | Does DNS work? | `nslookup example.com` or `dig example.com` |
| 4 | Is the port open? | `nc -vz example.com 443` or `Test-NetConnection example.com -Port 443` |
| 7 | Does the website itself respond? | `curl -I https://example.com` |

## 3. Address types

**Question:** Classify `10.4.7.2`, `172.31.255.1`, `172.32.0.1`, `169.254.7.7`, `127.8.8.8` and `224.0.0.251`.

### Solution

| Address | Type | Class |
|---|---|---|
| `10.4.7.2` | Private (`10.0.0.0/8`) | A |
| `172.31.255.1` | Private (the range `172.16.0.0/12` ends at `172.31.255.255`) | B |
| `172.32.0.1` | Public (just outside the private range) | B |
| `169.254.7.7` | Special: APIPA, self-assigned when DHCP fails | B range, but special |
| `127.8.8.8` | Special: loopback (the whole `127.0.0.0/8` block loops back) | No class |
| `224.0.0.251` | Special: multicast | D |

## 4. Subnet analysis

**Question:** For `192.168.100.200/26`, find the network address, first and last host, broadcast address and number of usable hosts.

### Solution

1. `/26` has 6 host bits, so the mask is `255.255.255.192` and the block size is 256 - 192 = **64**.
2. The blocks in the last octet are 0, 64, 128 and 192. The address `.200` is in the block that starts at **192**.

```text
Network address : 192.168.100.192
First host      : 192.168.100.193
Last host       : 192.168.100.254
Broadcast       : 192.168.100.255
Usable hosts    : 2^6 - 2 = 62
```

## 5. VLSM

**Question:** Divide `192.168.10.0/24` for Office (100 hosts), Lab (50), Servers (20) and a router link (2).

### Solution

For each segment, find the host bits `h` with `2^h - 2` at least equal to the need, and then the prefix is `32 - h`.

| Segment | Needs | Host bits | Prefix | Network | Usable range | Broadcast |
|---|---|---|---|---|---|---|
| Office | 100 | 7 (126 hosts) | /25 | 192.168.10.0/25 | .1 to .126 | .127 |
| Lab | 50 | 6 (62 hosts) | /26 | 192.168.10.128/26 | .129 to .190 | .191 |
| Servers | 20 | 5 (30 hosts) | /27 | 192.168.10.192/27 | .193 to .222 | .223 |
| Router link | 2 | 2 (2 hosts) | /30 | 192.168.10.224/30 | .225 to .226 | .227 |

Everything from `192.168.10.228` onward is still free. I allocate the **largest segment first**, because every subnet must start on a multiple of its own block size. Starting with the biggest blocks keeps the subnets aligned and leaves no gaps.

## 6. My own addresses

**Question:** In Kali, show my private and public addresses, explain why they differ, and check exercise 4 with Python.

### Solution

```bash
ip -br addr
hostname -I
curl -s ifconfig.me ; echo
python3 - <<'PY'
import ipaddress
n = ipaddress.ip_interface('192.168.100.200/26').network
print(n, n.network_address + 1, n.broadcast_address - 1, n.broadcast_address, n.num_addresses - 2)
PY
```

- `ip -br addr` and `hostname -I` show my **private** address (inside a private range).
- `curl -s ifconfig.me` shows my **public** address, which is different. It belongs to my Internet connection, and my router uses **NAT** to share it between all my devices.
- The Python script prints `192.168.100.192/26 192.168.100.193 192.168.100.254 192.168.100.255 62`, which matches my answer to exercise 4.

## 7. MAC addresses and ARP

**Question:** How does a PC at `192.168.1.10` find the MAC address of `192.168.1.1`? Show the gateway's cache entry before and after a ping.

### Solution

1. The PC checks its ARP cache. There is no entry for `192.168.1.1`.
2. It **broadcasts** an ARP request: "Who has 192.168.1.1? Tell 192.168.1.10". The destination MAC is `ff:ff:ff:ff:ff:ff`, so every device on the LAN receives it.
3. Only the owner of `192.168.1.1` replies, directly (unicast) to the PC: "192.168.1.1 is at `<its MAC>`".
4. The PC stores the answer in its cache and sends the frame.

```bash
GW=$(ip route | awk '/default/ {print $3}')
sudo ip neigh flush all
ip neigh show "$GW"
ping -c 1 "$GW"
ip neigh show "$GW"
```

- Before the ping, `ip neigh show` prints nothing for the gateway (I flushed the cache).
- After the ping, the gateway has an entry with its MAC address (`lladdr`) and a state such as `REACHABLE`, because the first packet needed ARP.
- ARP has no authentication, so an attacker can send fake replies. That is ARP spoofing, which allows a man-in-the-middle attack.

## 8. DNS

**Question:** Look up the `A` and `NS` records of `example.com`, the `MX` records of `gmail.com` and the reverse lookup of `8.8.8.8`. What does the TTL mean?

### Solution

```bash
dig example.com A +noall +answer
dig example.com NS +noall +answer
dig gmail.com MX +noall +answer
dig -x 8.8.8.8 +short
```

- The `A` query returns the IPv4 address of `example.com`. Each answer line has the name, the TTL, the class `IN`, the record type and the data.
- The `NS` query returns the authoritative name servers of the domain.
- The `MX` query returns the mail servers of `gmail.com`, each with a priority number. The lowest number is tried first.
- `dig -x 8.8.8.8 +short` is a reverse lookup (a PTR record), and it returns `dns.google.`.
- The **TTL** is the number of seconds that a resolver may cache the answer. If I repeat a query that a cache answers, the TTL counts down. The actual addresses and TTL values change over time.

## 9. DHCP and NAT

**Question:** Describe the four DHCP messages with ports and addresses, explain PAT, and write a translation table for two hosts.

### Solution

DHCP uses UDP: the server listens on port **67** and the client on port **68**.

| Step | Message | Direction | What happens |
|---|---|---|---|
| 1 | Discover | Client to broadcast (`0.0.0.0` to `255.255.255.255`) | "Is there a DHCP server? I need an address." |
| 2 | Offer | Server to client | "You can have `192.168.1.50` for 24 hours." |
| 3 | Request | Client to broadcast | "I accept `192.168.1.50` from this server." |
| 4 | Acknowledge | Server to client | "Confirmed. The lease starts now." |

The Offer and the ACK also carry the subnet mask, the gateway, the DNS server and the lease time. The client tries to renew at 50% of the lease time.

**PAT (NAT overload)** lets many private hosts share one public address. The router rewrites the source address and also the source **port**, and keeps a translation table. It uses the port numbers to tell the hosts apart:

| Inside local | Inside global | Outside |
|---|---|---|
| `192.168.1.10:51000` | `203.0.113.5:40001` | `198.51.100.20:80` |
| `192.168.1.11:51000` | `203.0.113.5:40002` | `198.51.100.20:80` |

A reply to `203.0.113.5:40001` is looked up in the table and sent to `192.168.1.10:51000`.

Attacks: a rogue DHCP server can hand out a malicious gateway or DNS server (defence: DHCP snooping), and a flood of fake Discover messages can exhaust the address pool (defence: port security).

## 10. TCP, UDP and ICMP

**Question:** Compare TCP and UDP, show the handshake, describe the replies to a SYN, and send a line over TCP with `nc`.

### Solution

| Feature | TCP | UDP |
|---|---|---|
| Connection | Handshake first | None |
| Reliability | Guaranteed and ordered, with retransmission | Best effort |
| Header | 20 bytes or more | 8 bytes |
| Typical use | Web, email, SSH, file transfer | DNS, DHCP, VoIP, streaming |

The three-way handshake, with example numbers:

```text
Client                              Server
  | -- SYN, seq=100 -------------->  |
  | <-- SYN+ACK, seq=300, ack=101 -- |
  | -- ACK, seq=101, ack=301 ------> |
```

The acknowledgment number is the next byte expected, so during the handshake it is the other side's sequence number plus 1.

| Port state | Reply to a SYN |
|---|---|
| Open | SYN+ACK |
| Closed | RST |
| Filtered | No reply, or an ICMP unreachable message |

ICMP is a Layer 3 protocol with no ports. `ping` uses ICMP echo request (type 8) and echo reply (type 0).

```bash
nc -lvnp 4444 > /tmp/nc-received.txt &
sleep 1
echo "hello over TCP" | nc -w 1 127.0.0.1 4444
sleep 1
cat /tmp/nc-received.txt
rm /tmp/nc-received.txt
```

- The listener (`-l`) waits on port 4444 and writes what it receives into a file. The second `nc` connects and sends one line.
- `cat` prints `hello over TCP`, which shows that a TCP connection carried the text.
- If `-p` is not accepted together with `-l` on my version of `nc`, I use `nc -lvn 4444` instead.

## 11. Common ports

**Question:** Give the default port, protocol and one security concern for eight services, and find the owner of a port with `ss`.

### Solution

| Service | Port | Protocol | Security concern |
|---|---|---|---|
| SSH | 22 | TCP | Brute-force attacks, so use keys |
| Telnet | 23 | TCP | Clear-text login, obsolete |
| SMTP | 25 | TCP | Open relay and user enumeration |
| DNS | 53 | UDP (and TCP for large answers and zone transfers) | Zone transfers, spoofing and tunnelling |
| HTTPS | 443 | TCP | Web application attacks, certificate problems |
| SMB | 445 | TCP | Exploited by EternalBlue and WannaCry |
| RDP | 3389 | TCP | Brute force and BlueKeep |
| MySQL | 3306 | TCP | A database should never be exposed to the Internet |

```bash
python3 -m http.server 8000 --bind 127.0.0.1 &
sleep 1
ss -tlnp | grep 8000
kill %1
```

- `ss -tlnp | grep 8000` shows a `LISTEN` socket on `127.0.0.1:8000`, owned by `python3`. The flags mean TCP (`t`), listening (`l`), numeric (`n`) and process (`p`).
- Binding to `127.0.0.1` means that only my own machine can reach the server.
- `kill %1` stops the server if it is job 1. If other background jobs are running in the same terminal, I check `jobs` first, because the server may have a higher number.

## 12. Wireshark filters

**Question:** Write the display filters, and explain capture filters vs display filters.

### Solution

| Goal | Display filter |
|---|---|
| Traffic to or from `192.168.1.10` | `ip.addr == 192.168.1.10` |
| Only DNS | `dns` |
| TCP SYN packets that are not SYN-ACK | `tcp.flags.syn == 1 && tcp.flags.ack == 0` |
| HTTP POST requests | `http.request.method == "POST"` |
| ICMP echo requests | `icmp.type == 8` |

- A **capture filter** (BPF syntax) is applied **before** recording and limits what is saved, and it cannot be changed during the capture. Example: `host 192.168.1.10` or `tcp port 80`.
- A **display filter** (Wireshark syntax) is applied **after** capture and only hides rows, so I can change it at any time. A green filter bar means that its syntax is valid.
- `ip.src == ...` matches only traffic from a host, and `ip.addr == ...` matches both directions.

## 13. A capture of my own

**Question:** Capture a TCP handshake on Windows, say which interface and filter I use, and annotate the first three packets.

### Solution

1. In Wireshark, select my Wi-Fi or Ethernet interface and start the capture.
2. Set the display filter `tcp.port == 80`.
3. In PowerShell, run `curl.exe -I http://example.com`.
4. Stop the capture.

| Packet | Flags | Relative sequence | Relative acknowledgment |
|---|---|---|---|
| 1 | SYN | 0 | none |
| 2 | SYN, ACK | 0 | 1 |
| 3 | ACK | 1 | 1 |

- Wireshark shows relative numbers by default, so the sequence numbers start at 0 and look small.
- The acknowledgment number is the other side's sequence number plus 1.
- After the handshake come the HTTP `HEAD` request and the response (for example `HTTP/1.1 200 OK`), and then `FIN, ACK` packets close the connection.
- To read the request and response as text, I right-click a packet and choose Follow, then TCP Stream.

## 14. Port scanning

**Question:** Scan my own machine with SYN, version, script and saved-output scans, and explain what `-sS` sends and what the port states mean.

### Solution

Setup:

```bash
sudo apt install nmap -y
sudo service ssh start
cd ~/practice
python3 -m http.server 8000 --bind 127.0.0.1 &
```

Scans:

```bash
sudo nmap -sS -p 22,80,443,8000 127.0.0.1
sudo nmap -sV -p 22,8000 127.0.0.1
nmap --script=http-title -p 8000 127.0.0.1
mkdir -p scans
sudo nmap -sV -p 22,8000 -oA scans/local 127.0.0.1
```

- The SYN scan lists ports 22 and 8000 as `open`, and ports 80 and 443 as `closed` (they answered with a reset). On a larger scan, Nmap hides the closed ports and prints a line such as "Not shown: 98 closed tcp ports (reset)".
- `-sV` identifies the software behind the open ports: OpenSSH on port 22 and a Python HTTP server on port 8000.
- `http-title` prints the title of the page the web server returns (`Directory listing for /`).
- `-oA scans/local` saves the results as `local.nmap`, `local.xml` and `local.gnmap`.

What `-sS` sends: a SYN packet to each port, without completing the handshake.

| State | What happened |
|---|---|
| open | The port replied with SYN-ACK, and Nmap sent a RST instead of the final ACK |
| closed | The port replied with a RST |
| filtered | No reply, or an ICMP unreachable message, because a firewall drops the probe |

Cleanup:

```bash
kill %1
sudo service ssh stop
rm -r scans
```

## 15. Commands on Linux and Windows

**Question:** Give the Linux and Windows command for eight tasks.

### Solution

| Task | Linux | Windows |
|---|---|---|
| Show my IP address | `ip a` | `ipconfig /all` |
| Show the routing table | `ip route` | `route print` |
| Test connectivity | `ping -c 4 host` | `ping host` |
| Show the ARP cache | `ip neigh` or `arp -a` | `arp -a` |
| Look up a DNS name | `dig host` or `nslookup host` | `nslookup host` or `Resolve-DnsName host` |
| Trace the route | `traceroute host` | `tracert host` |
| Show listening ports | `ss -tuln` | `netstat -ano` or `Get-NetTCPConnection -State Listen` |
| Renew a DHCP lease | `sudo dhclient -r` then `sudo dhclient` | `ipconfig /release` then `ipconfig /renew` |

In WSL, the network follows Windows, so I renew the lease on the Windows side.