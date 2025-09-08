---
title: "Aggregates & Invariants"
description: "Aggregate roots, consistency rules, and transactional boundaries as documented by upstream."
tags: [ddd, aggregates, firefly-iii, domain]
search: { boost: 3, exclude: false }
icon: material/source-branch
author: "Infrastructure/Platform Team"
date: 2025-01-09
---

**Purpose:** Make invariants explicit for Firefly III domain model.

**Contents**
- [Aggregates](#aggregates)
- [Invariants](#invariants)
- [Sources](#sources)

### Aggregates

| Aggregate Root | Entities/Value Objects | Invariants | Source |
|----------------|------------------------|-----------|--------|
| Account | Account, Balance, AccountMeta | Account balance must match sum of related transactions | "Account Management" — https://docs.firefly-iii.org/concepts/accounts — retrieved 2025-01-09 |
| TransactionJournal | Transaction, TransactionSplit, Attachment | Journal must balance to zero (double-entry bookkeeping) | "Transaction Journals" — https://docs.firefly-iii.org/concepts/transactions — retrieved 2025-01-09 |
| Budget | BudgetLimit, BudgetSpent, AutoBudget | Budget spending cannot exceed available amount unless explicitly allowed | "Budget Management" — https://docs.firefly-iii.org/concepts/budgets — retrieved 2025-01-09 |
| Bill | BillMatch, BillPayment | Bill amounts must match within configured tolerance range | "Bill Tracking" — https://docs.firefly-iii.org/concepts/bills — retrieved 2025-01-09 |
| Rule | RuleAction, RuleTrigger | Rule execution must be deterministic and not create cycles | "Rules Engine" — https://docs.firefly-iii.org/concepts/rules — retrieved 2025-01-09 |
| User | Preference, UserRole, Attachment | User can only access their own financial data (multi-tenancy) | "User Isolation" — https://docs.firefly-iii.org/explanation/more-information/architecture — retrieved 2025-01-09 |
| PiggyBank | PiggyBankEvent, PiggyBankRepetition | Saved amount cannot exceed target amount | "Piggy Banks" — https://docs.firefly-iii.org/concepts/piggy-banks — retrieved 2025-01-09 |

### Invariants

**Financial Integrity:**
- **Double-Entry Principle**: Every transaction journal must have balanced debits and credits (sum = 0)
- **Account Balance Consistency**: Account balances must always reflect the sum of all related transactions
- **Currency Consistency**: Transactions within a journal must use compatible currencies
- **Date Ordering**: Future transactions are allowed but require explicit user confirmation

**Business Rules:**
- **Budget Limits**: Cannot spend more than available budget unless auto-budget is enabled
- **Bill Matching**: Bills can only match transactions within the configured amount tolerance (e.g., ±10%)
- **Asset Account Minimums**: Some asset accounts may have minimum balance requirements
- **User Data Isolation**: Users can only access accounts, transactions, and budgets they own

**Data Integrity:**
- **Account Types**: Asset accounts cannot be converted to expense accounts (business rule preservation)
- **Transaction Splits**: Split transactions must have the same date and description
- **Recurring Transactions**: Must have valid next occurrence dates and proper template data
- **Category Assignments**: Transfers between asset accounts cannot have categories (business rule)

**Automation Rules:**
- **Rule Execution Order**: Rules within a group execute in sequence, not parallel
- **Rule Cycles**: Rules cannot create infinite loops or recursive triggers
- **Rule Scope**: Rules only apply to transactions matching their trigger conditions
- **Rule Audit**: All rule executions must be logged for transparency

**Multi-Currency Rules:**
- **Exchange Rates**: Must be positive values and properly dated
- **Currency Precision**: Amounts stored with appropriate decimal precision per currency
- **Conversion Tracking**: Foreign currency transactions maintain original amounts
- **Base Currency**: Each user must have a designated base currency

### Sources
- "Firefly III Data Model" — https://docs.firefly-iii.org/explanation/more-information/architecture — retrieved 2025-01-09
- "Double-Entry Bookkeeping" — https://docs.firefly-iii.org/explanation/financial-concepts/double-entry — retrieved 2025-01-09
- "Transaction Validation" — https://docs.firefly-iii.org/references/firefly-iii/api/validation — retrieved 2025-01-09

<!-- ai-docs-metadata
{"last_audit":"2025-01-09","fingerprints":{"sources":{"https://docs.firefly-iii.org/explanation/more-information/architecture":"sha256:pending","https://docs.firefly-iii.org/explanation/financial-concepts/double-entry":"sha256:pending","https://docs.firefly-iii.org/references/firefly-iii/api/validation":"sha256:pending"},"sections":{"aggregates":"sha256:pending"}}}
-->