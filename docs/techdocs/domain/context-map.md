---
title: "Context Map"
description: "Relationships among contexts; upstream and packaging."
tags: [ddd, mapping]
search: { boost: 2, exclude: false }
icon: material/map
author: "<Infrastructure/Platform Team>"
date: 2025-01-09
---

**Purpose:** Visualize dependencies and translation boundaries.

**Contents**
- [Map](#map)
- [Notes](#notes)
- [Sources](#sources)

## Map

```mermaid
flowchart LR
  AC[Account Management] 
  TX[Transaction Processing]
  BG[Budget Management]
  BL[Bill Tracking]
  CT[Categorization]
  RE[Rule Engine]
  SG[Savings Goals]
  DI[Data Import]
  RP[Reporting]
  CU[Currency Management]
  UM[User Management]
  
  %% Core relationships
  TX --> AC : validates accounts
  TX --> CT : applies categories/tags
  TX --> CU : handles currencies
  
  %% Processing flows
  RE --> TX : modifies transactions
  BG --> TX : tracks spending
  BL --> TX : matches bills
  
  %% Derived data
  RP --> TX : aggregates data
  RP --> AC : account summaries
  RP --> BG : budget reports
  
  %% Import/Export
  DI --> TX : creates transactions
  DI --> AC : validates accounts
  
  %% User context
  UM --> AC : owns accounts
  UM --> TX : user transactions
  UM --> BG : user budgets
  
  %% Goals tracking
  SG --> AC : asset accounts only
  SG --> TX : tracks contributions
```

## Notes

**Upstream Integration Points:**
- **Account Management ↔ Transaction Processing**: All transactions must reference valid accounts; account balances are derived from transaction history
- **Rule Engine → Transaction Processing**: Rules automatically categorize and modify transactions based on patterns
- **Budget Management ← Transaction Processing**: Budget progress calculated from categorized transactions
- **Bill Tracking ← Transaction Processing**: Bills are matched against transaction patterns using amount and description
- **Reporting ← Multiple Contexts**: Reports aggregate data from transactions, accounts, budgets, and bills

**Anti-Corruption Layer (ACL) Boundaries:**
- **Data Import Context**: Translates external bank formats (CSV, OFX, MT940) into internal transaction models
- **Reporting Context**: Transforms domain models into chart-friendly data structures
- **API Layer**: Converts internal aggregates to RESTful JSON representations
- **Database Layer**: Laravel ORM provides abstraction between domain objects and database schema

**Shared Kernel Components:**
- **Money Value Object**: Used across all contexts for monetary amounts with currency
- **Date Range Value Object**: Common time period representation for budgets and reports
- **User Identity**: Consistent user reference across all user-owned aggregates

**Context Dependencies:**
- **High Coupling**: Transaction Processing is central to most other contexts
- **Medium Coupling**: Account Management provides foundational account references
- **Low Coupling**: Reporting and Data Import are mostly one-way dependencies
- **Isolated**: User Management has minimal coupling except for ownership

**Translation Responsibilities:**
- Import Context handles all external format variations
- Reporting Context manages presentation layer transformations
- Each context maintains its own validation rules and business logic
- Cross-context communication happens through domain events where possible

## Sources
- "Firefly III Architecture Overview" — https://docs.firefly-iii.org/how-to/firefly-iii/features/ — retrieved 2025-01-09
- "Firefly III Data Flow" — https://docs.firefly-iii.org/how-to/firefly-iii/features/transactions/ — retrieved 2025-01-09

<!-- ai-docs-metadata
{"last_audit":"2025-01-09","fingerprints":{"sources":{"https://docs.firefly-iii.org/how-to/firefly-iii/features/":"","https://docs.firefly-iii.org/how-to/firefly-iii/features/transactions/":""},"sections":{"context-map":""}}}
-->