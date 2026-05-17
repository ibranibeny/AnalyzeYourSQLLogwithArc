---
title: Architecture
nav_order: 2
description: "System architecture, resource topology, and data flows"
permalink: /architecture/
---

# Architecture
{: .fs-9 }

Understand the system design, Azure resource relationships, and data flows.
{: .fs-6 .fw-300 }

---

## Resource Topology

```mermaid
graph TB
    %% ─── SOUTHEAST ASIA REGION ───────────────────────────────────
    subgraph SEA["Southeast Asia"]
        direction TB

        subgraph Compute["Compute"]
            VM["Azure VM<br/>Standard_D2s_v3<br/>Windows Server 2022"]
            SQL["SQL Server Express 2022<br/>TCP 1433<br/>InsuranceDB"]
        end

        subgraph Monitoring["Monitoring & Collection"]
            AMA["Azure Monitor Agent<br/>VM Extension"]
            DCE["Data Collection Endpoint"]
            DCR["Data Collection Rule<br/>Windows Event Log + Perf Counters"]
            LAW["Log Analytics Workspace<br/>PerGB2018 · 30-day retention<br/>Event table + Perf table"]
        end

        subgraph AppLayer["Application Layer"]
            ASP["App Service Plan<br/>B1 Linux"]
            APP["Web App (Streamlit)<br/>'Talk to Your SQL Logs'<br/>System-assigned MI"]
        end
    end

    %% ─── EAST US REGION ──────────────────────────────────────────
    subgraph EUS["East US"]
        direction TB

        subgraph AIServices["AI Services"]
            AOAI["Azure OpenAI<br/>S0 Tier<br/>GPT-4o (30K TPM)"]
            AIHUB["AI Foundry Hub"]
            AIPROJ["AI Foundry Project<br/>Credential-less Connection"]
        end
    end

    %% ─── EXTERNAL ────────────────────────────────────────────────
    subgraph EXT["External (Public)"]
        MCP["Microsoft Learn MCP<br/>learn.microsoft.com/api/mcp<br/>No authentication"]
    end

    %% ─── CONNECTIONS ─────────────────────────────────────────────
    VM --> SQL
    SQL -->|"Error logs &<br/>Event Log"| AMA
    AMA -->|"Stream via"| DCE
    DCE -->|"Transform with"| DCR
    DCR -->|"Ingest into"| LAW

    ASP --> APP
    APP ==>|"KQL queries<br/>azure-monitor-query SDK<br/>(Managed Identity)"| LAW
    APP ==>|"Chat completions<br/>(Managed Identity)"| AOAI
    APP -.->|"MCP protocol<br/>Documentation search"| MCP

    AIHUB --> AIPROJ
    AIPROJ -->|"Credential-less<br/>connection"| AOAI

    %% ─── STYLES ──────────────────────────────────────────────────
    classDef compute fill:#e1f5fe,stroke:#0277bd
    classDef monitor fill:#fff3e0,stroke:#ef6c00
    classDef app fill:#e8f5e9,stroke:#2e7d32
    classDef ai fill:#f3e5f5,stroke:#6a1b9a
    classDef ext fill:#fafafa,stroke:#616161

    class VM,SQL compute
    class AMA,DCE,DCR,LAW monitor
    class ASP,APP app
    class AOAI,AIHUB,AIPROJ ai
    class MCP ext
```

---

## Resource Inventory

| # | Resource | Type | Region | Key Config |
|---|---|---|---|---|
| 1 | `rg-contoso-sqlobs` | Resource Group | Southeast Asia | — |
| 2 | `law-contoso-sqlobs` | Log Analytics Workspace | Southeast Asia | PerGB2018, 30-day retention |
| 3 | `vm-sql-sea-01` | Virtual Machine | Southeast Asia | D2s_v3, Windows 2022, System MI |
| 4 | Data Collection Endpoint | DCE | Southeast Asia | Linked to VM |
| 5 | `dcr-sql-windows-logs` | Data Collection Rule | Southeast Asia | Event Log + Perf Counters |
| 6 | DCR Association | Association | Southeast Asia | VM → DCR |
| 7 | `aoai-contoso-sqllogs` | Azure OpenAI | East US | S0, GPT-4o 30K TPM |
| 8 | `ai-hub-contoso` | AI Foundry Hub | East US | — |
| 9 | `ai-proj-sqllogs` | AI Foundry Project | East US | Credential-less OpenAI connection |
| 10 | `app-contoso-sqllogs` | App Service (Web App) | Southeast Asia | B1 Linux, System MI |

---

## Data Flows

### Log Collection Flow

```mermaid
sequenceDiagram
    participant SQL as SQL Server Express
    participant EVT as Windows Event Log
    participant AMA as Azure Monitor Agent
    participant DCR as Data Collection Rule
    participant LAW as Log Analytics

    SQL->>EVT: Write error/event (EventID 911, 208, 17063, 18456)
    SQL->>EVT: Write performance counters
    AMA->>EVT: Poll events (XPath query)
    AMA->>DCR: Apply transformation rules
    DCR->>LAW: Ingest into Event & Perf tables
```

### Query Flow (Runtime)

```mermaid
sequenceDiagram
    participant User
    participant App as Streamlit App
    participant GPT as Azure OpenAI (GPT-4o)
    participant LAW as Log Analytics
    participant MCP as Microsoft Learn MCP

    User->>App: "Why did errors spike yesterday?"
    App->>GPT: Generate KQL from natural language
    GPT-->>App: KQL query
    App->>LAW: Execute KQL (Managed Identity)
    LAW-->>App: Query results
    App->>GPT: Summarize results + context
    App->>MCP: Search related documentation
    MCP-->>App: Documentation snippets
    GPT-->>App: Natural language answer + docs
    App-->>User: Display answer with references
```

---

## Security Model

| Principle | Implementation |
|---|---|
| No secrets in app settings | Managed Identity replaces API keys |
| Least privilege RBAC | `Cognitive Services OpenAI User` + `Log Analytics Reader` |
| System-assigned identity | Lifecycle tied to App Service |
| No credentials in code | `DefaultAzureCredential` for all tokens |
| HTTPS enforced | App Service TLS by default |

### RBAC Assignments

| Principal | Role | Scope |
|---|---|---|
| Web App MI | Cognitive Services OpenAI User | Azure OpenAI account |
| Web App MI | Log Analytics Reader | Log Analytics Workspace |

---

## Network Architecture

```mermaid
graph LR
    subgraph PublicInternet["Public Internet"]
        Users["Workshop Users"]
        Learn["learn.microsoft.com"]
    end

    subgraph Azure["Azure (Southeast Asia)"]
        NSG["Network Security Group<br/>Allow: 3389 (RDP)<br/>Allow: 443 (HTTPS)"]
        VM["VM + SQL Server<br/>10.0.0.0/24"]
        AppSvc["App Service<br/>*.azurewebsites.net"]
    end

    Users -->|"HTTPS"| AppSvc
    AppSvc -->|"Managed Identity"| AOAI["Azure OpenAI<br/>(East US)"]
    AppSvc -->|"Managed Identity"| LAW["Log Analytics"]
    AppSvc -.->|"HTTPS"| Learn
    NSG --> VM
```

---

## Infrastructure Scripts

The deployment is orchestrated by `deploy.sh` which provisions resources in order:

| Step | Script Section | Resources |
|---|---|---|
| 1 | `[1/7]` | Resource Group |
| 2 | `[2/7]` | Log Analytics Workspace |
| 3 | `[3/7]` | Windows VM + System MI |
| 4 | `[4/7]` | AMA + DCE + DCR + Association |
| 5 | `[5/7]` | Azure OpenAI + GPT-4o deployment |
| 6 | `[6/7]` | AI Foundry Hub + Project + Connection |
| 7 | `[7/7]` | App Service Plan + Web App + MI + RBAC |

{: .important }
> Steps 5–6 deploy to **East US** (Azure OpenAI availability). All other resources are in **Southeast Asia**.

---

[← Prerequisites]({{ site.baseurl }}{% link modules/01-prerequisites.md %}){: .btn .btn-outline .fs-5 .mb-4 .mb-md-0 .mr-2 }
[Next: Deploy Infrastructure →]({{ site.baseurl }}{% link modules/03-deploy-infrastructure.md %}){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 }
