# Secret Exposure Monitoring Pipeline - Setup Guide

## Quick Setup (10 minutes)

### Step 1: Prerequisites

- [ ] n8n instance running
- [ ] Google Sheets
- [ ] Internet connection
- [ ] (Optional) Gmail Account/Slack workspace for alerts

---

### Step 2: Import Workflow

1. Download workflow: [github-scanner-workflow.json](../workflows/semp-workflow.json)
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
2. Fetch contents from GitHub
3. Check the Google Sheets Detections Registry for duplicates
4. Triage findings by severity
 Check results:  
1. New findings will appear in your Google Sheet
2. You will receive a "Security Scan Report" if CRITICAL or MEDIUM risks are found

---

### Step 5: Review Findings

**CRITICAL RISK** (address immediately):
- Private SSH/RSA keys
- Hardcoded passwords in code

**MEDIUM RISK** (address within 24h):
- AWS keys/Github tokens
- Stripe / MongoDB / Google API Keys

**LOW RISK** (Monitoring only):
- Findings already present in the Google Sheet Registry
- Non-credible secrets (filtered by Entropy check)

---

## Customization

### Add Custom Patterns

Edit "Scan for Credentials" node:
```
Add new regex patterns or adjust the Shannon Entropy thresholds (e.g., increasing the AWS threshold to 4.0 for stricter matching).
```

Deduplication Management
```
To re-scan and alert on a previously found secret, simply delete its row from your Google Sheets `Security_Scan_Memory` file. The "Check If Already Reported" node will then treat it as a new detection.
```

### Schedule Regular Scans

1. Replace "Manual Trigger" with "Schedule Trigger"
2. Set interval (e.g., weekly)
3. Activate workflow

---

## Troubleshooting

**Gmail Alert is Blank:**
- Ensure you are using absolute references in the Gmail node (e.g., {{ $("Build Security Report").first().json.overallRisk }})

**Registry Not Updating:**
- Verify the "Save New Findings" node is correctly mapped to your Google Sheet ID and the Key column and others are exactly as named in the sheet

**False positives:**
- Update the `falsePositives` array in "Scan for Credentials" node to include specific placeholder strings used by your team

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
