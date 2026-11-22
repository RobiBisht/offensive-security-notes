# Wireshark Captures Explained

[Back to networking index](README.md) | [Wireshark notes](04-wireshark.md)

These are packet captures I made myself in my own lab, with a screenshot and an explanation for each one. The notes in [Wireshark notes](04-wireshark.md) explain the tool, and this file shows what real traffic looks like.

## My lab

| Item | What I used |
|---|---|
| Capture machine | <!-- for example: Kali VM --> |
| Other machines | <!-- for example: Ubuntu VM, Windows VM, Metasploitable 2 --> |
| Network | Host-only network, isolated from the Internet |
| Wireshark version | <!-- Help > About Wireshark --> |
| Interface captured on | <!-- for example: eth0 --> |

All captures are from my own lab network. Screenshots are cropped or blurred where they show real IP addresses, MAC addresses or passwords.

## Contents

1. [ICMP ping](#capture-1-icmp-ping)
2. [TCP three-way handshake](#capture-2-tcp-three-way-handshake)
3. [DNS lookup](#capture-3-dns-lookup)
4. [ARP request and reply](#capture-4-arp-request-and-reply)
5. [DHCP (DORA)](#capture-5-dhcp-dora)
6. [Clear-text login over HTTP](#capture-6-clear-text-login-over-http)
7. [Nmap SYN scan](#capture-7-nmap-syn-scan)

---

## Capture 1: ICMP ping

**Goal:** see the Ethernet, IP and ICMP layers of a simple request and reply.

**Setup**

- Command: `ping -c 4 <target IP>`
- Display filter: `icmp`

**What I should see**

- 4 Echo requests (ICMP type 8) and 4 Echo replies (type 0)
- In the packet details: Ethernet II (source and destination MAC), IPv4 (protocol ICMP, a TTL value), then ICMP (type, code, sequence number)

### What I saw

<!-- Add a screenshot: ![ICMP capture](images/01-icmp-ping.png) -->
<!-- Add a short packet table: No., Source, Destination, Info -->

### Layer breakdown

<!-- Write what each layer showed: MAC addresses, IP addresses and TTL, ICMP type and code -->

### My explanation

<!-- Explain in my own words how the request and reply travel, and what the TTL tells me -->

### What I learned

<!-- One or two lines, for example about TTL values or sequence numbers -->

---

## Capture 2: TCP three-way handshake

**Goal:** see a TCP connection start, send data and close.

**Setup**

- Command: `curl http://<lab web server IP>` (for example Metasploitable 2)
- Display filter: `tcp.port == 80`

**What I should see**

- The first three packets: `[SYN]`, `[SYN, ACK]`, `[ACK]`
- Then an HTTP GET request and a response, followed by `[FIN, ACK]` packets that close the connection
- Wireshark shows **relative** sequence numbers by default, so they start at 0

### What I saw

<!-- Add a screenshot: ![TCP handshake](images/02-tcp-handshake.png) -->
<!-- Add a table of the first three packets with their flags and sequence/ack numbers -->

### Following the stream

<!-- Right-click a packet > Follow > TCP Stream. Describe what the HTTP request and response looked like -->

### My explanation

<!-- Explain what each of the three packets means and why the ack number is the sequence number plus 1 -->

### What I learned

<!-- One or two lines -->

---

## Capture 3: DNS lookup

**Goal:** see how a name turns into an IP address.

**Setup**

- Command: `nslookup example.com` (or `dig example.com`)
- Display filter: `dns`

**What I should see**

- A query over UDP, destination port 53, type A
- A response with an Answers section that contains the IP address and a TTL

### What I saw

<!-- Add a screenshot: ![DNS capture](images/03-dns-lookup.png) -->

### Details I noted

<!-- The query name, the record type, the answer, the TTL and the DNS server that replied -->

### My explanation

<!-- Describe the steps of the lookup in my own words -->

### What I learned

<!-- One or two lines -->

---

## Capture 4: ARP request and reply

**Goal:** see how a device finds a MAC address on the local network.

**Setup**

- Commands: clear the ARP cache (`arp -d <gateway IP>` or `sudo ip neigh flush all`), then `ping <gateway IP>`
- Display filter: `arp`

**What I should see**

- One broadcast request: "Who has x.x.x.x? Tell y.y.y.y" sent to `ff:ff:ff:ff:ff:ff`
- One unicast reply: "x.x.x.x is at <MAC address>"

### What I saw

<!-- Add a screenshot: ![ARP capture](images/04-arp.png) (blur the MAC addresses) -->

### My explanation

<!-- Explain why the request is a broadcast and the reply is not, and what ARP spoofing would change -->

### What I learned

<!-- One or two lines -->

---

## Capture 5: DHCP (DORA)

**Goal:** see the four messages a device uses to get an IP address.

**Setup**

- Commands: `ipconfig /release` then `ipconfig /renew` on Windows, or `sudo dhclient -r` then `sudo dhclient` on Linux
- Display filter: `udp.port == 67 || udp.port == 68`

**What I should see**

- Discover, Offer, Request and Acknowledge, in that order
- Discover sent from `0.0.0.0` to `255.255.255.255`
- The offered address, the gateway, the DNS server and the lease time inside the Offer and ACK

### What I saw

<!-- Add a screenshot: ![DHCP capture](images/05-dhcp-dora.png) -->

### My explanation

<!-- Describe each of the four packets and what information they carry -->

### What I learned

<!-- One or two lines -->

---

## Capture 6: Clear-text login over HTTP

**Goal:** see why unencrypted protocols are dangerous.

**Setup**

- A lab target only (for example DVWA or Metasploitable 2) and a throwaway test account
- Display filter: `http.request.method == "POST"`

**What I should see**

- In the POST packet, under "HTML Form URL Encoded", the username and password in plain text
- Over HTTPS, only encrypted "TLS Application Data", with no readable password

### What I saw

<!-- Add a screenshot with the password blurred: ![HTTP POST capture](images/06-http-post.png) -->

### My explanation

<!-- Explain what an attacker on the same network could see, and how HTTPS prevents it -->

### What I learned

<!-- One or two lines -->

---

## Capture 7: Nmap SYN scan

**Goal:** see what a port scan looks like from the target's side.

**Setup**

- Command (against my own lab machine): `sudo nmap -sS <target IP>`
- Display filter: `tcp.flags.syn == 1 && tcp.flags.ack == 0`
- Then: Statistics > Conversations > TCP

**What I should see**

- Many SYN packets from one source to many different ports in a very short time
- `[SYN, ACK]` replies from open ports, and `[RST, ACK]` replies from closed ports
- Nmap sending `[RST]` instead of a final ACK, so the handshake never completes

### What I saw

<!-- Add a screenshot: ![Nmap SYN scan](images/07-nmap-syn-scan.png) -->

### My explanation

<!-- Explain how I can tell this is a scan and not normal traffic, and what a defender would look for -->

### What I learned

<!-- One or two lines -->

---

## Key takeaways

<!-- 3 to 5 lines in my own words after finishing all the captures -->