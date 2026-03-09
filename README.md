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
