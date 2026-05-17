---
title: "3. Configure SQL Server"
layout: default
nav_order: 3
parent: Home
---

# Milestone 3: Configure SQL Server

Install SQL Server Express on the VM and create the demo database.

---

## Install SQL Server Express

```bash
bash scripts/deploy-sql-express.sh
```

This runs remotely on the VM via `az vm run-command` and:
1. Downloads SQL Server 2022 Express silently
2. Configures TCP/IP protocol on port 1433
3. Opens Windows Firewall for SQL Server
4. Enables SQL Server Browser service
5. Verifies the installation

---

## Create the Insurance Database

The script `scripts/create-insurance-db.sql` creates the **InsuranceDB** with tables:
- `PolicyHolders` — Customer policy records
- `Claims` — Insurance claims processing
- `Premiums` — Premium calculation history

---

## Verify Log Ingestion

After deploying AMA and the DCR, allow **10–15 minutes** for the first logs to appear.

```bash
# Get the workspace ID
LAW_WORKSPACE_ID=$(az monitor log-analytics workspace show \
  --resource-group "$RESOURCE_GROUP" \
  --workspace-name "$LAW_NAME" \
  --query customerId -o tsv)

# Check if Event table has data
az monitor log-analytics query \
  --workspace "$LAW_WORKSPACE_ID" \
  --analytics-query "Event | summarize count() by EventLevelName" \
  --timespan PT1H \
  -o table
```

---

## Run Health Check

```bash
bash scripts/check-vm-sql.sh
```

| # | Check | What it verifies |
|---|---|---|
| 1 | VM power state | VM is running |
| 2 | VM agent status | Guest agent is ready |
| 3 | SQL Server service | `MSSQL$SQLEXPRESS` is running |
| 4 | TCP port 1433 | SQL Server is listening |
| 5 | AMA extension | Azure Monitor Agent is provisioned |

---

> **Next:** [Milestone 4: Simulate Errors →](4-simulate-errors.md)
