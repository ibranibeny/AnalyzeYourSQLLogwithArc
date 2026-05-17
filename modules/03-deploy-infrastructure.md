---
title: "3. Deploy Infrastructure"
nav_order: 4
parent: Home
description: "Provision all 10 Azure resources with a single script"
permalink: /modules/03-deploy-infrastructure/
---

# Module 3 — Deploy Infrastructure
{: .fs-8 }

The deployment creates **10 Azure resources** across **2 regions** using a single script.
{: .fs-5 .fw-300 }

---

## Resources Created

| Step | Resources | Region |
|---|---|---|
| 1 | Resource Group | Southeast Asia |
| 2 | Log Analytics Workspace (PerGB2018, 30-day retention) | Southeast Asia |
| 3 | Windows VM (D2s_v3) + system-assigned Managed Identity | Southeast Asia |
| 4 | AMA extension + Data Collection Endpoint + DCR + DCR association | Southeast Asia |
| 5 | Azure OpenAI Service (S0) + GPT-4o model deployment (30K TPM) | East US |
| 6 | AI Foundry Hub + Project + credential-less OpenAI connection | East US |
| 7 | App Service Plan (B1 Linux) + Web App + MI + RBAC role assignments | Southeast Asia |

**Estimated time:** 15–25 minutes.

---

## Run the Deployment

```bash
source .env
bash scripts/deploy.sh
```

### Successful Output

```
============================================================
 Deployment Complete
============================================================

 Resource Group:       rg-contoso-sqlobs
 VM:                   vm-sql-sea-01 (southeastasia)
 Log Analytics:        law-contoso-sqlobs
 Azure OpenAI:         https://aoai-contoso-sqllogs.openai.azure.com/
 AI Hub:               ai-hub-contoso (eastus)
 AI Project:           ai-proj-sqllogs
 Web App:              https://app-contoso-sqllogs.azurewebsites.net

 Authentication:       Managed Identity (no API keys)
   App Service → OpenAI:   Cognitive Services OpenAI User
   App Service → Logs:     Log Analytics Reader
```

---

## Verify RBAC Assignments

```bash
WEBAPP_PRINCIPAL_ID=$(az webapp identity show \
  --resource-group "$RESOURCE_GROUP" \
  --name "$WEBAPP_NAME" \
  --query principalId -o tsv)

az role assignment list \
  --assignee "$WEBAPP_PRINCIPAL_ID" \
  --query "[].{Role:roleDefinitionName, Scope:scope}" \
  -o table
```

| Role | Scope |
|---|---|
| Cognitive Services OpenAI User | .../Microsoft.CognitiveServices/accounts/aoai-contoso-sqllogs |
| Log Analytics Reader | .../Microsoft.OperationalInsights/workspaces/law-contoso-sqlobs |

---

## Troubleshooting

| Error | Cause | Fix |
|---|---|---|
| `SUBSCRIPTION_ID is not set` | Missing env var | Run `source .env` |
| `namespace not registered` | Provider not registered | `az provider register --namespace <ns> --wait` |
| `VM size not available` | Quota exhausted | `az vm list-usage --location southeastasia -o table` |
| `resource name already in use` | Name conflict | Change to a unique value in `.env` |

---

[← Architecture]({{ site.baseurl }}{% link architecture.md %}){: .btn .btn-outline .fs-5 .mb-4 .mb-md-0 .mr-2 }
[Next: Configure SQL Server →]({{ site.baseurl }}{% link modules/04-configure-sql.md %}){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 }
