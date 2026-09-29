# Home Network Recon with Nmap

A hands-on walkthrough practicing the standard Nmap recon workflow — discovery, port scanning, service/version detection, OS fingerprinting, and vulnerability scripting — against a home network router, with the network owner's permission.

**Target:** Home router (Cisco Meraki), IP redacted as `192.168.X.1`
**Tools:** Kali Linux (VirtualBox, bridged networking), Nmap 7.99

---

## 1. Host Discovery

```bash
nmap -sn 192.168.X.0/24
```
Ping sweep across the subnet to find live devices — no ports scanned yet. Found 14 active hosts on the network (routers, access points, personal devices).

**Takeaway:** Discovery comes first so later scans target real, live hosts instead of wasting time on dead addresses.

---

## 2. Port Scan

```bash
nmap 192.168.X.1
```
Scanned the top 1000 ports on the router.

**Result:**
```
PORT     STATE  SERVICE
80/tcp   open   http
81/tcp   closed hosts2-ns
179/tcp  closed bgp
8090/tcp open   opsmessaging
```

**Takeaway:** Two open web services on different ports — one standard (80), one non-standard (8090). Scanning beyond just the default port revealed a second, less obvious interface.

---

## 3. Service & Version Detection

```bash
nmap -sV 192.168.X.1
```
Identified port 80 as the router's admin login page and port 8090 as a separate local management/status interface (confirmed via HTTP response content).

**Takeaway:** Version detection turns "a port is open" into "here's what's actually running," which is what you need before assessing risk.

---

## 4. OS Detection

```bash
nmap -O 192.168.X.1
```
**Result:** No confident match — low-confidence guesses spanning Linux kernel versions and unrelated device types (Android, streaming devices).

**Takeaway:** OS fingerprinting is unreliable against embedded/appliance hardware. These devices run heavily customized, stripped-down OS builds that don't match standard fingerprint databases, and many unrelated device types share the same underlying kernel family — so guesses get noisy. This is expected behavior, not a scan failure.

---

## 5. Vulnerability Scanning

```bash
nmap --script vuln 192.168.X.1
```

**Findings:**
- No file upload, DOM-based XSS, or stored XSS issues
- **Possible CSRF** flagged on two admin config forms — likely low real-world risk, since exploitation requires an active authenticated session
- **Slowloris DoS (CVE-2007-6750)** flagged — a 15+ year old denial-of-service technique, most likely a false positive on actively maintained firmware. Even if valid, this is a temporary-availability risk, not a path to compromise

**Takeaway:** Vulnerability scanners report *possibilities*, not confirmed exploits. Every flag needs manual judgment — checking the CVE's age and relevance before treating it as an actual finding. This is the core skill separating a scan from an assessment.

---

## 6. Full Aggressive Scan

```bash
sudo nmap -A 192.168.X.1
```
Consolidated scan (version + OS detection + default scripts + traceroute). Confirmed all prior findings; traceroute showed a single hop, consistent with a direct local connection.

---

## Summary

| Stage | Purpose | Result |
|---|---|---|
| Discovery | Find live hosts | 14 devices found |
| Port scan | Find open services | 2 open (80, 8090) |
| Version detection | ID software | Router admin panel + local status page |
| OS detection | ID operating system | Inconclusive (expected for embedded HW) |
| Vuln scan | Check known CVEs | 2 low-confidence flags, no confirmed exploit |
| Aggressive scan | Consolidate everything | Findings confirmed, no new issues |

**Conclusion:** No exploitable vulnerabilities identified. The router showed properly enforced access control and no confirmed real-world weaknesses — a realistic outcome when scanning actively maintained, cloud-managed hardware.

---

*Recon performed on a personally owned/authorized home network. IP addresses and hardware-identifying details redacted.*
