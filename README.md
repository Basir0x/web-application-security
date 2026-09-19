# 🎯 Bug Bounty Notes & Toolkit

A structured collection of **bug bounty tools, commands, reconnaissance techniques, and workflows** for authorized security testing.

> ⚠️ **Disclaimer:** Only use these tools against systems you own or are explicitly authorized to test. Always follow the target's bug bounty scope, rules, and rate limits.


> Red Team > ready > lest GO



---

## 📋 Table of Contents

* [Directory Bruteforcing](#1-directory-bruteforcing)
* [Subdomain Enumeration](#2-subdomain-enumeration)
* [Screenshotting](#3-screenshotting)
* [Parameter Discovery](#4-parameter-discovery)
* [Vulnerability Scanning](#5-vulnerability-scanning)
* [Recon Workflow Pipeline](#6-recon-workflow-pipeline)
* [Network Scanning](#7-network-scanning)
* [OSINT & Threat Intelligence](#8-osint--threat-intelligence)
* [Google Dorking](#9-google-dorking)
* [Quick Workflow Cheat Sheet](#-quick-workflow-cheat-sheet)
* [Responsible Testing](#-responsible-testing)

---

# 1. 📁 Directory Bruteforcing

Directory and file discovery can help identify publicly accessible endpoints, directories, and resources.

## 🔧 Dirsearch

### Installation

```bash
git clone https://github.com/maurosoria/dirsearch.git
cd dirsearch
pip3 install -r requirements.txt
```

### Usage

```bash
python3 dirsearch.py -u <URL> -e <EXTENSIONS>
```

### Example

```bash
python3 dirsearch.py -u https://target.com -e php,html,js
```

---

# 2. 🌐 Subdomain Enumeration

Subdomain enumeration is commonly used during the reconnaissance phase to identify authorized hosts belonging to a target.

## 🔧 Amass

### Installation

```bash
apt-get update
apt-get install amass
```

## 🔧 Subfinder

### Usage

```bash
subfinder -d target.com | tee subs.txt
```

This saves discovered subdomains to `subs.txt`.

---

# 3. 📸 Screenshotting

Screenshotting discovered web services can make it easier to review a large number of hosts.

## 🔧 Aquatone

### Installation

```bash
gem install aquatone
```

### Repository

[GitHub — Aquatone](https://github.com/teamghsoftware/aquatone)

---

# 4. 🔎 Parameter Discovery

Parameter discovery can help identify URLs containing parameters that may be relevant during authorized web application testing.

## 🔧 ParamSpider

### Installation

```bash
git clone https://github.com/devanshbatham/ParamSpider.git
cd ParamSpider
pip3 install -r requirements.txt
```

### Usage

```bash
python3 paramspider.py --domain target.com
```

---

# 5. 🛡️ Vulnerability Scanning

Automated scanners can help identify potential vulnerabilities and exposed technologies. Scanner findings should always be manually verified before reporting.

## 🔧 Scant3r

### Installation

```bash
git clone https://github.com/knassar702/scant3r.git
cd scant3r
pip3 install -r requirements.txt
```

### Usage

```bash
./scant3r.py -h
```

---

## 🔧 Nuclei

Nuclei can be used with templates to identify known vulnerabilities, misconfigurations, and exposed services.

### Run a Specific Template

For example, using a Swagger-related template:

```bash
cat result.txt | nuclei -t /Desktop/quick-nuclei/swagger.yaml
```

### Run Templates by Tag

For example, scanning with Spring Boot-related templates:

```bash
echo "target.com" | nuclei -t /nuclei_templates/ -tags Springboot
```

> 💡 Always review the templates you run and make sure their behavior is permitted by the target's program rules.

---

# 6. 🔄 Recon Workflow Pipeline

A typical reconnaissance workflow can be organized into the following stages:

```text
Subdomain Enumeration
        ↓
DNS Resolution
        ↓
HTTP Probing
        ↓
Port Discovery
        ↓
Directory Discovery
        ↓
Vulnerability Scanning
        ↓
Manual Verification
        ↓
Report
```

## Step 1 — Subdomain Enumeration

```bash
subfinder -d target.com | tee subs.txt
```

## Step 2 — Resolve Subdomains

```bash
cat subs.txt | puredns resolve | tee resolved.txt
```

## Step 3 — Probe Live HTTP Hosts

```bash
cat resolved.txt | httpx-toolkit -sc -location -silent | tee live.txt
```

## Step 4 — Port Discovery

```bash
cat resolved.txt | naabu -p 9000
```

## Step 5 — Port Discovery + HTTP Probing

```bash
cat resolved.txt | naabu -p 9000 | \
httpx -status-code -location -title | tee ports.txt
```

## Step 6 — Directory Fuzzing

```bash
ffuf \
  -w onelistforallmicro.txt \
  -u https://target.com/FUZZ \
  -mc 200,301,302
```

## Step 7 — Vulnerability Scanning

```bash
cat live.txt | nuclei -t /nuclei_templates/ -tags Springboot
```

---

# 7. 🔌 Network Scanning

Network discovery can be useful when the bug bounty scope explicitly includes network infrastructure.

## Advanced IP Scanner

A GUI-based tool for scanning IP ranges and discovering devices on authorized networks.

## Advanced Port Scanner

A GUI-based tool for port discovery and service detection.

> ⚠️ Network scanning can generate significant traffic. Confirm that the target's rules explicitly permit the activity before scanning.

---

# 8. 🕵️ OSINT & Threat Intelligence

Passive intelligence sources can provide useful information during reconnaissance without directly interacting with the target infrastructure.

| Resource                                      | Purpose                                       |
| --------------------------------------------- | --------------------------------------------- |
| [AlienVault OTX](https://otx.alienvault.com/) | Passive URL and domain intelligence           |
| [Wayback Machine](https://web.archive.org/)   | Historical URLs and content                   |
| [Shodan](https://www.shodan.io/)              | Search engine for internet-connected services |

## AlienVault OTX API

Example query:

```bash
curl "https://otx.alienvault.com/api/v1/indicators/domain/<DOMAIN>/url_list?limit=500&page=1"
```

Example:

```bash
curl "https://otx.alienvault.com/api/v1/indicators/domain/target.com/url_list?limit=500&page=1"
```

---

## 🔍 Shodan Search Filters

Shodan filters can help locate publicly indexed services associated with an authorized target.

### SSL Certificate

```text
ssl:"target.com"
```

### Hostname

```text
hostname:"target.com"
```

### Organization

```text
org:"Target Organization"
```

### HTTP Title + Hostname

```text
http.title:"Dashboard" hostname:"target.com"
```

---

# 9. 🔎 Google Dorking

Search-engine operators can help identify publicly indexed resources.

| Dork                                 | Purpose                                     |
| ------------------------------------ | ------------------------------------------- |
| `site:target.com`                    | Find indexed pages                          |
| `site:target.com ext:php`            | Find indexed PHP files                      |
| `site:target.com inurl:admin`        | Find pages containing `admin` in the URL    |
| `site:target.com intitle:"index of"` | Find potentially indexed directory listings |
| `site:target.com ext:log`            | Find indexed log files                      |
| `site:target.com ext:sql`            | Find indexed SQL-related files              |
| `site:target.com ext:env`            | Find indexed environment files              |
| `site:target.com inurl:api`          | Find indexed API-related URLs               |

> 💡 Search-engine results are not proof that a resource is accessible or vulnerable. Verify findings only within the authorized scope.

---

# 🚀 Quick Workflow Cheat Sheet

```bash
# 1. Subdomain Enumeration
subfinder -d target.com | tee subs.txt

# 2. DNS Resolution
cat subs.txt | puredns resolve | tee resolved.txt

# 3. HTTP Probing
cat resolved.txt | httpx-toolkit -sc -location -silent | tee live.txt

# 4. Port Discovery + HTTP Probing
cat resolved.txt | naabu -p 9000 | \
httpx -status-code -location -title | tee ports.txt

# 5. Directory Fuzzing
ffuf \
  -w onelistforallmicro.txt \
  -u https://target.com/FUZZ \
  -mc 200,301,302

# 6. Nuclei Scanning
cat live.txt | nuclei \
  -t /nuclei_templates/ \
  -tags Springboot
```

---

# 🧰 Tool Summary

| Category               | Tool            |
| ---------------------- | --------------- |
| Directory Discovery    | Dirsearch       |
| Subdomain Enumeration  | Amass           |
| Subdomain Enumeration  | Subfinder       |
| Screenshotting         | Aquatone        |
| Parameter Discovery    | ParamSpider     |
| Vulnerability Scanning | Scant3r         |
| Vulnerability Scanning | Nuclei          |
| DNS Resolution         | PureDNS         |
| HTTP Probing           | HTTPX           |
| Port Discovery         | Naabu           |
| Directory Fuzzing      | FFUF            |
| OSINT                  | AlienVault OTX  |
| Internet Intelligence  | Shodan          |
| Historical Discovery   | Wayback Machine |

---

# 📌 Notes

* Replace `target.com` with an **authorized target**.
* Do not scan systems outside the defined bug bounty scope.
* Respect program-specific rate limits.
* Avoid unnecessary traffic or disruptive testing.
* Manually verify automated scanner findings.
* Never access, download, modify, or disclose data that you are not authorized to access.
* Follow responsible disclosure requirements.
* Keep notes of commands, timestamps, hosts, endpoints, and evidence for reproducibility.

---

# ⚖️ Responsible Testing

This repository is intended for **authorized security research and educational purposes**.

