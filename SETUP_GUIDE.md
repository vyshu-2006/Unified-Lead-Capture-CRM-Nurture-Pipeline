# 🚀 Complete Setup Guide

A step-by-step guide to get the Lead Capture → CRM → Nurture Pipeline up and running.

---

## 📋 Prerequisites Checklist

Before starting, ensure you have:

- [ ] **n8n Instance** (Cloud or Self-hosted)
- [ ] **HubSpot Account** (Free or Pro tier minimum)
- [ ] **Airtable Account** (Free tier is fine)
- [ ] **Gmail Account** (for sending emails)
- [ ] **Slack Workspace** (optional, for alerts)
- [ ] **WhatsApp Business Account** (optional)
- [ ] **Administrator Access** to all services

---

## Step 1: Import the Workflow

### 1.1 Download the Workflow File

The workflow file is available as:
```
Unified Lead Capture → CRM → Nurture Pipeline (1).json
```

### 1.2 Import into n8n

**Method A: Using n8n UI**

1. Open your n8n instance
2. Go to **Menu** (≡) → **Import from File**
3. Click **Select file**
4. Choose `Unified Lead Capture → CRM → Nurture Pipeline (1).json`
5. Click **Import**
6. Name the workflow (e.g., "Lead Pipeline - Production")
7. Click **Save**

**Method B: Using API** (Advanced)

```bash
curl -X POST http://your-n8n-instance/api/v1/workflows/import \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d @workflow.json
```

---

## Step 2: Configure HubSpot Integration

### 2.1 Create HubSpot OAuth2 Credentials

1. **Log in to HubSpot** → Settings
2. **Integrations** → **Private apps**
3. Click **Create private app**
4. **App name**: "n8n Lead Pipeline"
5. **Permissions** (Scopes Required):
   - **CRM** → **Contacts** → `read`, `write`
   - **CRM** → **Contacts** → `custom_properties_read`, `custom_properties_write`

6. Copy:
   - **Client ID**
   - **Client Secret**

### 2.2 Configure n8n Credential

1. In n8n, open the workflow
2. Find node: **"Add to HubSpot"**
3. Click on the **Credentials** field (if not already set)
4. Click **Create New Credential** → **HubSpot OAuth2 API**
5. Fill in:
   - **Client ID**: [From HubSpot]
   - **Client Secret**: [From HubSpot]
6. Click **Authenticate** (will redirect to HubSpot)
7. Click **Allow** to authorize
8. Save the credential

### 2.3 Verify HubSpot Setup

- Test the "Add to HubSpot" node with sample data
- Check HubSpot dashboard for test contact creation
- Verify custom properties exist (or create them if needed)

---

## Step 3: Configure Airtable Integration

### 3.1 Create Airtable Base

1. **Log in to Airtable**
2. Click **Create** → **Start from scratch**
3. **Base name**: "Lead Pipeline"
4. **Table name**: "Leads"

### 3.2 Create Table Structure

Add these columns to your Leads table:

| Column Name | Type | Notes |
|-------------|------|-------|
| Name | Single line text | Lead's full name |
| Email | Email | Primary identifier |
| Phone | Phone number | Contact number |
| Company | Single line text | Company name |
| Company size | Number | Number of employees |
| Source | Single select | website / linkedin / whatsapp |
| Score | Number | 0-100 scale |
| Temperature | Single select | hot / warm / cold |
| Created | Created time | Auto-populated |
| Last Modified | Last modified time | Auto-populated |

**Setup Single Select Options**:
- **Source**: "website", "linkedin", "whatsapp"
- **Temperature**: "hot", "warm", "cold"

### 3.3 Get Airtable Credentials

1. **Account settings** → **Personal access tokens**
2. Click **Create token**
3. **Token name**: "n8n Lead Pipeline"
4. **Scopes needed**:
   - `data.records:read`
   - `data.records:write`
   - `schema.bases:read`
5. **Access**: Select your base
6. **Duration**: 1 year or custom
7. Copy the token

### 3.4 Get Base & Table IDs

1. Open your Airtable base
2. Click **Share** → **Show API key**
3. URL format: `https://airtable.com/appXXXXXXXXXXXXXX/tblXXXXXXXXXXXXXX`
   - `appXXX...` = **Base ID**
   - `tblXXX...` = **Table ID**

### 3.5 Configure n8n Credential

1. In n8n, find node: **"Add to Airtable"**
2. Click **Credentials** field
3. Click **Create New Credential** → **Airtable OAuth2 API**
4. Fill in:
   - **Token**: [Your personal access token]
5. Save credential

### 3.6 Update Node Configuration

In the "Add to Airtable" node:

1. **Base**: Select your "Lead Pipeline" base
2. **Table**: Select "Leads" table
3. **Column Mapping**: Verify these mappings:
   - Name → Name
   - Email → Email
   - Phone → Phone
   - Company → Company
   - Company size → Company size
   - Source → Source
   - Lead score → Score
   - Lead temperature → Temperature

---

## Step 4: Configure Gmail Integration

### 4.1 Enable Gmail API

1. Go to [Google Cloud Console](https://console.cloud.google.com)
2. **Create new project** (or select existing)
3. **Project name**: "n8n Lead Pipeline"
4. **APIs & Services** → **Enable APIs and Services**
5. Search: **Gmail API**
6. Click **Enable**

### 4.2 Create OAuth2 Credentials

1. **Credentials** → **Create Credentials** → **OAuth 2.0 Client ID**
2. **Application type**: Web application
3. **Name**: "n8n"
4. **Authorized redirect URIs**: 
   ```
   https://your-n8n-instance/rest/oauth2/callback
   ```
5. Copy:
   - **Client ID**
   - **Client Secret**

### 4.3 Configure n8n Credential

All Gmail nodes in the workflow use the same credential:
- "Send Welcome Email"
- "Send Follow-up Email"
- "Send Offer Email"
- "Send Weekly Report Email"

1. Find any **Gmail node**
2. Click **Credentials** field
3. Click **Create New Credential** → **Gmail OAuth2 API**
4. Fill in:
   - **Client ID**: [From Google Cloud]
   - **Client Secret**: [From Google Cloud]
5. Click **Authenticate** (will redirect to Google)
6. Click **Allow** to authorize
7. Save credential

### 4.4 Update Email Content

Edit each email node to customize:
1. **Send Welcome Email** node
   - Subject: "Welcome, {{$json.name}}! ..."
   - Message: Your custom welcome text
2. **Send Follow-up Email** node
   - Subject: "Just checking in, {{$json.name}}"
   - Message: Your follow-up text
3. **Send Offer Email** nodes
   - Subject: "Special offer for {{$json.company}}"
   - Message: Your offer text
4. **Send Weekly Report Email** node
   - To: "sales.manager@yourcompany.com"
   - Subject: "Weekly Lead Pipeline Report"

---

## Step 5: Configure Slack Integration (Optional)

### 5.1 Create Slack App

1. Go to [Slack API Dashboard](https://api.slack.com)
2. Click **Create New App** → **From scratch**
3. **App name**: "Lead Pipeline Bot"
4. **Workspace**: Select your workspace
5. Click **Create App**

### 5.2 Configure OAuth Scopes

1. **OAuth & Permissions** (left sidebar)
2. **Scopes** → **Bot Token Scopes**
3. Add scopes:
   - `chat:write` - Send messages
   - `channels:manage` - Manage channels
4. Scroll to top → **Install App to Workspace**
5. Click **Allow**
6. Copy **Bot User OAuth Token** (starts with `xoxb-`)

### 5.3 Configure n8n Credentials

For **both** Slack nodes ("Slack Alert - Hot Lead" and "Send Weekly Report Slack"):

1. Click **Credentials** field
2. Click **Create New Credential** → **Slack OAuth2 API**
3. Paste your **Bot User OAuth Token**
4. Save credential

### 5.4 Create Slack Channels

Create two channels in your Slack workspace:

1. **#hot-leads**
   - Purpose: "Hot lead notifications from n8n"
   - Set as public channel

2. **#sales-pipeline**
   - Purpose: "Weekly pipeline reports"
   - Set as public channel

Then:
- Go to each channel → **Members**
- Add your Bot to both channels

### 5.5 Test Slack Connection

1. Find **"Slack Alert - Hot Lead"** node
2. Click **Test** (on the node)
3. Should see test message in #hot-leads channel

---

## Step 6: Configure WhatsApp Integration (Optional)

### 6.1 Set Up WhatsApp Business Account

1. Go to [Meta Business Platform](https://business.facebook.com)
2. Set up WhatsApp Business Account
3. Get **Phone Number ID**
4. Get **Business Account ID**

### 6.2 Generate Access Token

1. **Settings** → **System User**
2. Create system user (or select existing)
3. **Generate Token**
4. Permissions needed: `whatsapp_business_messaging`
5. Copy access token

### 6.3 Configure n8n Credentials

For **"Send Welcome WhatsApp"** node:

1. Click **Credentials** field
2. Click **Create New Credential** → **HTTP Header Auth**
3. Add header:
   - **Key**: `Authorization`
   - **Value**: `Bearer YOUR_ACCESS_TOKEN`
4. Save credential

### 6.4 Update WhatsApp Node

In **"Send Welcome WhatsApp"** node:
1. Update URL: Replace `YOUR_PHONE_NUMBER_ID` with your actual ID
2. Test with a WhatsApp number

---

## Step 7: Configure Webhooks

### 7.1 Extract Webhook URLs

Each webhook node has a unique URL. To get them:

1. **Website Form Webhook** node:
   - Click node
   - Look for **Webhook URL** in node info
   - Format: `https://your-instance/webhook/website-lead`
   - Copy this URL

2. **LinkedIn Lead Webhook** node:
   - Similar process
   - Format: `https://your-instance/webhook/linkedin-lead`

3. **WhatsApp Webhook** node:
   - Format: `https://your-instance/webhook/whatsapp-lead`

### 7.2 Website Form Integration

**For Typeform**:
1. Open your form
2. **Connect** → **Webhooks**
3. Click **Add webhook**
4. **URL**: Paste Website Form Webhook URL
5. **Events**: Select "Form Completed"
6. Save

**For HTML Form**:
```javascript
// JavaScript to submit form data to webhook
document.getElementById('contactForm').addEventListener('submit', async (e) => {
  e.preventDefault();
  
  const formData = {
    name: document.getElementById('name').value,
    email: document.getElementById('email').value,
    phone: document.getElementById('phone').value,
    company: document.getElementById('company').value,
    message: document.getElementById('message').value
  };
  
  await fetch('https://your-instance/webhook/website-lead', {
    method: 'POST',
    headers: {'Content-Type': 'application/json'},
    body: JSON.stringify(formData)
  });
  
  alert('Thank you for your submission!');
});
```

### 7.3 LinkedIn Lead Gen Forms

1. Create Lead Gen Form in LinkedIn Campaign Manager
2. **Lead Center** → **Webhooks**
3. Add webhook:
   - **URL**: LinkedIn Lead Webhook URL
   - **Events**: Form submissions
4. Save and test

### 7.4 WhatsApp Integration

1. Set up WhatsApp webhook in Meta Business Platform
2. **App Settings** → **Webhooks**
3. **URL**: WhatsApp Webhook URL
4. **Verify Token**: Generate any random string
5. **Subscribe to events**: messages

---

## Step 8: Test the Workflow

### 8.1 Test Webhook Ingestion

**Test Website Webhook**:
```bash
curl -X POST https://your-instance/webhook/website-lead \
  -H "Content-Type: application/json" \
  -d '{
    "name": "John Doe",
    "email": "john@example.com",
    "phone": "555-0123",
    "company": "Acme Corp",
    "message": "Interested in your product"
  }'
```

### 8.2 Monitor Execution

1. Open workflow
2. Click **Execution History** tab
3. Look for successful execution
4. Click execution to see detailed logs

### 8.3 Verify Data Flow

After successful test:

1. **Check HubSpot**: New contact should appear
2. **Check Airtable**: New record in Leads table
3. **Check Gmail**: Welcome email should arrive
4. **Check Slack** (if configured): Alert for hot leads

---

## Step 9: Activate Workflow

### 9.1 Pre-activation Checklist

Before going live:

- [ ] All credentials configured and tested
- [ ] Email templates customized
- [ ] Webhook URLs configured in source systems
- [ ] Slack channels created (if using)
- [ ] Airtable base structure verified
- [ ] HubSpot contact properties created
- [ ] Test lead processed successfully

### 9.2 Activate

1. Open the workflow
2. Click **Activate** button (top right)
3. Workflow is now **LIVE**

### 9.3 Monitor Initial Activity

1. Watch **Execution History** for new leads
2. Check for errors or failures
3. Monitor email delivery
4. Verify CRM sync

---

## Step 10: Customize & Optimize

### 10.1 Email Templates

Edit each email node with your brand voice:

```
# Welcome Email Template

Subject: Welcome, {{$json.name}}! Great to connect

Hi {{$json.name}},

Thanks for reaching out through {{$json.source}}. 
We're excited to learn more about {{$json.company}}.

We'll be in touch shortly with next steps.

Best regards,
[Your Sales Team]
```

### 10.2 Scoring Adjustments

Edit **"Score Lead"** code node to adjust:
- Company size point values
- Industry focus areas
- Source weighting
- Temperature thresholds

### 10.3 Wait Times

Modify wait periods in **Wait** nodes:
- **Wait 3 Days**: Adjust follow-up timing
- **Wait 4 Days**: Adjust offer email timing

---

## Step 11: Maintenance & Monitoring

### 11.1 Weekly Tasks

- [ ] Review execution history for errors
- [ ] Check HubSpot contact sync
- [ ] Verify email delivery
- [ ] Monitor Slack alerts

### 11.2 Monthly Tasks

- [ ] Analyze lead sources and conversion
- [ ] Review email performance
- [ ] Update email templates if needed
- [ ] Check API rate limits
- [ ] Verify OAuth token expiry dates

### 11.3 Quarterly Tasks

- [ ] Review lead scoring accuracy
- [ ] Analyze ROI by lead source
- [ ] Optimize nurture sequences
- [ ] Plan feature enhancements

---

## Troubleshooting

### Issue: Webhooks not receiving data

**Solution**:
1. Verify webhook URL is correct and accessible
2. Test URL manually with curl
3. Check n8n logs for webhook errors
4. Ensure webhook node is active

### Issue: Leads not syncing to HubSpot

**Solution**:
1. Check HubSpot OAuth2 credentials
2. Verify HubSpot API quota
3. Review error logs in execution history
4. Test "Add to HubSpot" node manually

### Issue: Emails not sending

**Solution**:
1. Verify Gmail OAuth2 token validity
2. Check Gmail security settings
3. Test Gmail node with sample data
4. Review execution logs for specific errors

### Issue: Airtable write failing

**Solution**:
1. Verify Airtable credentials
2. Check table structure matches mappings
3. Verify column names exactly match
4. Test Airtable node with sample data

### Issue: Slack notifications not appearing

**Solution**:
1. Check Slack token validity
2. Verify bot is member of channels
3. Check channel names are correct
4. Test Slack node with sample message

---

## Best Practices

1. **Secure Credentials**: Never share or expose OAuth tokens
2. **Regular Backups**: Export workflow weekly
3. **Test First**: Always test changes in dev environment
4. **Monitor Costs**: Watch API call volumes
5. **Clean Data**: Validate input before processing
6. **Document Changes**: Keep changelog of modifications
7. **Audit Logs**: Review execution history regularly

---

## Getting Help

- **n8n Docs**: https://docs.n8n.io
- **HubSpot Support**: https://support.hubspot.com
- **Airtable Support**: https://support.airtable.com
- **Gmail API Docs**: https://developers.google.com/gmail/api
- **Slack API Docs**: https://api.slack.com/docs

---

**Setup Guide Complete! | Last Updated: July 2026**

**Next Steps**: Customize email templates, adjust scoring logic, and start capturing leads! 🚀
