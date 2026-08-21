# Nmap Network Reconnaissance & Enumeration Project

A hands-on lab project demonstrating the network reconnaissance and enumeration phase of a penetration test — using Nmap to discover hosts, identify services, fingerprint the OS, scan for vulnerabilities, and test evasion techniques against a controlled target.

## ⚠️ Scope & Ethics
All scans in this repository were run against a private, isolated lab VM (Ubuntu Server) that I own and configured myself. No external, third-party, or production systems were scanned. This project is for educational and skill-building purposes only.

## Environment
| Role | System |
|---|---|
| Attacker | Kali Linux (VirtualBox VM) |
| Target | Ubuntu Server (VirtualBox VM) |

Both VMs were placed on an isolated internal/lab network for the scanning phase, with a separate NAT connection on the Kali VM for internet access (package installs, GitHub).

## What This Project Covers
1. **Host discovery** — confirming the target is live on the network
2. **Full port scan** — scanning all 65,535 TCP ports
3. **Service & version detection** — identifying exact software running on each open port
4. **OS fingerprinting** — identifying the target's operating system
5. **Vulnerability scanning** — using Nmap's `vulners` NSE script to flag known CVEs
6. **Timing & evasion techniques** — slow timing, packet fragmentation, and decoy scanning
7. **Verification of scanner output** — cross-checking banner-based CVE matches against actual installed package versions (a key finding of this project — see `findings.md`)

## Repository Structure
```
nmap-recon-project/
├── README.md
├── findings.md          # Full write-up of results and analysis
└── scans/
    ├── 01_host_discovery.*
    ├── 02_full_port_scan.*
    ├── 03_service_version.*
    ├── 04_os_fingerprint.*
    ├── 05_vuln_scripts.*
    └── 06_evasion_demo.*
```
Each scan set includes `.nmap` (human-readable), `.xml`, and `.gnmap` output formats.

## Key Result
Out of 65,535 ports scanned, only 3 were open: FTP (21), SSH (22), and HTTP (80). Automated vulnerability scanning flagged numerous CVEs against these services based on banner version strings — but direct package inspection (`dpkg`, `apt-cache policy`) revealed the actual installed packages were newer than the banners suggested, meaning most flagged CVEs were likely false positives. Full analysis in [`findings.md`](./findings.md).

## Tools Used
- Nmap 7.95 (including NSE scripts: `vuln`, `vulners`, default `-sC` scripts)
- Kali Linux
- Ubuntu Server (vsftpd, OpenSSH, Apache)

## Skills Demonstrated
- Network scanning and enumeration methodology
- Service/version fingerprinting
- Vulnerability assessment using automated tooling
- Critical evaluation of scanner output vs. verified ground truth
- Basic firewall/IDS evasion concepts
- Git/GitHub workflow for documenting security work
