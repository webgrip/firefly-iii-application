---
title: "Domain Events"
description: "Notable events that upstream documents (webhooks, audit logs, callbacks)."
tags: [ddd, events]
search: { boost: 3, exclude: false }
icon: material/flash
author: "<Infrastructure/Platform Team>"
date: 2025-01-09
---

**Purpose:** Clarify observable domain signals.

**Contents**
- [Events](#events)
- [Delivery/Contracts](#deliverycontracts)
- [Sources](#sources)

## Events

| Name | When it fires | Payload essentials | Consumers | Source |
|------|----------------|--------------------|-----------|--------|
| TransactionCreated | New transaction is successfully recorded | Transaction ID, amounts, accounts, date, user | Rules engine, budget tracker, reports | "Firefly III Rules" — https://docs.firefly-iii.org/how-to/firefly-iii/features/rules/ — retrieved 2025-01-09 |
| TransactionUpdated | Existing transaction is modified | Transaction ID, changed fields, old/new values | Rules engine, budget recalculation | "Firefly III Rules" — https://docs.firefly-iii.org/how-to/firefly-iii/features/rules/ — retrieved 2025-01-09 |
| TransactionDeleted | Transaction is removed from system | Transaction ID, original amount, accounts affected | Budget adjustment, account balance update | "Firefly III Rules" — https://docs.firefly-iii.org/how-to/firefly-iii/features/rules/ — retrieved 2025-01-09 |
| BudgetLimitExceeded | Spending exceeds budget limit for period | Budget ID, limit amount, spent amount, overage | Email notifications, dashboard alerts | "Firefly III Budgets" — https://docs.firefly-iii.org/how-to/firefly-iii/features/budgets/ — retrieved 2025-01-09 |
| BillMatched | Transaction matches a bill pattern | Bill ID, transaction ID, match confidence | Bill tracking, payment history | "Firefly III Bills" — https://docs.firefly-iii.org/how-to/firefly-iii/features/bills/ — retrieved 2025-01-09 |
| RuleExecuted | Automatic rule processes a transaction | Rule ID, transaction ID, actions performed | Audit log, user notifications | "Firefly III Rules" — https://docs.firefly-iii.org/how-to/firefly-iii/features/rules/ — retrieved 2025-01-09 |
| AccountBalanceChanged | Account balance updates due to transaction | Account ID, old balance, new balance, transaction | Account reconciliation, alerts | "Firefly III Accounts" — https://docs.firefly-iii.org/how-to/firefly-iii/features/accounts/ — retrieved 2025-01-09 |
| PiggyBankGoalReached | Savings goal target amount is achieved | Piggy bank ID, target amount, achievement date | Goal completion notifications | "Firefly III Piggy Banks" — https://docs.firefly-iii.org/how-to/firefly-iii/features/piggy-banks/ — retrieved 2025-01-09 |
| DataImportCompleted | External data import finishes processing | Import job ID, records processed, errors count | User notifications, import status | "Firefly III Data Import" — https://docs.firefly-iii.org/how-to/data-importer/ — retrieved 2025-01-09 |
| RecurringTransactionExecuted | Scheduled recurring transaction is created | Recurring transaction ID, new transaction ID | Transaction processing, scheduling | "Firefly III Recurring" — https://docs.firefly-iii.org/how-to/firefly-iii/features/recurring/ — retrieved 2025-01-09 |

## Delivery/Contracts

**Event Delivery Mechanisms:**
- **Laravel Events**: Internal event system for real-time processing
- **Webhooks**: HTTP callbacks for external system integration (if configured)
- **Audit Log**: Database storage for compliance and debugging
- **Email Notifications**: User-configurable email alerts
- **API Events**: RESTful endpoints for polling event history

**Event Structure (internal Laravel events):**
```php
class TransactionCreated
{
    public $transaction_id;
    public $user_id;
    public $account_from;
    public $account_to;
    public $amount;
    public $currency_code;
    public $description;
    public $date;
    public $created_at;
}
```

**Webhook Payload (if configured):**
```json
{
  "event": "transaction.created",
  "timestamp": "2025-01-09T10:30:00Z",
  "user_id": 123,
  "data": {
    "transaction_id": 456,
    "amount": "150.00",
    "currency": "EUR",
    "description": "Grocery shopping",
    "date": "2025-01-09",
    "source_account": "Checking Account",
    "destination_account": "Supermarket"
  }
}
```

**Event Ordering and Consistency:**
- Events are fired synchronously during transaction processing
- Rule execution events follow transaction events
- Budget and bill events are triggered after transaction persistence
- Import events are fired after batch processing completion

**Retry and Reliability:**
- Internal events use Laravel's database queue for reliability
- Webhook delivery includes retry logic with exponential backoff
- Failed webhook deliveries are logged for manual retry
- Event sourcing is not implemented; events are notifications only

**Idempotency:**
- Events include unique identifiers for deduplication
- Webhook endpoints should handle duplicate deliveries gracefully
- Event replaying is not supported; events are fire-and-forget

## Sources
- "Firefly III Rules Documentation" — https://docs.firefly-iii.org/how-to/firefly-iii/features/rules/ — retrieved 2025-01-09
- "Firefly III Webhooks" — https://docs.firefly-iii.org/how-to/firefly-iii/advanced-concepts/webhooks/ — retrieved 2025-01-09

<!-- ai-docs-metadata
{"last_audit":"2025-01-09","fingerprints":{"sources":{"https://docs.firefly-iii.org/how-to/firefly-iii/features/rules/":"","https://docs.firefly-iii.org/how-to/firefly-iii/advanced-concepts/webhooks/":""},"sections":{"domain-events":""}}}
-->