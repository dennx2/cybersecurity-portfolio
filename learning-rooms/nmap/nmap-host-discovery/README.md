# Nmap Host Discovery Guide

Host Discovery is crucial because trying to port-scan offline systems will only waste time and create unnecessary noise on the network.

**Check if the host is alive**

- Approaches:
  - ARP scan: This scan uses ARP requests to discover live hosts
  - ICMP scan: This scan uses ICMP requests to identify live hosts
  - TCP/UDP ping scan: This scan sends packets to TCP ports and UDP ports to determine live hosts.
- Nmap sends a small set of probe packets to the target.
  If the host responds, Nmap considers it "up."
- Remember to add `-sn` if you are only interested in host discovery without port-scanning.
  Omitting `-sn` will let Nmap default to port-scanning the live hosts.

**Resolve hostnames (optional)**

- By default, Nmap will try a reverse DNS lookup to get the hostname of the live IPs.

- `-n` disables DNS lookups.

- `-R` forces reverse DNS lookup even if `-n` is used globally.
- `--dns-servers DNS_SERVER` — use a specific DNS server

---

## 1. Scan Types

- `sudo nmap -PR -sn MACHINE_IP/24` — ARP Scan
- `sudo nmap -PE -sn MACHINE_IP/24` — ICMP Echo Scan
- `sudo nmap -PP -sn MACHINE_IP/24` — ICMP Timestamp Scan
- `sudo nmap -PM -sn MACHINE_IP/24` — ICMP Address Mask Scan
- `sudo nmap -PS22,80,443 -sn MACHINE_IP/30` — TCP SYN Ping Scan
- `sudo nmap -PA22,80,443 -sn MACHINE_IP/30` — TCP ACK Ping Scan — expects RST reply
- `sudo nmap -PU53,161,162 -sn MACHINE_IP/30` — UDP Ping Scan — expects ICMP port unreachable

**Discover hosts only (no port scan)**

- `sudo nmap -sn 10.10.0.0/24`
- When you don’t specify a host discovery method, Nmap chooses the "best method automatically based on your privileges and the type of network.
  - Privileged (sudo/root) on a local Ethernet network: Nmap uses ARP requests, because it’s the most reliable way to see which hosts are up.
  - Non-privileged or remote networks: Nmap will try ICMP Echo, TCP SYN to common ports (like 443, 80), or TCP ACK depending on what’s likely to succeed.

**Count the number of IPs without actually sending any probes.**

- How many IP addresses will Nmap scan if provide the range 10.10.0-255.101-125?
- `nmap -sL -n 10.10.0-255.101-125` → 6400
- `-sL` → List scan, only lists targets, does not send packets
- `-n` → Skip DNS resolution, speeds up the listing

---

## 2. Masscan (Alternative to Nmap)

- Very fast scanner, similar to Nmap but much more aggressive.
- Always requires `sudo` (raw packet sending).
- Best for scanning huge ranges quickly, but noisy (easy to detect).

Examples:

```bash
sudo masscan TARGETS -p80
sudo masscan 10.10.68.0/24 -p22-25
```

---

## 3. Port Scanning vs Host Discovery

- **Host discovery** → "Is the host alive?"
- **Port scanning** → "Which ports are open and reachable?"

Example: `-PS` and `-sS` both may use TCP SYN, but they serve different goals:

| Option | Purpose        | What It Does                                                                    |
| ------ | -------------- | ------------------------------------------------------------------------------- |
| `-PS`  | Host Discovery | Sends SYN to one/few ports → if host replies at all, it’s alive.                |
| `-sS`  | Port Scan      | Sends SYN to many ports → determines which ports are open, closed, or filtered. |

---

## 4. Decision Guide

- Local subnet? → Use ARP (`-PR`).
- ICMP allowed? → Use `-PE` or `-PP`.
- ICMP blocked? → Use TCP SYN (`-PS`) or TCP ACK (`-PA`).
- Last resort → Use UDP (`-PU`) (slow).
- Need speed across large ranges? → Use Masscan.
