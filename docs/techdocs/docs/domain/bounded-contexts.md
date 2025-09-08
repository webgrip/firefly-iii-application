---
title: "Bounded Contexts"
description: "Logical boundaries inferred from Firefly III features/modules."
tags: [ddd, contexts, firefly-iii, domain]
search: { boost: 3, exclude: false }
icon: material/vector-square
author: "Infrastructure/Platform Team"
date: 2025-01-09
---

**Purpose:** Identify contexts to avoid model bleed.

**Contents**
- [Contexts](#contexts)
- [Shared Kernel / ACL](#shared-kernel--acl)
- [Sources](#sources)

### Contexts

| Context | Responsibility | Key Models/Terms | Upstream Artifact | Source |
|--------|-----------------|------------------|-------------------|--------|
| Account Management | Account lifecycle, types, balances | Asset Account, Liability Account, Account Balance | Account management pages/API | "Accounts Documentation" — https://docs.firefly-iii.org/concepts/accounts — retrieved 2025-01-09 |
| Transaction Processing | Recording, categorizing, splitting money movements | Transaction, Transaction Journal, Transaction Split | Transaction management system | "Transactions Documentation" — https://docs.firefly-iii.org/concepts/transactions — retrieved 2025-01-09 |
| Budget Planning | Financial planning, spending limits, tracking | Budget, Budget Limit, Available Budget | Budget management module | "Budgets Documentation" — https://docs.firefly-iii.org/concepts/budgets — retrieved 2025-01-09 |
| Bill Management | Recurring expense tracking and prediction | Bill, Bill Match, Expected Bill | Bill tracking system | "Bills Documentation" — https://docs.firefly-iii.org/concepts/bills — retrieved 2025-01-09 |
| Automation Engine | Rule-based transaction processing | Rule, Rule Group, Rule Action, Rule Trigger | Rules engine | "Rules Documentation" — https://docs.firefly-iii.org/concepts/rules — retrieved 2025-01-09 |
| Savings Goals | Goal-oriented savings tracking | Piggy Bank, Piggy Bank Event | Piggy bank functionality | "Piggy Banks Documentation" — https://docs.firefly-iii.org/concepts/piggy-banks — retrieved 2025-01-09 |
| Currency Exchange | Multi-currency support and conversion | Currency, Exchange Rate, Currency Conversion | Currency management | "Currency Documentation" — https://docs.firefly-iii.org/concepts/currencies — retrieved 2025-01-09 |
| Reporting Analytics | Financial reporting and analysis | Report, Chart, Balance History | Reporting system | "Reports Documentation" — https://docs.firefly-iii.org/how-to/firefly-iii/features/reports — retrieved 2025-01-09 |
| Data Import | External data ingestion and mapping | Import, Import Mapping, CSV Configuration | Import functionality | "Data Import Documentation" — https://docs.firefly-iii.org/how-to/firefly-iii/importing-data — retrieved 2025-01-09 |
| User Management | Authentication, authorization, preferences | User, UserRole, Preference | User system | "User Management" — https://docs.firefly-iii.org/how-to/firefly-iii/features/organize — retrieved 2025-01-09 |

### Shared Kernel / ACL

**Shared Kernel Components:**
- **Money Value Object**: Shared across all financial contexts for amount representation
- **Currency Domain**: Referenced by Transaction Processing, Budget Planning, and Currency Exchange contexts
- **Date/Time Handling**: Consistent temporal operations across all contexts
- **User Context**: Authentication and authorization shared across all bounded contexts

**Anti-Corruption Layer (ACL):**
- **Infrastructure Packaging ACL**: Translates between Firefly III domain concepts and our containerized deployment model
- **Database Schema ACL**: Maps domain models to Laravel Eloquent ORM representations
- **API Response ACL**: Transforms internal domain objects to REST API JSON representations
- **Import Format ACL**: Converts external bank/CSV formats to internal transaction models

**Context Integration Patterns:**
- **Transaction → Budget**: Transactions automatically update budget spending via domain events
- **Bill → Transaction**: Bill matching creates transaction associations through rule engine
- **Account → Report**: Reporting context aggregates data from account and transaction contexts
- **Rule → Transaction**: Automation engine processes transactions through event subscriptions

### Sources
- "Firefly III Architecture Overview" — https://docs.firefly-iii.org/explanation/more-information/architecture — retrieved 2025-01-09
- "Firefly III API Documentation" — https://docs.firefly-iii.org/firefly-iii/api — retrieved 2025-01-09
- "Firefly III Features Overview" — https://docs.firefly-iii.org/how-to/firefly-iii/features — retrieved 2025-01-09

<!-- ai-docs-metadata
{"last_audit":"2025-01-09","fingerprints":{"sources":{"https://docs.firefly-iii.org/explanation/more-information/architecture":"sha256:pending","https://docs.firefly-iii.org/firefly-iii/api":"sha256:pending","https://docs.firefly-iii.org/how-to/firefly-iii/features":"sha256:pending"},"sections":{"bounded-contexts":"sha256:pending"}}}
-->