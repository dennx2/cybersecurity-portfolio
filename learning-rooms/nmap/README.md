# Nmap Scanning Guide

Nmap is an open-source tool for network discovery and security testing. It supports many scan types, scripts, and evasion methods—useful for penetration testing, auditing, and administration.

This guide provides an overview of Nmap scanning techniques, including TCP scans, UDP scans, Nmap Scripting Engine (NSE) scripts, and firewall evasion techniques.

[Nmap guide](https://nmap.org/book/man-port-scanning-techniques.html)

---

## Table of Contents

- [TCP Scans](#tcp-scans)
  - [SYN Scan (`-sS`)](#syn-scan--ss)
  - [Connect Scan (`-sT`)](#connect-scan--st)
  - [Stealth & Aggressive Options](#stealth--aggressive-options)
- [UDP Scans (`-sU`)](#udp-scans--su)
- [ICMP Network Scanning (Ping Sweep) (`-sn`)](#icmp-network-scanning-ping-sweep--sn)
- [Nmap Scripting Engine (NSE) Scans (`--script`)](#nmap-scripting-engine-nse-scans--script)
- [Firewall Evasion (`-Pn`)](#firewall-evasion--pn)
- [Notes Summary](#notes-summary)
- [Command Examples](#command-examples)
- [Hands-on Practice](#hands-on-practice)

---

## TCP Scans

TCP scans are used to determine which TCP ports on a target are open, closed, or filtered.

### SYN Scan (`-sS`)

- Also called: Half-open scans (doesn’t complete handshake, stealthier)
- Finds open TCP ports
- Command: `nmap -sS <target-ip>`

How it works (open port):

- Nmap → sends SYN
- Target → replies with SYN/ACK
- Nmap → sends RST (instead of completing handshake)

Advantages:

- Can bypass some older IDS (looks stealthier than full handshake)
- Often not logged by applications (connection never fully established)
- Faster than TCP Connect scan (no full handshake per port)

Disadvantages:

- Needs sudo/root (creates raw packets)
- Can crash unstable services (rare but possible)

Defaults:

- With sudo → Nmap uses SYN scan by default
- Without sudo → falls back to TCP Connect scan

Port behavior (same as TCP Connect):

- Closed port → server sends RST
- Filtered port → packet dropped or fake RST from firewall
- Open port → SYN/ACK (then reset by Nmap)

SYN scans are fast, efficient, and relatively stealthy — that’s why they’re the default scan type in Nmap (when run as root).

### Connect Scan (`-sT`)

- Full TCP handshake (less stealthy)
- Use if SYN scan is blocked or needs root privileges
- Command: `nmap -sT <target-ip>`

Uses the full TCP 3-way handshake:

- Attacker → SYN
- Target → SYN/ACK
- Attacker → ACK (handshake complete)

How Nmap decides port state:

- Open port → SYN/ACK received → handshake completed
- Closed port → target sends RST (reset)
- Filtered port → no response (packet dropped by firewall)
- OR firewall may fake a RST (harder to detect filtering)

Pros:

- Works without sudo/root (default if not root)
- Reliable and accurate

Cons:

- Slower than SYN scan (full handshake with each port)
- Easier to detect/log (connection fully established)

TCP Connect scan is simple, accurate, and doesn’t need special privileges, but it’s slower and more obvious compared to a SYN scan.

### Stealth & Aggressive Options

Purpose: Extra stealthy scans used to bypass firewalls/IDS. Try to sneak past basic filtering (firewalls that block SYN packets).

- `-sF` : FIN scan (sneaky)
- `-sX` : Xmas scan (FIN+PSH+URG)
- `-sN` : Null scan (no flags)
- `-A` : Aggressive: OS detection + version + scripts + traceroute

How they work:

- NULL scan (`-sN`): Sends TCP packet with no flags.
- FIN scan (`-sF`): Sends TCP packet with FIN flag (normally closes connections).
- Xmas scan (`-sX`): Sends TCP packet with FIN, PSH, URG flags → looks like a blinking Xmas tree in Wireshark.

Expected responses:

- Closed port: Target replies with RST.
- Open port: No response.
- Filtered port: No response OR ICMP unreachable.

Limitations:

- Results often show `open|filtered` (cannot always tell the difference).
- Windows & Cisco devices: Reply with RST to everything → all look "closed."
- Modern IDS: Usually detects these scans → less effective today.

---

## UDP Scans (`-sU`)

UDP scans check for open UDP ports.

- Commands:
  - UDP only: `nmap -sU <target-ip>`
  - TCP + UDP together: `nmap -sS -sU <target-ip>  `

Key points:

- UDP is stateless → no handshake, just sends packets.
- Open port & Filtered UDP port (blocked by firewall): Usually no response → Nmap by default will retransmit the probe multiple times (often 1–3 retries, depending on timing options) before marking → retries causes slower scans → Nmap marks as open|filtered.
- Closed port: Responds with ICMP "port unreachable" → Nmap marks as closed.

Challenges:

- Hard to distinguish open vs filtered
- Scanning top 1000 ports can take ~20 minutes.
- Can speed up scan with `--top-ports <number>` (e.g., top 20 UDP ports): `nmap -sU --top-ports 20 <target>`

Requests:

- Nmap usually sends empty UDP packets.
- For well-known services, it may send protocol-specific payloads to trigger a response.

Note:

- Responses:
  - ICMP "Port Unreachable" → Closed
  - No response → Open or Filtered
- UDP scans are slower and less reliable than TCP scans.
- Use targeted ports or top ports to save time.

---

## ICMP Network Scanning (Ping Sweep) (`-sn`)

- Find which IPs are alive on a network.

- Command:

  - Range: `nmap -sn 192.168.0.1-254`

  - CIDR: `nmap -sn 192.168.0.0/24`

- `-sn` option:

  - Tells Nmap not to scan ports.

  - Uses ICMP echo (ping) or ARP requests (on local networks, requires sudo/root) to discover which hosts are alive, then stops without scanning ports.

  - If ping/ARP doesn’t work, Nmap also sends:

    - TCP SYN → port 443

    - TCP ACK (or SYN if not root) → port 80

- Ping sweeps quickly map active hosts, but results may not always be accurate (some hosts block ICMP).

---

## Nmap Scripting Engine (NSE) Scans (`--script`)

- Automates vulnerability and service checks
- Extends Nmap with scripts written in Lua
- Command: `nmap --script <script-name> <target-ip>`

Script Categories

- default → basic discovery
- safe → non-intrusive
- intrusive → may affect target
- vuln → vulnerability checks
- exploit → exploit vulnerabilities
- auth → bypass authentication (e.g., anonymous FTP)
- brute → brute force credentials
- discovery → get extra info about services

Running Scripts

- Run all scripts in a category:

  - `nmap --script=vuln <target>`
  - `nmap --script=safe <target>`
  - Example: `nmap -sV --script vuln <target-ip>`

- Run a specific script:

  - `nmap --script=<script-name> <target>`

- Run multiple scripts:
  - `nmap --script=script1,script2 <target>`

Scripts with arguments:

```bash
nmap -p 80 --script http-put \
--script-args http-put.url='/dav/shell.php',http-put.file='./shell.php'
```

- Use periods to connect script and argument (`<script>.<arg>`)

- Use commas to separate multiple arguments

Finding Scripts

- Local directory: `/usr/share/nmap/scripts`

Search scripts:

- `grep "ftp" /usr/share/nmap/scripts/script.db`
- `ls -l /usr/share/nmap/scripts/*ftp*`

Search by category:

- `grep "safe" /usr/share/nmap/scripts/script.db`

Installing/Updating Scripts

- Update Nmap scripts:

  - `sudo apt update && sudo apt install nmap`

- Manually download a script:

  - Download:

    ```bash
    sudo wget -O /usr/share/nmap/scripts/<script-name>.nse \
    https://svn.nmap.org/nmap/scripts/<script-name>.nse
    ```

  - Use the script:

    ```bash
    nmap --script=<script-name> <target>
    ```

- Update script database:
  - `nmap --script-updatedb`

Key Takeaways:

- NSE is powerful for scanning, enumeration, and exploits.
- Scripts can be run individually, in categories, or with arguments.
- Scripts are stored locally but can also be downloaded or created in Lua.

---

## Firewall Evasion (`-Pn`)

- Skips host discovery (ping) → treats host as online
- Useful if ping blocked by firewall
- Slower if host is actually offline
- Command: `nmap -Pn <target-ip>`

Other evasion techniques:

- `-f` → fragment packets (harder to detect)
- `--mtu <number>` → set custom packet size (alternative to `-f`)
- `--scan-delay <time>` → pause between packets (evade time-based IDS/firewalls or unstable networks)
- `--badsum` → send invalid checksums (may trigger firewall/IDS responses)

Note:

- On a local network, Nmap can also use ARP to detect hosts instead of ICMP.

---

## Notes Summary

- TCP scans → faster, reliable; use SYN (`-sS`) or Connect (`-sT`)
- UDP scans → slower, less reliable; use `-sU`
- NSE → automates checks, use `--script`
- Firewall evasion → `-Pn`, fragment packets, change source port, add delays

---

## Command Examples

- SYN scan + OS detection: `nmap -sS -O <target-ip>`
- UDP scan verbose: `nmap -sU -v <target-ip>`
- Run default NSE scripts: `nmap -sV --script default <target-ip>`
- Stealth scan skipping host discovery: `nmap -sS -Pn -f <target-ip>`

---

## Hands-on Practice

**1. Xmas Scan on First 999 Ports**

Question: How many ports are open or filtered?

- Answer: 999
- Lesson:
  - Xmas scan sends FIN, PSH, and URG flags. If no response is received, Nmap considers the port `open|filtered`.
  - The scan shows `No Response` for many ports because firewalls drop packets instead of replying. This demonstrates stealthy scanning and port filtering behavior.

![xmas scan](./screenshots/nmap%20pn%20xmas%20scan.png)

**2. TCP SYN Scan on First 5000 Ports**

Question: How many ports are open?

- Answer: 5
- Lesson:
  - SYN scan (`-sS`) is faster and more reliable for finding open ports. It can detect ports even if ping is blocked.
  - This teaches how to find actual open services, not just filtered ones.

![tcp scan](./screenshots/nmap%20pn%20tcp%20scan.png)

**3. TCP Connect Scan using FTP Script**

Question: Can Nmap login to FTP on port 21?

- Answer: Yes
- Lesson:
  - NSE scripts check for weak setups like anonymous FTP login.
  - Must also check what service is running and how it’s configured.
  - Service enumeration = finding weaknesses after discovering open ports.

![ftp script scan](./screenshots/nmap%20pn%20ftp%20script%20scan.png)
