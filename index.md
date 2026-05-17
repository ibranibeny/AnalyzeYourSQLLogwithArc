---
title: Home
layout: home
nav_order: 0
---

# Analyze Your SQL Logs with Azure Arc & AI

> A hands-on workshop by **Contoso Ltd.** (Singapore Financial Services)

---

## Workshop Goals

| | |
|---|---|
| **What you will learn** | Deploy an end-to-end AI-powered SQL Server log analysis solution using Azure Monitor Agent, GPT-4o, and Microsoft Learn MCP |
| **What you'll need** | Azure subscription (Contributor + User Access Admin), Azure CLI 2.61+, WSL/Bash |
| **Duration** | 60–90 minutes |
| **Microsoft Cloud Topics** | Azure Monitor, Log Analytics, Azure OpenAI, Azure AI Foundry, App Service, Managed Identity |
| **Just want to see the app?** | [Streamlit App Code](https://github.com/ibranibeny/AnalyzeYourSQLLogwithArc/tree/main/streamlit-app) |

---

## Solution Architecture

```
┌──────────────────────────── SOUTHEAST ASIA ────────────────────────────────┐
│                                                                             │
│   Azure VM (Win 2022)              Log Analytics Workspace                  │
│   SQL Server Express   ──AMA/DCR──▶  Event table + Perf table              │
│                                           │                                 │
│                                           │ KQL queries                     │
│                                           ▼                                 │
│   Azure App Service (Linux B1)    ◀───── azure-monitor-query SDK            │
│   Streamlit "Talk to Your SQL Logs"                                         │
│     │                                                                       │
│     │  Managed Identity                                                     │
│     │  (Cognitive Services OpenAI User)                                     │
│     │  (Log Analytics Reader)                                               │
└─────┼───────────────────────────────────────────────────────────────────────┘
      │
      │                     ┌───────── EAST US ─────────┐
      │                     │                            │
      ├───GPT-4o calls────▶ │  Azure OpenAI (S0)        │
      │                     │  GPT-4o deployment         │
      │                     │                            │
      │                     │  Azure AI Foundry          │
      │                     │  Hub + Project             │
      │                     └────────────────────────────┘
      │
      │                     ┌───────── PUBLIC ──────────┐
      └───MCP protocol───▶ │  learn.microsoft.com/     │
                            │  api/mcp (no auth)        │
                            └───────────────────────────┘
```

---

## Workshop Milestones

| # | Milestone | Description |
|---|---|---|
| 1 | [Prerequisites](docs/1-prerequisites.md) | Verify Azure access, CLI tools, quotas, and environment setup |
| 2 | [Deploy Infrastructure](docs/2-deploy-infrastructure.md) | Provision all 10 Azure resources with a single script |
| 3 | [Configure SQL Server](docs/3-configure-sql.md) | Install SQL Express, create the InsuranceDB, verify log flow |
| 4 | [Simulate Errors](docs/4-simulate-errors.md) | Generate realistic SQL errors and deadlocks for testing |
| 5 | [Deploy the AI App](docs/5-deploy-ai-app.md) | Deploy the Streamlit application and verify end-to-end |
| 6 | [Production & Cleanup](docs/6-production-cleanup.md) | Harden for production or tear down resources |

---

## Who Should Take This Workshop

| Role | Focus Area |
|---|---|
| **Platform / Cloud Engineer** | Run `deploy.sh` to provision infrastructure |
| **DevOps Engineer** | Set up CI/CD for the Streamlit app |
| **Data / AI Engineer** | Tune KQL prompts in `kql_prompt.py` |
| **Security Engineer** | Review RBAC assignments and plan Private Link |

---

## Pre-Learning

- [Azure Monitor Agent overview](https://learn.microsoft.com/azure/azure-monitor/agents/azure-monitor-agent-overview)
- [Azure OpenAI Service documentation](https://learn.microsoft.com/azure/ai-services/openai/)
- [Managed identities for Azure resources](https://learn.microsoft.com/entra/identity/managed-identities-azure-resources/overview)
- [KQL quick reference](https://learn.microsoft.com/azure/data-explorer/kusto/query/kql-quick-reference)

---

> **Ready?** Start with [Milestone 1: Prerequisites →](docs/1-prerequisites.md)
