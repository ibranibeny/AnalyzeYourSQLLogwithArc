---
title: "1. Prerequisites"
layout: default
nav_order: 1
parent: Home
---

# Milestone 1: Prerequisites

Complete every item below before running the deployment script.

---

## Azure Subscription Access

```bash
az login
az account show --query "{Name:name, SubscriptionId:id, State:state}" -o table
```

You need **Contributor** on the subscription, plus **User Access Administrator** (or **Owner**) to create RBAC role assignments.

---

## Azure CLI Version

```bash
az version --query '"azure-cli"' -o tsv
# Required: 2.61.0 or later
az upgrade
```

---

## Azure CLI ML Extension

```bash
az extension add --name ml --upgrade --yes
az extension show --name ml --query version -o tsv
# Required: 2.26.0 or later
```

---

## Resource Provider Registration

```bash
# Check registration status
az provider show --namespace Microsoft.Compute --query registrationState -o tsv
az provider show --namespace Microsoft.OperationalInsights --query registrationState -o tsv
az provider show --namespace Microsoft.Insights --query registrationState -o tsv
az provider show --namespace Microsoft.CognitiveServices --query registrationState -o tsv
az provider show --namespace Microsoft.MachineLearningServices --query registrationState -o tsv
az provider show --namespace Microsoft.Web --query registrationState -o tsv

# Register any that show "NotRegistered"
az provider register --namespace Microsoft.Insights --wait
az provider register --namespace Microsoft.CognitiveServices --wait
az provider register --namespace Microsoft.MachineLearningServices --wait
```

---

## Regional Quotas

| Region | Resource | Minimum Required |
|---|---|---|
| Southeast Asia | Standard_D2s_v3 vCPUs | 2 vCPUs |
| Southeast Asia | App Service Plan (B1) | 1 instance |
| East US | Azure OpenAI GPT-4o Standard | 30K tokens per minute |

```bash
az vm list-usage --location southeastasia \
  --query "[?contains(localName, 'Standard DSv3 Family')].{Name:localName, Current:currentValue, Limit:limit}" \
  -o table
```

---

## Prepare Environment Variables

Create a `.env` file in the project root:

```bash
cat > .env << 'EOF'
export SUBSCRIPTION_ID="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
export RESOURCE_GROUP="rg-contoso-sqlobs"
export LOCATION_SEA="southeastasia"
export LOCATION_US="eastus"
export VM_NAME="vm-sql-sea-01"
export VM_SIZE="Standard_D2s_v3"
export VM_IMAGE="MicrosoftWindowsServer:WindowsServer:2022-datacenter-azure-edition:latest"
export ADMIN_USERNAME="contosoadmin"
export ADMIN_PASSWORD="YourStr0ng!Pass#2026"
export LAW_NAME="law-contoso-sqlobs"
export AI_HUB_NAME="ai-hub-contoso"
export AI_PROJECT_NAME="ai-proj-sqllogs"
export AOAI_NAME="aoai-contoso-sqllogs"
export AOAI_DEPLOYMENT="gpt-4o"
export AOAI_MODEL="gpt-4o"
export AOAI_MODEL_VERSION="2024-08-06"
export APP_PLAN_NAME="asp-contoso-streamlit"
export WEBAPP_NAME="app-contoso-sqllogs"
EOF
```

> ⚠️ **Security:** Never commit `.env` to source control.

---

> **Next:** [Milestone 2: Deploy Infrastructure →](2-deploy-infrastructure.md)
