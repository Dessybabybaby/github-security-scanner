# GitHub Security Scanner - Setup Guide

## Quick Setup (10 minutes)

### Step 1: Prerequisites

- [ ] n8n instance running
- [ ] Internet connection
- [ ] (Optional) Slack workspace for alerts

---

### Step 2: Import Workflow

1. Download workflow: [github-scanner-workflow.json](../workflows/github-scanner-workflow.json)
2. n8n UI → Workflows → Import from File
3. Select downloaded JSON

---

### Step 3: Configure Target Repositories

1. Open "Load Target Repos" node
2. Edit the `repositories` array:
```javascript
   const repositories = [
     'https://github.com/YOUR-ORG/repo1',
     'https://github.com/YOUR-ORG/repo2',
     // Add more repos...
   ];
```
3. Save

---

### Step 4: Test Scan

1. Click "Execute Workflow"
2. Wait for execution to complete
3. Check results:
   - CSV: `/tmp/github-scan-results.csv`
   - Slack: #security-alerts channel

---

### Step 5: Review Findings

**Critical Findings** (address immediately):
- Private keys
- Database passwords
- Production API keys

**High Findings** (address within 24h):
- AWS keys
- OAuth tokens
- Stripe keys

**Medium/Low** (review and prioritize):
- Test credentials
- Development tokens

---

## Customization

### Add Custom Patterns

Edit "Load Credential Patterns" node:
```javascript
const patterns = [
  // ...existing patterns...
  {
    name: 'Your Custom Pattern',
    regex: 'your-regex-here',
    severity: 'HIGH'
  }
];
```

### Scan Multiple Files

Currently scans README only. To scan all files:

1. Add HTTP Request node after "Fetch README"
2. URL: `={{ $json.apiUrl }}/git/trees/main?recursive=1`
3. Loop through files and fetch each
4. Concatenate content for scanning

### Schedule Regular Scans

1. Replace "Manual Trigger" with "Schedule Trigger"
2. Set interval (e.g., weekly)
3. Activate workflow

---

## Troubleshooting

**No findings detected:**
- Verify target repos have README files
- Check if patterns match actual credential formats
- Review execution logs for errors

**Rate limit errors:**
- GitHub API limit: 60 requests/hour (unauthenticated)
- Add GitHub token to increase limit to 5000/hour
- In HTTP Request node → Authentication → Header Auth
  - Name: `Authorization`
  - Value: `token YOUR_GITHUB_TOKEN`

**False positives:**
- Add keywords to `falsePositives` array in "Load Patterns" node
- Adjust regex patterns to be more specific

---

## Ethical Use Reminder

**DO:**
- Scan your own organization's repos
- Get written permission before scanning others' repos
- Report findings responsibly (private disclosure)

**DON'T:**
- Use discovered credentials without authorization
- Mass-scan random repos without permission
- Publicly disclose findings without coordination

---

## Support

- Issues: [GitHub Issues](../issues)
- Documentation: [README](../README.md)
