---
title: "5. Deploy the AI App"
layout: default
nav_order: 5
parent: Home
---

# Milestone 5: Deploy the AI App

Deploy the Streamlit application that provides natural language queries over SQL Server logs.

---

## Application Components

| File | Purpose |
|---|---|
| `streamlit-app/app.py` | Main Streamlit UI — chat interface for log queries |
| `streamlit-app/kql_prompt.py` | GPT-4o prompt engineering + KQL generation from natural language |
| `streamlit-app/learn_search.py` | Microsoft Learn MCP integration for documentation references |
| `streamlit-app/requirements.txt` | Python dependencies |

---

## How It Works

1. User types a natural language question about SQL logs
2. GPT-4o generates a KQL query based on table schemas in `kql_prompt.py`
3. Query executes against Log Analytics via `azure-monitor-query` SDK
4. Results are summarized by GPT-4o with Microsoft Learn references
5. All authentication uses **Managed Identity** — no API keys

---

## Deploy Option A: ZIP Deploy (Simplest)

```bash
cd streamlit-app
zip -r ../deploy.zip . -x "*.pyc" "__pycache__/*" ".env"

az webapp deploy \
  --resource-group "$RESOURCE_GROUP" \
  --name "$WEBAPP_NAME" \
  --src-path ../deploy.zip \
  --type zip

cd ..
rm deploy.zip
```

---

## Deploy Option B: Local Git

```bash
az webapp deployment source config-local-git \
  --resource-group "$RESOURCE_GROUP" \
  --name "$WEBAPP_NAME" \
  --query url -o tsv

# Add as remote and push
git remote add azure <url>
git push azure main
```

---

## Deploy Option C: GitHub Actions CI/CD

1. Get the publish profile:
   ```bash
   az webapp deployment list-publishing-profiles \
     --resource-group "$RESOURCE_GROUP" \
     --name "$WEBAPP_NAME" --xml
   ```
2. Store as GitHub secret `AZURE_WEBAPP_PUBLISH_PROFILE`
3. Use [Azure/webapps-deploy](https://github.com/Azure/webapps-deploy) action

---

## Verify Deployment

```bash
# Check HTTP response
curl -s -o /dev/null -w "%{http_code}" "https://${WEBAPP_NAME}.azurewebsites.net"
# Expected: 200

# Check application logs
az webapp log tail \
  --resource-group "$RESOURCE_GROUP" \
  --name "$WEBAPP_NAME" \
  --timeout 30
```

---

## Smoke Tests

```bash
# Test Azure OpenAI via Managed Identity
az rest --method post \
  --url "$AOAI_ENDPOINT/openai/deployments/$AOAI_DEPLOYMENT/chat/completions?api-version=2024-06-01" \
  --body '{"messages":[{"role":"user","content":"hello"}],"max_tokens":10}' \
  --headers Content-Type=application/json

# Test KQL query
az monitor log-analytics query \
  --workspace "$LAW_WORKSPACE_ID" \
  --analytics-query "Event | take 5 | project TimeGenerated, Source, EventLevelName" \
  --timespan P1D -o table
```

---

## Troubleshooting

| Symptom | Check | Fix |
|---|---|---|
| HTTP 500 | App logs | `az webapp log tail -g $RESOURCE_GROUP -n $WEBAPP_NAME` |
| `DefaultAzureCredential` fails | MI not enabled | Verify `principalId` exists on webapp identity |
| `AuthorizationFailed` to OpenAI | RBAC not assigned | Re-run RBAC from deploy.sh Step 7 |
| `ModuleNotFoundError` | Code not deployed | Deploy the app using one of the options above |
| No doc snippets in answers | Network restriction | Ensure outbound access to `learn.microsoft.com` |

---

> **Next:** [Milestone 6: Production & Cleanup →](6-production-cleanup.md)
