---
title: "6. Production & Cleanup"
layout: default
nav_order: 6
parent: Home
---

# Milestone 6: Production & Cleanup

Harden for production use or tear down all resources.

---

## Security Posture

### What This Deployment Does Right

| Practice | Implementation |
|---|---|
| **No secrets in app settings** | Managed Identity replaces API keys |
| **Least privilege RBAC** | OpenAI User (not Contributor); Log Analytics Reader (not Contributor) |
| **System-assigned identity** | Auto-cleaned on App Service deletion |
| **No credentials in code** | `DefaultAzureCredential` handles all tokens |
| **HTTPS by default** | App Service enforces TLS |

### Recommended Hardening

| Risk | Fix |
|---|---|
| Public VM IP with RDP | Use Azure Bastion or JIT VM access |
| Public App Service endpoint | Add IP restrictions or Azure Front Door + WAF |
| Azure OpenAI public endpoint | Enable Azure Private Link |
| SQL Server SA password in env | Use Azure Key Vault |

---

## Production Networking

```bash
# Remove public IP from VM
az network nic ip-config update \
  --resource-group "$RESOURCE_GROUP" \
  --nic-name "${VM_NAME}VMNic" \
  --name ipconfig1 \
  --remove publicIpAddress

# Deploy Azure Bastion
az network bastion create \
  --resource-group "$RESOURCE_GROUP" \
  --name "bastion-contoso" \
  --vnet-name "${VM_NAME}VNET" \
  --location "$LOCATION_SEA"

# App Service VNet Integration
az webapp vnet-integration add \
  --resource-group "$RESOURCE_GROUP" \
  --name "$WEBAPP_NAME" \
  --vnet "${VM_NAME}VNET" \
  --subnet "webapp-subnet"
```

---

## Scaling Recommendations

| Component | Current | Production |
|---|---|---|
| VM size | D2s_v3 (2 vCPU) | D4s_v3 or higher |
| SQL Server | Express (10 GB limit) | Standard or Enterprise |
| App Service | B1 (1 core) | S1 or P1v3 with auto-scale |
| OpenAI TPM | 30K | 100K+ |
| Log retention | 30 days | 90+ days |

---

## Day-2 Operations

```bash
# Enable Application Insights
az monitor app-insights component create \
  --resource-group "$RESOURCE_GROUP" \
  --app "ai-contoso-streamlit" \
  --location "$LOCATION_SEA" --kind web

# Update GPT-4o model version
az cognitiveservices account deployment create \
  --resource-group "$RESOURCE_GROUP" \
  --name "$AOAI_NAME" \
  --deployment-name "$AOAI_DEPLOYMENT" \
  --model-name "gpt-4o" \
  --model-version "2024-11-20" \
  --model-format OpenAI \
  --sku-name "Standard" --sku-capacity 30
```

---

## Cleanup

### Full Teardown

```bash
bash scripts/cleanup.sh
```

Deletes the entire resource group and all resources. Prompts for confirmation.

### Selective Cleanup

```bash
# Delete only the web app
az webapp delete --resource-group "$RESOURCE_GROUP" --name "$WEBAPP_NAME"

# Delete only AI resources
az cognitiveservices account delete --resource-group "$RESOURCE_GROUP" --name "$AOAI_NAME"
az ml workspace delete --resource-group "$RESOURCE_GROUP" --name "$AI_PROJECT_NAME" --yes
az ml workspace delete --resource-group "$RESOURCE_GROUP" --name "$AI_HUB_NAME" --yes

# Stop VM to save costs (keep disk)
az vm deallocate --resource-group "$RESOURCE_GROUP" --name "$VM_NAME"
```

---

## Next Steps

- [Azure Monitor best practices](https://learn.microsoft.com/azure/azure-monitor/best-practices)
- [Azure OpenAI production deployment checklist](https://learn.microsoft.com/azure/ai-services/openai/how-to/production-best-practices)
- [Managed identity best practices](https://learn.microsoft.com/entra/identity/managed-identities-azure-resources/managed-identity-best-practice-recommendations)

---

> ← [Back to Workshop Home](../)
