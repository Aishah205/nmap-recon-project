# Findings — Nmap Network Reconnaissance & Enumeration Project

## Environment
- **Attacker machine:** Kali Linux (VirtualBox VM)
- **Target machine:** Ubuntu Server (VirtualBox VM), IP `192.168.100.86`
- Both VMs isolated on a private lab network; no external/production systems were scanned.

## 1. Host Discovery
```
sudo nmap -sn 192.168.100.86
```
Confirmed the target was live and reachable on the lab network.

## 2. Full Port Scan
```
sudo nmap -p- 192.168.100.86
```
Scanned all 65,535 TCP ports. Result: 3 open ports out of 65,535 — a minimal, well-contained attack surface.

## 3. Service & Version Detection
```
sudo nmap -sV -sC 192.168.100.86
```

| Port | Service | Version (banner) |
|---|---|---|
| 21/tcp | FTP | vsftpd 3.0.5 |
| 22/tcp | SSH | OpenSSH 9.6p1 (Ubuntu) |
| 80/tcp | HTTP | Apache 2.4.58 (Ubuntu, default page) |

## 4. OS Fingerprinting
```
sudo nmap -O 192.168.100.86
```
Confirmed target as a Linux-based system, consistent with the Ubuntu Server VM configured for this lab.

## 5. Vulnerability Scanning
```
sudo nmap -sV --script vuln 192.168.100.86
```
The `vulners` NSE script flagged a large number of CVEs against both OpenSSH and Apache, including high-severity entries such as CVE-2024-6387 ("regreSSHion") for SSH and several 9.8-rated CVEs for Apache.

**However, these results needed verification rather than face-value acceptance.** The vulners script matches CVEs purely against the version string reported in the service banner. Ubuntu backports security patches into its packages without always updating that banner string, which can make an already-patched system look vulnerable to automated scanners.

### Verifying real patch level
Direct package inspection (bypassing the banner) revealed a mismatch:

| Service | Nmap banner reported | Actual installed package |
|---|---|---|
| OpenSSH | 9.6p1 | 10.2p1-3 |
| Apache | 2.4.58 | 2.4.66-2 |

```
dpkg -l | grep openssh-server
dpkg -l | grep apache2
apt-cache policy openssh-server
apt-cache policy apache2
```

This confirms the actual installed packages are meaningfully newer than what the service banners advertised — meaning most of the CVEs flagged by the automated `vulners` script are **likely false positives** on this system, since they target version ranges the real installed packages have already moved past.

## 6. Timing & Evasion Techniques
```
sudo nmap -T2 -f -D RND:5 192.168.100.86
```
Demonstrated:
- `-T2` — slower, less detectable scan timing
- `-f` — packet fragmentation to evade simple packet-filtering firewalls
- `-D RND:5` — decoy scanning, obscuring the true scan source among 5 random fake IPs

## Key Takeaway
The most valuable finding from this project wasn't a specific CVE — it was the gap between **automated tool output** and **verified ground truth**. Nmap's `vulners` script is a fast way to surface *candidate* risks, but Ubuntu's patch-backporting practice means banner-based version matching alone isn't reliable evidence of real exposure. Confirming findings via `dpkg`/`apt-cache policy` before treating a scan result as a genuine vulnerability is a core practice in real vulnerability assessment work — separating raw scanner noise from actionable risk.
