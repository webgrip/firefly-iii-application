---
title: "Aggregates & Invariants"
description: "Aggregate roots, consistency rules, and transactional boundaries as documented by upstream."
tags: [ddd, aggregates]
search: { boost: 3, exclude: false }
icon: material/source-branch
author: "<Infrastructure/Platform Team>"
date: 2025-01-09
---

**Purpose:** Make invariants explicit.

**Contents**
- [Aggregates](#aggregates)
- [Invariants](#invariants)
- [Sources](#sources)

## Aggregates

| Aggregate Root | Entities/Value Objects | Invariants | Source |
|----------------|------------------------|-----------|--------|
| Account | Account Type, Account Balance, Account Meta | Account balance must reflect sum of all transactions; Account type cannot change once set | "Firefly III Accounts" — https://docs.firefly-iii.org/how-to/firefly-iii/features/accounts/ — retrieved 2025-01-09 |
| Transaction Group | Transaction, Transaction Journal, Source/Destination Accounts | Transaction amounts must balance (double-entry); All transactions in group occur on same date | "Firefly III Transactions" — https://docs.firefly-iii.org/how-to/firefly-iii/features/transactions/ — retrieved 2025-01-09 |
| Budget | Budget Limit, Budget Period, Available Amount | Budget limits cannot be negative; Spent amount cannot exceed available amount (warning only) | "Firefly III Budgets" — https://docs.firefly-iii.org/how-to/firefly-iii/features/budgets/ — retrieved 2025-01-09 |
| Bill | Bill Match, Expected Amount, Due Date, Payment History | Bill amount must be positive; Due dates must be in future when created | "Firefly III Bills" — https://docs.firefly-iii.org/how-to/firefly-iii/features/bills/ — retrieved 2025-01-09 |
| Rule | Rule Trigger, Rule Action, Rule Condition | Rule must have at least one trigger and one action; Actions must be valid for trigger type | "Firefly III Rules" — https://docs.firefly-iii.org/how-to/firefly-iii/features/rules/ — retrieved 2025-01-09 |
| Piggy Bank | Target Amount, Current Amount, Savings Goal, Account Reference | Target amount must be positive; Saved amount cannot exceed target; Must reference valid asset account | "Firefly III Piggy Banks" — https://docs.firefly-iii.org/how-to/firefly-iii/features/piggy-banks/ — retrieved 2025-01-09 |
| User | User Preferences, User Role, Account Ownership | User must own all accounts they access; Preferences must be valid key-value pairs | "Firefly III Administration" — https://docs.firefly-iii.org/how-to/firefly-iii/administration/ — retrieved 2025-01-09 |

## Invariants

**Account Aggregate Invariants:**
- Account balance must always equal the sum of all associated transaction amounts
- Account type (asset, expense, revenue, liability) cannot be changed once set
- Account currency cannot be changed if transactions exist
- Account must have a unique name within account type for the user

**Transaction Group Aggregate Invariants:**
- All transactions within a group must occur on the same date
- Total debits must equal total credits (double-entry bookkeeping)
- Source and destination accounts must be different for each transaction
- Transaction amounts must be positive values
- Split transactions must reference the same source/destination accounts

**Budget Aggregate Invariants:**
- Budget amount must be positive or zero
- Budget period dates must be valid (start before end)
- Only one active budget per category per period
- Spent amount calculation is derived, not stored directly

**Bill Aggregate Invariants:**
- Bill minimum and maximum amounts must be positive
- Maximum amount must be greater than or equal to minimum amount
- Bill due dates must be valid date patterns
- Bill matches must reference valid transactions

**Rule Aggregate Invariants:**
- Rule must have at least one trigger condition
- Rule must have at least one action
- Rule triggers and actions must be compatible types
- Rule execution order must be positive integer

**Piggy Bank Aggregate Invariants:**
- Target amount must be positive
- Current saved amount cannot be negative
- Current saved amount cannot exceed target amount
- Must reference an existing asset account owned by the user

**Cross-Aggregate Invariants:**
- Account references in transactions must exist
- Category assignments must reference valid categories
- Currency references must be valid ISO 4217 codes
- User ownership must be consistent across all aggregates

## Sources
- "Firefly III Data Model" — https://docs.firefly-iii.org/how-to/firefly-iii/features/accounts/ — retrieved 2025-01-09
- "Firefly III Transaction Rules" — https://docs.firefly-iii.org/how-to/firefly-iii/features/transactions/ — retrieved 2025-01-09

<!-- ai-docs-metadata
{"last_audit":"2025-01-09","fingerprints":{"sources":{"https://docs.firefly-iii.org/how-to/firefly-iii/features/accounts/":"","https://docs.firefly-iii.org/how-to/firefly-iii/features/transactions/":""},"sections":{"aggregates":""}}}
-->