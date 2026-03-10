# PagerDuty L1/L2 Acknowledgment Automation

> **Automatically add Layer 2 responders to incidents when Layer 1 doesn't have sufficient acknowledgments**

This repository contains a GitHub Actions workflow that integrates with PagerDuty Incident Workflows to implement custom escalation logic. If less than 2 responders acknowledge an incident within 3 minutes, Layer 2 responders from your escalation policy are automatically added to the incident.

---

## 📋 Table of Contents

- [How It Works](#how-it-works)
- [Prerequisites](#prerequisites)
- [Setup Guide](#setup-guide)
  - [Step 1: Add GitHub Action Workflow](#step-1-add-github-action-workflow)
  - [Step 2: Create PagerDuty API Key](#step-2-create-pagerduty-api-key)
  - [Step 3: Add Secrets to GitHub](#step-3-add-secrets-to-github)
  - [Step 4: Create GitHub Personal Access Token](#step-4-create-github-personal-access-token)
  - [Step 5: Create PagerDuty Incident Workflow](#step-5-create-pagerduty-incident-workflow)
  - [Step 6: Configure Workflow Trigger](#step-6-configure-workflow-trigger)
  - [Step 7: Add Webhook Action](#step-7-add-webhook-action)
  - [Step 8: Publish the Workflow](#step-8-publish-the-workflow)
- [Testing](#testing)
- [Customization](#customization)
- [Troubleshooting](#troubleshooting)
- [How It Handles Schedules](#how-it-handles-schedules)

---

## 🔄 How It Works

**Flow:**

1. Incident Triggered (on selected services)
2. PagerDuty Incident Workflow - Triggered on incident creation - Sends webhook to GitHub Actions
3. GitHub Actions Workflow - Waits 3 minutes - Checks acknowledgment count - If < 2 acks: Adds Layer 2 responders - If ≥ 2 acks: No action needed

### Key Features:
- ✅ **Automatic Layer 2 escalation** based on acknowledgment count
- ✅ **Works with schedules** - Automatically resolves who is on-call
- ✅ **Works with multiple schedules** in Layer 2
- ✅ **Preserves existing assignments** - Adds Layer 2 without removing Layer 1
- ✅ **Detailed logging** - Full visibility in GitHub Actions logs

---

## 📦 Prerequisites

Before you begin, ensure you have:

1. **PagerDuty Account** with admin or manager access
2. **GitHub Repository** (this one!)
3. **Escalation Policy** with at least 2 layers:
   - **Layer 1**: Your primary responders (users or schedules)
   - **Layer 2**: Your secondary responders (users or schedules)
4. **Service(s)** configured to use the escalation policy

---

## 🚀 Setup Guide

### Step 1: Add GitHub Action Workflow

The workflow file already exists in this repository at `.github/workflows/check-incident-acks.yml`.

**✅ This step is already complete!**

---

### Step 2: Create PagerDuty API Key

1. Log in to your PagerDuty account
2. Navigate to **Integrations** → **API Access Keys**
3. Click **Create New API Key**
4. Give it a description (e.g., "GitHub Actions - L1/L2 Automation")
5. **Important**: Make sure it's NOT read-only (needs write access)
6. Click **Create Key**
7. **Copy the API key immediately** - you won't be able to see it again!

---

### Step 3: Add Secrets to GitHub

You need to add two secrets to your GitHub repository:

1. Go to your GitHub repository
2. Click **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret**

#### Add Secret #1: `PD_API_KEY`
- **Name**: `PD_API_KEY`
- **Value**: [Paste your PagerDuty API key from Step 2]
- Click **Add secret**

#### Add Secret #2: `PD_FROM_EMAIL`
- **Name**: `PD_FROM_EMAIL`
- **Value**: [Your PagerDuty login email address]
- Click **Add secret**

> **Why email?** The PagerDuty API requires a `From` header with a valid user email for audit logging purposes.

---

### Step 4: Create GitHub Personal Access Token

This allows PagerDuty to trigger your GitHub Action.

1. Go to **GitHub** → Click your profile picture → **Settings**
2. Scroll down to **Developer settings** (bottom left)
3. Click **Personal access tokens** → **Tokens (classic)**
4. Click **Generate new token** → **Generate new token (classic)**
5. Give it a note (e.g., "PagerDuty Incident Workflow")
6. Set expiration (recommend: 90 days or No expiration)
7. Select scope: **`repo`** (Full control of private repositories)
8. Click **Generate token**
9. **Copy the token immediately** - you won't see it again!

---

### Step 5: Create PagerDuty Incident Workflow

1. Log in to **PagerDuty**
2. Navigate to **Automation** → **Incident Workflows**
3. Click **Create Workflow**
4. Name it: **"Check L1 Acknowledgments and Add L2"**
5. Click **Create Workflow**

You should now see the workflow builder.

---

### Step 6: Configure Workflow Trigger

1. Click on the **Conditional Trigger** box at the top
2. Set trigger to: **"Incident is triggered"**
3. **(Optional)** Add conditions to filter by:
   - Specific services
   - Priority levels
   - Teams
4. Click **Save**

> **Note**: If you don't add any conditions, this will run for ALL incidents on ALL services.

---

### Step 7: Add Webhook Action

1. Click the **"+"** button to add an action
2. Search for and select **"Send a webhook POST request"**
3. Configure the webhook:

**Webhook URL:**

https://api.github.com/repos/YOUR_GITHUB_USERNAME/incident-ack-count/dispatches

Replace `YOUR_GITHUB_USERNAME` with your actual GitHub username!

**Headers** (click "Add header" for each):

| Header Name | Header Value |
|-------------|--------------|
| `Accept` | `application/vnd.github+json` |
| `Authorization` | `Bearer YOUR_GITHUB_PAT` |
| `X-GitHub-Api-Version` | `2022-11-28` |

> Replace `YOUR_GITHUB_PAT` with the Personal Access Token you created in Step 4.

**Body** (select "JSON" format):

{
  "event_type": "check-incident-acks",
  "client_payload": {
    "incident_id": "{{incident.id}}",
    "incident_number": "{{incident.incident_number}}"
  }
}

4. Click **Save**

---

### Step 8: Publish the Workflow

1. Go back to your PagerDuty Incident Workflow
2. Click the blue **"Publish"** button in the top right
3. Confirm by clicking **"Publish"** again

**✅ Your workflow is now active!**

---

## 🧪 Testing

### Test the Automation

1. **Trigger a test incident** on one of the services associated with your workflow
   - You can create a test incident manually in PagerDuty
   - Or trigger one through your monitoring tool

2. **What should happen:**
   - ⏱️ Incident triggers and notifies Layer 1 responders
   - 📤 PagerDuty sends webhook to GitHub
   - ⏳ GitHub Action waits 3 minutes
   - 🔍 GitHub Action checks acknowledgment count
   - **If < 2 acks**: Layer 2 responders get added to the incident
   - **If ≥ 2 acks**: No action (workflow completes successfully)

3. **View the logs:**
   - Go to your GitHub repository
   - Click **Actions** tab
   - You should see a workflow run with your incident number
   - Click on it to see detailed logs

### Expected Behavior:

- **0-1 acknowledgments after 3 minutes** → Layer 2 responders added ✅
- **2+ acknowledgments after 3 minutes** → No action needed ✅
- **Incident resolved before 3 minutes** → No action needed ✅

---

## ⚙️ Customization

### Change the Wait Time

Edit `.github/workflows/check-incident-acks.yml` and change line 17:

sleep 180  # Change 180 to desired seconds (e.g., 300 for 5 minutes)

### Change the Acknowledgment Threshold

Edit `.github/workflows/check-incident-acks.yml` and change line 67:

if ack_count < 2 and status != "resolved":  # Change 2 to desired threshold

### Target a Different Layer

By default, this uses Layer 2 (index 1). To use Layer 3 instead, edit line 83:

layer_2_targets = escalation_rules[1]['targets']  # Change to [2] for Layer 3

---

## 🔍 How It Handles Schedules

### Single Schedule in Layer 2
If Layer 2 contains a schedule, the script automatically:
1. Fetches the current on-call user(s) from that schedule
2. Adds them to the incident

### Multiple Schedules in Layer 2
If Layer 2 contains multiple schedules, the script:
1. Loops through each schedule
2. Fetches the on-call user from each schedule
3. Adds all on-call users to the incident simultaneously

### Example:
**Escalation Policy Structure:**
- **Layer 1**: Schedule "Primary On-Call" (2 users on-call)
- **Layer 2**: 
  - Schedule "Secondary On-Call" (1 user on-call)
  - Schedule "Manager On-Call" (1 user on-call)

**What happens:**
- Incident triggers → 2 users from Layer 1 get paged
- After 3 minutes, if < 2 acks → Both on-call users from Layer 2 schedules get added

---

## 🐛 Troubleshooting

### Workflow isn't triggering

**Check:**
- ✅ PagerDuty Incident Workflow is **Published** (not disabled)
- ✅ Incident matches the conditional trigger filters
- ✅ GitHub PAT is valid and has `repo` scope
- ✅ Webhook URL is correct (check your GitHub username)

### GitHub Action fails with "401 Unauthorized"

**Fix:**
- Regenerate your GitHub Personal Access Token
- Make sure it has `repo` scope
- Update the webhook in PagerDuty with the new token

### GitHub Action fails with "404 Not Found" from PagerDuty API

**Fix:**
- Verify your `PD_API_KEY` secret is correct
- Make sure the API key has write permissions (not read-only)

### Layer 2 responders aren't being added

**Check:**
- ✅ Escalation Policy has at least 2 layers
- ✅ Layer 2 has users or schedules assigned
- ✅ `PD_FROM_EMAIL` secret matches a valid PagerDuty user

### View detailed logs

1. Go to your GitHub repository
2. Click **Actions** tab
3. Click on the workflow run
4. Click on **check-and-add-responders** job
5. Expand each step to see detailed output

---

## 📚 Additional Resources

- [PagerDuty API Documentation](https://developer.pagerduty.com/api-reference/)
- [PagerDuty Incident Workflows](https://support.pagerduty.com/docs/incident-workflows)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)

---

## 🤝 Support

If you encounter issues:
1. Review the GitHub Actions logs for detailed error messages
2. Verify all secrets are correctly configured

---

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---
