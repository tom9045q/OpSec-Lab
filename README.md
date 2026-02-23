# OpSec-Lab

Kali Linux anonymous attack environment configured for traffic anonymization using MAC spoofing, Tor, and Proxychains. Built for cybersecurity portfolio demonstration.

---

## Objectives

- Configure a clean Kali Linux VM with snapshot discipline
- Spoof MAC address to prevent Layer 2 identification
- Route all tool traffic through the Tor network
- Force non-proxy-aware tools through Tor via Proxychains
- Verify anonymization and document limitations

---

## Tools Used

| Tool | Purpose |
|------|---------|
| Kali Linux | Attack platform |
| VirtualBox/VMware | VM isolation |
| `ip link` | MAC address spoofing |
| Tor | Layer 3 traffic anonymization via onion routing |
| Proxychains4 | Force tools through Tor SOCKS5 proxy |
| `curl` / `nmap` | Verification and testing |

---

## Architecture
```
[Kali VM] → [Spoofed MAC] → [Proxychains4] → [Tor SOCKS5 :9050] → [Tor Network (3 hops)] → [Target]
```

Target only sees Tor exit node IP. Real IP and MAC never exposed beyond local segment.

---

## Setup

### 1. VM Snapshot
Before any configuration, take a clean snapshot to preserve base state.

![Clean Snapshot](screenshots/01-clean-snapshot.png)

### 2. MAC Address Spoofing
```bash
ip link show                                          # identify interface
sudo ip link set eth0 down
sudo ip link set eth0 address 00:11:22:33:44:55
sudo ip link set eth0 up
ip link show eth0                                     # verify change
```

![MAC Before](screenshots/02-mac-before.png)
![MAC After](screenshots/03-mac-after.png)

### 3. Tor Installation
```bash
sudo apt update && sudo apt install tor -y
sudo service tor start
sudo service tor status
```

![Tor Running](screenshots/04-tor-running.png)

### 4. Proxychains Configuration
```bash
sudo apt install proxychains4 -y
sudo nano /etc/proxychains4.conf
```

Changes made:
- `dynamic_chain` uncommented
- `strict_chain` commented out
- `[ProxyList]` set to `socks5 127.0.0.1 9050`

Config file: [proxychains4.conf](configs/proxychains4.conf)

### 5. Verification
```bash
proxychains4 curl https://ifconfig.me    # confirm Tor exit node IP
proxychains4 nmap -sT -Pn <target>       # confirm tool routing works
```

![IP Verification](screenshots/05-ip-verification.png)
![Proxychains Test](screenshots/06-proxychains-test.png)

---

## Limitations

- Tor exit node IPs are publicly listed — defenders can detect Tor usage even without identifying the source
- MAC spoofing only effective at Layer 2; irrelevant once traffic crosses a router
- TCP connect scans (`-sT`) required through Proxychains — SYN scans not supported
- Tor does not protect against application-layer identification (cookies, account logins, browser fingerprinting)
- Timing analysis by sophisticated adversaries can potentially de-anonymize Tor users

---

## Lessons Learned

See [docs/lessons-learned.md](docs/lessons-learned.md) — completed after lab execution.

---

## References

- [Tor Project](https://www.torproject.org)
- [Proxychains-ng](https://github.com/haad/proxychains)
- [Kali Linux Documentation](https://www.kali.org/docs/)
