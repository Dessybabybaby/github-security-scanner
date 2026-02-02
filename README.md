# GitHub Security Scanner

> Automated security scanner using n8n to detect exposed credentials (API keys, passwords, tokens) in public GitHub repositories

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![n8n](https://img.shields.io/badge/n8n-workflow-orange)](https://n8n.io)

![Workflow Screenshot](media/workflow-screenshot-gss.png)

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Demo](#demo)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Usage](#usage)
- [Detection Patterns](#detection-patterns)
- [Sample Data](#sample-data)
- [Troubleshooting](#troubleshooting)
- [Security & Ethics](#security--ethics)
- [License](#license)

---

## Overview

**Problem:** Developers accidentally commit sensitive credentials (AWS keys, API tokens, private keys, passwords) to public GitHub repositories. In 2023, over 10 million secrets were exposed on GitHub, leading to data breaches and unauthorized access.

**Solution:** This n8n workflow automatically scans public GitHub repositories for exposed credentials using regex pattern matching. It checks README files, code files, and configuration files for common secret patterns and generates detailed security reports.

**Technology:** 
- n8n (workflow automation)
- GitHub REST API (public endpoints - no auth required for public repos)
- Regex pattern matching for credential detection
- CSV/Slack reporting

**Use Cases:**
- Security audits of your organization's public repos
- OSINT reconnaissance for penetration testing
- Developer education (show what NOT to commit)
- Automated security monitoring

---

## Features

- **Pattern-based detection** - Detects 15+ types of credentials
- **Multi-file scanning** - Checks README, config files, source code
- **Zero authentication** - Uses public GitHub API (no token needed)
- **Detailed reporting** - CSV logs with file paths and matched patterns
- **Slack alerts** - Real-time notifications for findings
- **Batch processing** - Scan multiple repos in one execution
- **False positive filtering** - Excludes example/placeholder credentials

---

## Demo

### Audio Case Study (Coming Soon)
Full walkthrough will be available after workflow build completion.

### Visual Demo
![Demo GIF](media/demo.gif)

---

## Prerequisites

**Required:**
- **n8n instance** (self-hosted or cloud)
- **Internet connection** (to access GitHub public API)

**Optional:**
- **Slack workspace** (for instant alerts)
- **GitHub account** (for higher rate limits - 60/hr unauthenticated, 5000/hr authenticated)

---

## Installation

### Quick Start: Import Workflow (5 minutes)

1. **Download workflow export:**
   - Go to: [Releases](https://github.com/Dessybabybaby/github-security-scanner/releases)
   - Download `github-scanner-workflow.json`

2. **Import to n8n:**
   - n8n UI → Workflows → Add Workflow → Import from File
   - Select `github-scanner-workflow.json`

3. **Configure target repositories:**
   - Edit "Load Target Repos" node
   - Add GitHub repo URLs to scan

4. **Test execution:**
   - Click "Execute Workflow"
   - Review scan results

5. **Activate for scheduled scans:**
   - Toggle "Active" switch

---

## Usage

### Scanning Repositories

**Manual Scan:**
1. Open workflow in n8n
2. Edit "Load Target Repos" node
3. Add repo URLs:
```javascript
   [
     'https://github.com/user/repo1',
     'https://github.com/user/repo2'
   ]
```
4. Click "Execute Workflow"
5. Check results in CSV output or Slack

**Scheduled Scans:**
- Workflow can run daily/weekly to monitor repos
- Configure in Schedule Trigger node

### Reading Scan Results

**CSV Output Format:**
```csv
Timestamp,Repository,File,Pattern-Type,Matched-Text,Line-Number,Severity
2026-01-19T15:30:00Z,user/repo1,README.md,AWS Access Key,AKIA...,45,HIGH
2026-01-19T15:30:00Z,user/repo1,config.yml,Private Key,-----BEGIN PRIVATE KEY-----,12,CRITICAL
```

**Severity Levels:**
- **CRITICAL:** Private keys, database passwords
- **HIGH:** API keys, access tokens
- **MEDIUM:** Email addresses in code
- **LOW:** Placeholder/example credentials (filtered out)

---

## Detection Patterns

### Supported Credential Types

| Pattern Type | Regex | Example | Severity |
|--------------|-------|---------|----------|
| **AWS Access Key** | `AKIA[0-9A-Z]{16}` | AKIA1234567890ABCDEF | HIGH |
| **AWS Secret Key** | `[0-9a-zA-Z/+=]{40}` | wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY | HIGH |
| **GitHub Token** | `gh[pousr]_[A-Za-z0-9_]{36,255}` | ghp_1234567890abcdefghijklmnopqr | HIGH |
| **Private SSH Key** | `-----BEGIN (RSA\|DSA\|EC\|OPENSSH) PRIVATE KEY-----` | -----BEGIN RSA PRIVATE KEY----- | CRITICAL |
| **Generic API Key** | `api[_-]?key['\"]?\s*[:=]\s*['\"]\w{20,}` | api_key="sk_live_abc123..." | HIGH |
| **Password in Code** | `password\s*[:=]\s*['\"]\w+` | password="MyPassword123" | CRITICAL |
| **Database Connection** | `mongodb://[^:]+:[^@]+@` | mongodb://user:pass@host | CRITICAL |
| **Slack Token** | `xox[baprs]-[0-9a-zA-Z-]+` | xoxb-123456-abcdef | HIGH |
| **Google API Key** | `AIza[0-9A-Za-z_-]{35}` | AIzaSyD... | HIGH |
| **Stripe Key** | `sk_(live\|test)_[0-9a-zA-Z]{24,}` | sk_live_abc123... | HIGH |

**Full pattern file:** See `patterns/credential-patterns.json`

---

## Sample Data

### Test Repositories

Included sample repos with intentional (safe) "vulnerabilities" for testing:

- `sample-data/test-repos.csv` - List of public repos for testing
- `sample-data/scan-results-sample.csv` - Expected output
- `patterns/credential-patterns.json` - All detection regex patterns

### Running Test Scan

1. Use sample repo: `https://github.com/octocat/Hello-World`
2. Execute workflow
3. Compare output with `sample-data/scan-results-sample.csv`

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| **Rate limit exceeded** | GitHub API limits: 60/hr unauthenticated. Add GitHub token to increase to 5000/hr |
| **File not found errors** | Some repos may not have README. Workflow handles this gracefully |
| **Too many false positives** | Adjust regex patterns in "Detect Credentials" node to be more specific |
| **Slow execution** | Scanning large repos is slow. Consider scanning specific files/folders only |

**Enable Debug Logging:**
```bash
# Docker n8n:
docker logs -f n8n

# Check execution details in n8n UI → Executions tab
```

---

## Security & Ethics

### IMPORTANT: Ethical Use Guidelines

**This tool is for educational and authorized security testing ONLY.**

**Permitted Uses:**
- Scanning your own organization's public repositories
- Security audits with written permission
- Educational demonstrations (with sanitized data)
- Responsible disclosure to repository owners

**Prohibited Uses:**
- Scanning repos without permission for malicious purposes
- Using discovered credentials to access systems
- Publicly disclosing found credentials
- Automated mass-scanning of unknown repos

**Responsible Disclosure:**
If you discover credentials in someone else's repo:
1. DO NOT use the credentials
2. Contact the repo owner privately (GitHub Issues, email)
3. Give them 90 days to remediate before public disclosure
4. Follow coordinated vulnerability disclosure practices

**Legal Considerations:**
- Scanning public repos is legal (public data)
- Using discovered credentials without authorization is illegal
- Always follow ethical hacking principles

---

## License

This project is licensed under the MIT License - see [LICENSE](LICENSE) file.

**Disclaimer:** This tool is provided for educational purposes. Users are responsible for ensuring their use complies with applicable laws and ethical standards.

---

## Acknowledgments

- Inspired by [Mad Hat](https://youtube.com/@MadHat) - OSINT and security research
- Built with [n8n.io](https://n8n.io)
- Pattern database inspired by [GitGuardian](https://github.com/GitGuardian/sample_secrets)

---

## Contact

**Creator:** Achusi Desmond
- Portfolio: [My Story](https://Dessybabybaby.github.io/portfolio-site)
- GitHub: [Dessybabybaby](https://github.com/Dessybabybaby)
- LinkedIn: [Achusi Desond](https://linkedin.com/in/achusi-desmond)

---

**If this tool helps improve your security posture, please star the repo!**

**Use responsibly. Report vulnerabilities ethically.**
