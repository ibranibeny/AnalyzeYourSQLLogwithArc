---
layout: home
title: Home
nav_order: 1
description: "AI-Powered SQL Server Log Analysis with Azure Monitor, GPT-4o & Microsoft Learn MCP"
permalink: /
---

# Analyze Your SQL Logs with Arc & AI — Workshop
{: .fs-9 }

Deploy an **AI-powered SQL Server log analysis** solution using Azure Monitor Agent, GPT-4o, and Microsoft Learn MCP — with zero secrets and fully managed identity.
{: .fs-6 .fw-300 }

[Get Started]({{ site.baseurl }}{% link modules/01-prerequisites.md %}){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }
[View Architecture]({{ site.baseurl }}{% link architecture.md %}){: .btn .fs-5 .mb-4 .mb-md-0 }

---

## What You'll Build

A complete AI log analysis system that:

- **Centralises SQL Server logs** — AMA + DCR streams events to Log Analytics
- **Enables natural language queries** — Ask questions like *"Why did errors spike yesterday?"*
- **References official docs** — Microsoft Learn MCP provides documentation context
- **Eliminates secrets** — Managed Identity for all service-to-service auth

---

## Architecture at a Glance

| Component | Technology | Azure Resource |
|---|---|---|
| 🖥️ SQL Server | SQL Express 2022 | VM — `Standard_D2s_v3` |
| 📊 Log Collection | Azure Monitor Agent + DCR | Log Analytics Workspace |
| 🤖 AI Engine | GPT-4o | Azure OpenAI (S0) |
| 🧠 AI Platform | Azure AI Foundry | Hub + Project |
| 🌐 Web App | Streamlit | App Service (B1 Linux) |
| 📚 Documentation | Microsoft Learn MCP | Public API (no auth) |

---

## Workshop Modules

| # | Module | Duration | Description |
|---|---|---|---|
| 1 | [Prerequisites]({{ site.baseurl }}{% link modules/01-prerequisites.md %}) | 15 min | Verify Azure access and tooling |
| 2 | [Architecture]({{ site.baseurl }}{% link architecture.md %}) | 10 min | Understand the system design |
| 3 | [Deploy Infrastructure]({{ site.baseurl }}{% link modules/03-deploy-infrastructure.md %}) | 25 min | Provision all Azure resources |
| 4 | [Configure SQL Server]({{ site.baseurl }}{% link modules/04-configure-sql.md %}) | 15 min | Install SQL Express & create database |
| 5 | [Simulate Errors]({{ site.baseurl }}{% link modules/05-simulate-errors.md %}) | 15 min | Generate test events |
| 6 | [Deploy the AI App]({{ site.baseurl }}{% link modules/06-deploy-ai-app.md %}) | 20 min | Deploy Streamlit & verify end-to-end |
| 7 | [Production & Cleanup]({{ site.baseurl }}{% link modules/07-production-cleanup.md %}) | 10 min | Harden or tear down |

**Total estimated time: ~2 hours**
{: .fs-5 .fw-300 }

---

## Key Design Decisions

- **Azure CLI over Bicep/Terraform** — lower barrier for workshop attendees
- **Managed Identity everywhere** — zero secrets in scripts or app settings
- **GPT-4o for KQL generation** — natural language to Kusto queries
- **Microsoft Learn MCP** — enrich answers with official documentation
- **Single deploy script** — all 10 resources in one `deploy.sh` run

---

{: .note }
> This workshop targets **`southeastasia`** and **`eastus`** regions. Verify Azure OpenAI GPT-4o quota availability before starting.
