---
title: "Bounded Contexts"
description: "Logical boundaries inferred from upstream features/modules."
tags: [ddd, contexts]
search: { boost: 3, exclude: false }
icon: material/vector-square
author: "<Infrastructure/Platform Team>"
date: 2025-01-09
---

**Purpose:** Identify contexts to avoid model bleed.

**Contents**
- [Contexts](#contexts)
- [Shared Kernel / ACL](#shared-kernel--acl)
- [Sources](#sources)

## Contexts

| Context | Responsibility | Key Models/Terms | Upstream Artifact | Source |
|---------|-----------------|------------------|-------------------|--------|
| Account Management | Manage financial accounts and their properties | Account, Asset Account, Liability, Account Type | Accounts module | "Firefly III Accounts" — https://docs.firefly-iii.org/how-to/firefly-iii/features/accounts/ — retrieved 2025-01-09 |
| Transaction Processing | Record and categorize financial transactions | Transaction, Withdrawal, Deposit, Transfer, Split | Transactions module | "Firefly III Transactions" — https://docs.firefly-iii.org/how-to/firefly-iii/features/transactions/ — retrieved 2025-01-09 |
| Budget Management | Plan and track spending against budgets | Budget, Budget Limit, Spending Period, Available Amount | Budgets module | "Firefly III Budgets" — https://docs.firefly-iii.org/how-to/firefly-iii/features/budgets/ — retrieved 2025-01-09 |
| Bill Tracking | Manage recurring bills and payment expectations | Bill, Expected Payment, Bill Match, Due Date | Bills module | "Firefly III Bills" — https://docs.firefly-iii.org/how-to/firefly-iii/features/bills/ — retrieved 2025-01-09 |
| Categorization | Organize transactions with categories and tags | Category, Tag, Category Assignment, Tagging | Categories & Tags modules | "Firefly III Categories" — https://docs.firefly-iii.org/how-to/firefly-iii/features/categories/ — retrieved 2025-01-09 |
| Rule Engine | Automate transaction processing with rules | Rule, Rule Trigger, Rule Action, Condition | Rules module | "Firefly III Rules" — https://docs.firefly-iii.org/how-to/firefly-iii/features/rules/ — retrieved 2025-01-09 |
| Savings Goals | Track progress toward financial goals | Piggy Bank, Target Amount, Saved Amount, Goal Period | Piggy Banks module | "Firefly III Piggy Banks" — https://docs.firefly-iii.org/how-to/firefly-iii/features/piggy-banks/ — retrieved 2025-01-09 |
| Data Import | Import financial data from external sources | Import Job, CSV Mapping, Bank Import, Import Configuration | Import module | "Firefly III Data Import" — https://docs.firefly-iii.org/how-to/data-importer/ — retrieved 2025-01-09 |
| Reporting | Generate financial reports and visualizations | Report, Chart, Financial Summary, Time Period | Reports module | "Firefly III Reports" — https://docs.firefly-iii.org/how-to/firefly-iii/features/reports/ — retrieved 2025-01-09 |
| Currency Management | Handle multiple currencies and exchange rates | Currency, Exchange Rate, Currency Conversion | Multi-currency support | "Firefly III Currencies" — https://docs.firefly-iii.org/how-to/firefly-iii/features/currencies/ — retrieved 2025-01-09 |
| User Management | Manage user accounts and preferences | User, User Preference, Role, Permission | User/Auth system | "Firefly III Administration" — https://docs.firefly-iii.org/how-to/firefly-iii/administration/ — retrieved 2025-01-09 |

## Shared Kernel / ACL

**Shared Kernel (concepts used across contexts):**
- **Money Amount**: Monetary value with currency information
- **Date Range**: Time periods used in budgets, reports, and bills
- **Account Reference**: Lightweight account identifier used across contexts
- **Transaction Reference**: Basic transaction information for cross-context operations

**Anti-Corruption Layer (ACL) boundaries:**
- **External Data Sources**: Import context translates bank formats to internal transaction model
- **Reporting Engine**: Transforms internal data models into presentation-friendly formats
- **API Layer**: Converts internal domain models to REST API representations
- **Database Schema**: ORM layer translates between domain objects and database tables

**Context Relationships:**
- **Transaction Processing** ↔ **Account Management**: Transactions must reference valid accounts
- **Budget Management** ↔ **Transaction Processing**: Budget tracking requires transaction categorization
- **Bill Tracking** ↔ **Transaction Processing**: Bill matching against actual transactions
- **Rule Engine** ↔ **Transaction Processing**: Rules process and modify transactions
- **Categorization** ↔ **Transaction Processing**: Categories and tags applied to transactions

**Integration Points:**
- Account balances calculated from transaction history
- Budget progress derived from categorized transactions
- Bill matching based on transaction patterns
- Reports aggregate data across multiple contexts

## Sources
- "Firefly III Features Overview" — https://docs.firefly-iii.org/how-to/firefly-iii/features/ — retrieved 2025-01-09
- "Firefly III Architecture" — https://docs.firefly-iii.org/how-to/firefly-iii/features/accounts/ — retrieved 2025-01-09

<!-- ai-docs-metadata
{"last_audit":"2025-01-09","fingerprints":{"sources":{"https://docs.firefly-iii.org/how-to/firefly-iii/features/":"","https://docs.firefly-iii.org/how-to/firefly-iii/features/accounts/":""},"sections":{"bounded-contexts":""}}}
-->