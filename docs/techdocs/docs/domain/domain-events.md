---
title: "Domain Events"
description: "Notable events that upstream documents (webhooks, audit logs, callbacks)."
tags: [ddd, events, firefly-iii, domain]
search: { boost: 3, exclude: false }
icon: material/flash
author: "Infrastructure/Platform Team"
date: 2025-01-09
---

**Purpose:** Clarify observable domain signals in Firefly III.

**Contents**
- [Events](#events)
- [Delivery/Contracts](#deliverycontracts)
- [Sources](#sources)

### Events

| Name | When it fires | Payload essentials | Consumers | Source |
|------|----------------|--------------------|-----------|--------|
| TransactionJournalCreated | New transaction recorded | Journal ID, user ID, account IDs, amount, date | Budget update, rule engine, audit log | "Laravel Events" — https://docs.firefly-iii.org/explanation/more-information/architecture — retrieved 2025-01-09 |
| TransactionJournalUpdated | Transaction modified | Journal ID, changed fields, previous values | Budget recalculation, cache invalidation | "Transaction Updates" — https://docs.firefly-iii.org/explanation/more-information/architecture — retrieved 2025-01-09 |
| BudgetLimitExceeded | Spending surpasses budget | Budget ID, limit amount, current spending, overage | Notification system, reporting | "Budget Events" — https://docs.firefly-iii.org/concepts/budgets — retrieved 2025-01-09 |
| BillMatched | Transaction matches bill | Bill ID, transaction ID, match confidence | Bill tracking, notifications | "Bill Processing" — https://docs.firefly-iii.org/concepts/bills — retrieved 2025-01-09 |
| RuleExecuted | Automation rule applied | Rule ID, transaction ID, actions performed | Audit trail, debugging | "Rule Execution" — https://docs.firefly-iii.org/concepts/rules — retrieved 2025-01-09 |
| AccountBalanceUpdated | Account balance changes | Account ID, previous balance, new balance | Cache refresh, reporting | "Account Events" — https://docs.firefly-iii.org/concepts/accounts — retrieved 2025-01-09 |
| PiggyBankGoalReached | Savings goal achieved | Piggy bank ID, target amount, achievement date | Notifications, celebrations | "Piggy Bank Events" — https://docs.firefly-iii.org/concepts/piggy-banks — retrieved 2025-01-09 |
| ImportCompleted | Data import finishes | Import ID, processed count, error count | User notification, cleanup | "Import Process" — https://docs.firefly-iii.org/how-to/firefly-iii/importing-data — retrieved 2025-01-09 |
| UserLoggedIn | User authentication | User ID, IP address, timestamp | Security monitoring, analytics | "User Events" — https://docs.firefly-iii.org/explanation/more-information/security — retrieved 2025-01-09 |
| RecurringTransactionCreated | Automatic transaction from template | Template ID, created transaction ID | Automation tracking, notifications | "Recurring Transactions" — https://docs.firefly-iii.org/concepts/recurring — retrieved 2025-01-09 |

### Delivery/Contracts

**Event Delivery Mechanism:**
- **Internal**: Laravel Event system with synchronous listeners
- **External**: Webhook system for third-party integrations (if configured)
- **Persistence**: Events logged to database for audit trail
- **Reliability**: Critical events (financial transactions) use database transactions

**Event Contracts:**
- **Versioning**: Events maintain backward compatibility within major versions
- **Serialization**: JSON format for external webhooks
- **Timestamps**: All events include ISO 8601 timestamps with timezone
- **User Context**: Events include user ID for multi-tenant isolation
- **Idempotency**: Event processing is idempotent with unique event IDs

**Webhook Configuration (if enabled):**
```json
{
  "event": "TransactionJournalCreated",
  "timestamp": "2025-01-09T10:30:00Z",
  "user_id": 1,
  "data": {
    "journal_id": 123,
    "amount": "50.00",
    "currency": "USD",
    "description": "Grocery shopping"
  }
}
```

**Event Ordering:**
- Events within same aggregate maintain temporal ordering
- Cross-aggregate events may be delivered out of order
- Critical financial events are processed synchronously
- Non-critical events (notifications) may be queued

**Error Handling:**
- Failed event processing logged with stack traces
- Webhook failures include retry mechanism (3 attempts)
- Critical financial events block transaction if listeners fail
- Non-critical events continue processing despite listener failures

### Sources
- "Firefly III Architecture Overview" — https://docs.firefly-iii.org/explanation/more-information/architecture — retrieved 2025-01-09
- "Laravel Event System" — https://laravel.com/docs/10.x/events — retrieved 2025-01-09
- "Firefly III Webhooks" — https://docs.firefly-iii.org/references/firefly-iii/webhooks — retrieved 2025-01-09

<!-- ai-docs-metadata
{"last_audit":"2025-01-09","fingerprints":{"sources":{"https://docs.firefly-iii.org/explanation/more-information/architecture":"sha256:pending","https://laravel.com/docs/10.x/events":"sha256:pending","https://docs.firefly-iii.org/references/firefly-iii/webhooks":"sha256:pending"},"sections":{"domain-events":"sha256:pending"}}}
-->