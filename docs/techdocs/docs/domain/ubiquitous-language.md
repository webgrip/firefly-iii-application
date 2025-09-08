---
title: "Ubiquitous Language"
description: "Canonical terms used by the Firefly III application."
tags: [ddd, language, firefly-iii, domain]
search: { boost: 4, exclude: false }
icon: material/alphabetical
author: "Infrastructure/Platform Team"
date: 2025-01-09
---

**Purpose:** Capture upstream's domain terminology for precise communication.

**Contents**
- [Glossary](#glossary)
- [Notes](#notes)
- [Sources](#sources)

### Glossary

| Term | Definition (short, from upstream) | Source |
|------|-----------------------------------|--------|
| Account | A container for money, representing real-world financial accounts | "Accounts" — https://docs.firefly-iii.org/concepts/accounts — retrieved 2025-01-09 |
| Asset Account | An account you own, like checking or savings accounts | "Asset Accounts" — https://docs.firefly-iii.org/concepts/accounts — retrieved 2025-01-09 |
| Expense Account | Accounts representing where money goes (stores, utilities, etc.) | "Expense Accounts" — https://docs.firefly-iii.org/concepts/accounts — retrieved 2025-01-09 |
| Revenue Account | Accounts representing where money comes from (employers, etc.) | "Revenue Accounts" — https://docs.firefly-iii.org/concepts/accounts — retrieved 2025-01-09 |
| Liability Account | Accounts representing money you owe (credit cards, loans) | "Liability Accounts" — https://docs.firefly-iii.org/concepts/accounts — retrieved 2025-01-09 |
| Transaction | A movement of money between accounts | "Transactions" — https://docs.firefly-iii.org/concepts/transactions — retrieved 2025-01-09 |
| Transaction Journal | A container holding one or more transactions that balance to zero | "Transaction Journals" — https://docs.firefly-iii.org/concepts/transactions — retrieved 2025-01-09 |
| Transaction Split | Multiple transactions within a single journal for complex entries | "Transaction Splits" — https://docs.firefly-iii.org/concepts/transactions — retrieved 2025-01-09 |
| Category | Grouping mechanism for transactions to track spending patterns | "Categories" — https://docs.firefly-iii.org/concepts/transactions — retrieved 2025-01-09 |
| Budget | Financial planning tool to set spending limits for categories | "Budgets" — https://docs.firefly-iii.org/concepts/budgets — retrieved 2025-01-09 |
| Budget Limit | Spending limit for a specific budget during a period | "Budget Limits" — https://docs.firefly-iii.org/concepts/budgets — retrieved 2025-01-09 |
| Available Budget | Amount available to spend in a budget period | "Available Budgets" — https://docs.firefly-iii.org/concepts/budgets — retrieved 2025-01-09 |
| Bill | Recurring expense that can be predicted and tracked | "Bills" — https://docs.firefly-iii.org/concepts/bills — retrieved 2025-01-09 |
| Rule | Automated logic to categorize and process transactions | "Rules" — https://docs.firefly-iii.org/concepts/rules — retrieved 2025-01-09 |
| Rule Group | Collection of rules processed in sequence | "Rule Groups" — https://docs.firefly-iii.org/concepts/rules — retrieved 2025-01-09 |
| Tag | Flexible labeling system for transactions | "Tags" — https://docs.firefly-iii.org/concepts/tags — retrieved 2025-01-09 |
| Piggy Bank | Savings goal for accumulating money toward a target | "Piggy Banks" — https://docs.firefly-iii.org/concepts/piggy-banks — retrieved 2025-01-09 |
| Currency | Monetary unit (USD, EUR, etc.) with exchange rate support | "Currencies" — https://docs.firefly-iii.org/concepts/currencies — retrieved 2025-01-09 |
| Recurring Transaction | Template for automatically creating repeated transactions | "Recurring Transactions" — https://docs.firefly-iii.org/concepts/recurring — retrieved 2025-01-09 |

### Notes
- Avoid synonyms that upstream does not use (e.g., "expense" vs "withdrawal")
- Transaction types follow double-entry bookkeeping principles
- All monetary amounts are stored with precision to avoid rounding errors
- Currency conversions maintain historical exchange rates

### Sources
- "Firefly III Concepts Documentation" — https://docs.firefly-iii.org/concepts — retrieved 2025-01-09
- "Firefly III Financial Concepts" — https://docs.firefly-iii.org/references/firefly-iii/financial-concepts — retrieved 2025-01-09

<!-- ai-docs-metadata
{"last_audit":"2025-01-09","fingerprints":{"sources":{"https://docs.firefly-iii.org/concepts":"sha256:pending","https://docs.firefly-iii.org/references/firefly-iii/financial-concepts":"sha256:pending"},"sections":{"ubiquitous-language":"sha256:pending"}}}
-->