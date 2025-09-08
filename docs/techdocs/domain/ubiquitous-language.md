---
title: "Ubiquitous Language"
description: "Canonical terms used by the Firefly III application."
tags: [ddd, language]
search: { boost: 4, exclude: false }
icon: material/alphabetical
author: "<Infrastructure/Platform Team>"
date: 2025-01-09
---

**Purpose:** Capture upstream's domain terminology for precise communication.

**Contents**
- [Glossary](#glossary)
- [Notes](#notes)
- [Sources](#sources)

## Glossary

| Term | Definition (short, from upstream) | Source |
|------|-----------------------------------|--------|
| Account | A financial container that holds money (asset, expense, revenue, liability) | "Firefly III Concepts" — https://docs.firefly-iii.org/how-to/firefly-iii/features/accounts/ — retrieved 2025-01-09 |
| Asset Account | Account representing money you own (checking, savings, cash) | "Firefly III Concepts" — https://docs.firefly-iii.org/how-to/firefly-iii/features/accounts/ — retrieved 2025-01-09 |
| Bill | A recurring expense that is expected regularly (rent, utilities) | "Firefly III Concepts" — https://docs.firefly-iii.org/how-to/firefly-iii/features/bills/ — retrieved 2025-01-09 |
| Budget | A spending limit for a specific category and time period | "Firefly III Concepts" — https://docs.firefly-iii.org/how-to/firefly-iii/features/budgets/ — retrieved 2025-01-09 |
| Category | A way to group transactions by purpose (groceries, entertainment) | "Firefly III Concepts" — https://docs.firefly-iii.org/how-to/firefly-iii/features/categories/ — retrieved 2025-01-09 |
| Deposit | Money coming into an asset account from a revenue account | "Firefly III Concepts" — https://docs.firefly-iii.org/how-to/firefly-iii/features/transactions/ — retrieved 2025-01-09 |
| Expense Account | Account representing places where money is spent (store, landlord) | "Firefly III Concepts" — https://docs.firefly-iii.org/how-to/firefly-iii/features/accounts/ — retrieved 2025-01-09 |
| Liability | Account representing money you owe (credit card, mortgage) | "Firefly III Concepts" — https://docs.firefly-iii.org/how-to/firefly-iii/features/accounts/ — retrieved 2025-01-09 |
| Piggy Bank | A virtual savings goal within an asset account | "Firefly III Concepts" — https://docs.firefly-iii.org/how-to/firefly-iii/features/piggy-banks/ — retrieved 2025-01-09 |
| Revenue Account | Account representing sources of income (employer, client) | "Firefly III Concepts" — https://docs.firefly-iii.org/how-to/firefly-iii/features/accounts/ — retrieved 2025-01-09 |
| Rule | Automatic action triggered when transaction criteria are met | "Firefly III Concepts" — https://docs.firefly-iii.org/how-to/firefly-iii/features/rules/ — retrieved 2025-01-09 |
| Split Transaction | Single transaction divided into multiple categories/budgets | "Firefly III Concepts" — https://docs.firefly-iii.org/how-to/firefly-iii/features/transactions/ — retrieved 2025-01-09 |
| Tag | A flexible label that can be attached to transactions | "Firefly III Concepts" — https://docs.firefly-iii.org/how-to/firefly-iii/features/tags/ — retrieved 2025-01-09 |
| Transaction | A movement of money between accounts at a specific time | "Firefly III Concepts" — https://docs.firefly-iii.org/how-to/firefly-iii/features/transactions/ — retrieved 2025-01-09 |
| Transaction Group | Collection of related transactions that happen simultaneously | "Firefly III Concepts" — https://docs.firefly-iii.org/how-to/firefly-iii/features/transactions/ — retrieved 2025-01-09 |
| Transfer | Money moving between two asset accounts you own | "Firefly III Concepts" — https://docs.firefly-iii.org/how-to/firefly-iii/features/transactions/ — retrieved 2025-01-09 |
| Withdrawal | Money leaving an asset account to go to an expense account | "Firefly III Concepts" — https://docs.firefly-iii.org/how-to/firefly-iii/features/transactions/ — retrieved 2025-01-09 |

## Notes

- Avoid synonyms that upstream does not use
- Account types follow double-entry bookkeeping principles
- All monetary movements are recorded as transactions between accounts
- Tags and categories serve different purposes (tags are flexible, categories are hierarchical)
- Bills represent expected expenses, budgets represent spending limits

## Sources
- "Firefly III Account Concepts" — https://docs.firefly-iii.org/how-to/firefly-iii/features/accounts/ — retrieved 2025-01-09
- "Firefly III Transaction Concepts" — https://docs.firefly-iii.org/how-to/firefly-iii/features/transactions/ — retrieved 2025-01-09
- "Firefly III Budget Concepts" — https://docs.firefly-iii.org/how-to/firefly-iii/features/budgets/ — retrieved 2025-01-09

<!-- ai-docs-metadata
{"last_audit":"2025-01-09","fingerprints":{"sources":{"https://docs.firefly-iii.org/how-to/firefly-iii/features/accounts/":"","https://docs.firefly-iii.org/how-to/firefly-iii/features/transactions/":"","https://docs.firefly-iii.org/how-to/firefly-iii/features/budgets/":""},"sections":{"ubiquitous-language":""}}}
-->