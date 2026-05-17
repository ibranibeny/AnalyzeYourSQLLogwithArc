---
title: "4. Simulate Errors"
layout: default
nav_order: 4
parent: Home
---

# Milestone 4: Simulate Errors

Generate realistic SQL Server error events to test the AI log analysis application.

---

## Quick Start — Run Everything

```bash
bash scripts/run-all-simulations.sh
```

This runs all phases:
1. **Phase 1:** 19 SQL error events (`simulate-errors.sh`)
2. **Phase 2:** 9 deadlock + slow query events (`simulate-deadlock-slowquery.sh`)
3. **Phase 3:** Waits 30s then queries Log Analytics for verification

**Total events:** ~28 | **Duration:** ~10–15 minutes

---

## Phase 1: SQL Error Events

```bash
bash scripts/simulate-errors.sh
```

### Error Categories

| Category | Events | EventID |
|---|---|---|
| Non-existent database access | 3 | 911 |
| Non-existent table queries | 2 | 208 |
| Business-context RAISERROR WITH LOG | 8 | 17063 |
| Failed login attempts | 5 | 18456 |
| Backup failure | 1 | 3201 |

### Business-Context Errors

| Error Message | Scenario |
|---|---|
| Database NonExistentDB does not exist. Cannot process claim batch job. | Missing database |
| Transaction deadlock detected on table PolicyHolders. | Deadlock |
| Timeout expired connecting to database ClaimsDB. | Connection timeout |
| Cannot open database RequestedByLogin. Login failed for user sa. | Login failure |
| Disk space critically low on drive E:. | Disk space |
| Premium calculation service timeout. sp_CalculatePremium exceeded 30s. | Stored proc timeout |
| Duplicate NRIC detected in PolicyHolders table. | Constraint violation |
| Transaction log full for database InsuranceDB. | Transaction log full |

---

## Phase 2: Deadlocks & Slow Queries

```bash
bash scripts/simulate-deadlock-slowquery.sh
```

Uses `scripts/simulate-deadlock-slowquery.sql` to create:
- Artificial deadlock scenarios between concurrent transactions
- Long-running queries that exceed threshold (simulating slow queries)

---

## AI Test Questions

After waiting 5–10 minutes for log ingestion, test with:

| Question | Tests |
|---|---|
| *"What errors happened in the last 30 minutes?"* | General error retrieval |
| *"Are there any database not found errors?"* | EventID 911 filtering |
| *"Were there any deadlock or timeout errors?"* | Pattern matching |
| *"Show me all failed login attempts today"* | Security event analysis |
| *"Is anyone trying to brute force SQL Server?"* | Threat detection |
| *"Show me all disk space related warnings"* | Infrastructure monitoring |

---

> **Next:** [Milestone 5: Deploy the AI App →](5-deploy-ai-app.md)
