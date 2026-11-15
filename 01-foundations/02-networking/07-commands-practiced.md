# Commands I Practiced

[Back to networking index](README.md)

A quick-reference table of the networking commands I ran in my Kali and Ubuntu VMs.

| Command | What it does | Example I ran |
|---|---|---|
| `ip a` (Windows: `ipconfig /all`) | Shows my IP addresses, MAC address and interfaces | `ip a` |
| `ip route` | Shows the routing table and my default gateway | `ip route` |
| `ping` | Tests connectivity to a host using ICMP | `ping -c 4 8.8.8.8` (Windows: `ping -n 4 8.8.8.8`) |
| `arp -a` (Linux also: `ip neigh`) | Shows the ARP table, which maps IPs to MACs on my LAN | `arp -a` |
| `nslookup` | Looks up DNS records (basic) | `nslookup example.com` |
| `dig` | Looks up DNS records in detail | `dig example.com A +short` |
| `traceroute` (Windows: `tracert`) | Shows the path packets take to a host, hop by hop | `traceroute example.com` |
| `ss -tuln` (Windows: `netstat -ano`) | Shows open listening TCP and UDP ports | `ss -tuln` |
| `curl` | Fetches a web page or just its headers | `curl -I http://example.com` |
| `curl ifconfig.me` | Shows my public IP, which differs from my private IP because of NAT | `curl ifconfig.me` |
| `nc` (netcat) | Tests whether a port is open, or opens a TCP or UDP connection | `nc -vz scanme.nmap.org 80` |
| `nmap -sn` | Ping sweep that finds live hosts on my own network | `sudo nmap -sn 192.168.1.0/24` |
| `ipconfig /release` and `/renew` | Windows: gives up my DHCP lease and requests a new one | `ipconfig /renew` |