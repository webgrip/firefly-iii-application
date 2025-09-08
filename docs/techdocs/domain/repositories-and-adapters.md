---
title: "Repositories & Adapters"
description: "Data access patterns and external service integration."
tags: [ddd, repositories, adapters]
search: { boost: 2, exclude: false }
icon: material/database-sync
author: "<Infrastructure/Platform Team>"
date: 2025-01-09
---

**Purpose:** Document data access and external integration patterns.

**Contents**
- [Repository Patterns](#repository-patterns)
- [Adapter Implementations](#adapter-implementations)
- [Data Access Strategy](#data-access-strategy)
- [Sources](#sources)

## Repository Patterns

**Domain Repositories (Interfaces):**

| Repository | Aggregate | Key Methods | Purpose | Source |
|------------|-----------|-------------|---------|--------|
| AccountRepositoryInterface | Account | findByUser(), findByType(), findActive() | Account discovery and validation | "Firefly III Account Management" — https://docs.firefly-iii.org/how-to/firefly-iii/features/accounts/ — retrieved 2025-01-09 |
| TransactionRepositoryInterface | Transaction Group | findByDateRange(), findByCategory(), store() | Transaction persistence and querying | "Firefly III Transaction System" — https://docs.firefly-iii.org/how-to/firefly-iii/features/transactions/ — retrieved 2025-01-09 |
| BudgetRepositoryInterface | Budget | findByPeriod(), calculateSpent(), updateLimit() | Budget tracking and limits | "Firefly III Budget System" — https://docs.firefly-iii.org/how-to/firefly-iii/features/budgets/ — retrieved 2025-01-09 |
| BillRepositoryInterface | Bill | findDue(), matchTransaction(), updatePayment() | Bill management and matching | "Firefly III Bill Tracking" — https://docs.firefly-iii.org/how-to/firefly-iii/features/bills/ — retrieved 2025-01-09 |
| RuleRepositoryInterface | Rule | findActive(), executeRule(), findByTrigger() | Rule engine operations | "Firefly III Rules" — https://docs.firefly-iii.org/how-to/firefly-iii/features/rules/ — retrieved 2025-01-09 |

**Implementation Strategy:**
- Repositories define **intent-based methods** reflecting domain operations
- No generic CRUD leakage to application layer
- Repository implementations handle database specifics
- Laravel Eloquent ORM used for concrete implementations

**Example Repository Interface:**
```php
interface TransactionRepositoryInterface
{
    public function findByDateRange(DateRange $range, User $user): Collection;
    public function findUncategorized(User $user): Collection;
    public function store(TransactionGroup $group): TransactionGroup;
    public function findForBudgetCalculation(Budget $budget, DateRange $period): Collection;
    public function getTotalsByCategory(User $user, DateRange $range): array;
}
```

## Adapter Implementations

**External Service Adapters:**

| Service | Adapter | Purpose | Integration Pattern | Source |
|---------|---------|---------|---------------------|--------|
| Bank APIs | NordigenAdapter | Fetch account data and transactions | OAuth2 + REST API | "Firefly III Data Import" — https://docs.firefly-iii.org/how-to/data-importer/ — retrieved 2025-01-09 |
| Email Service | MailgunAdapter | Send notifications and reports | HTTP API + webhooks | "Firefly III Email" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| Exchange Rates | CurrencyAPIAdapter | Real-time currency conversion | REST API + caching | "Firefly III Currencies" — https://docs.firefly-iii.org/how-to/firefly-iii/features/currencies/ — retrieved 2025-01-09 |
| File Storage | S3StorageAdapter | Store user uploads and exports | AWS S3 compatible API | "Firefly III File Storage" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |

**Infrastructure Adapters:**

| Component | Adapter | Technology | Configuration | Source |
|-----------|---------|------------|---------------|--------|
| Cache | RedisAdapter | Redis | Connection pooling, TTL management | "Firefly III Performance" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| Queue | DatabaseQueueAdapter | MySQL/PostgreSQL | Job serialization, retry logic | "Firefly III Background Jobs" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| Session | RedisSessionAdapter | Redis | Session persistence, clustering | "Firefly III Sessions" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| Logging | FileLogAdapter | Local filesystem | Log rotation, structured format | "Firefly III Logging" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |

**Adapter Pattern Example:**
```php
class NordigenBankAdapter implements BankDataSourceInterface
{
    public function fetchAccounts(string $institutionId): Collection
    {
        $response = $this->httpClient->get("/accounts/{$institutionId}");
        
        return $response->json()['accounts']
            ->map(fn($data) => $this->transformToAccount($data));
    }
    
    private function transformToAccount(array $data): Account
    {
        // Transform external format to domain model
        return Account::create([
            'name' => $data['name'],
            'type' => $this->mapAccountType($data['type']),
            'currency' => Currency::fromCode($data['currency'])
        ]);
    }
}
```

## Data Access Strategy

**Read Model Optimization:**
- **Denormalized Views**: Pre-calculated budget summaries and account balances
- **Report Aggregates**: Monthly/yearly financial summaries for performance
- **Search Indexes**: Full-text search on transaction descriptions and categories
- **Caching Layer**: Redis cache for frequently accessed account and transaction data

**Write Model Consistency:**
- **Transaction Boundaries**: Database transactions ensure consistency across aggregates
- **Event Publishing**: Domain events published after successful persistence
- **Optimistic Locking**: Version fields prevent concurrent modification conflicts
- **Audit Trail**: All modifications logged with user and timestamp

**Performance Patterns:**
```php
// Eager loading to prevent N+1 queries
$transactions = $this->transactionRepository
    ->findByDateRange($dateRange, $user)
    ->with(['category', 'budget', 'tags', 'attachments']);

// Cached calculations
$budgetProgress = Cache::remember(
    "budget_progress_{$budget->id}_{$period->hash()}", 
    3600, 
    fn() => $this->calculateBudgetProgress($budget, $period)
);
```

**Query Optimization:**
- Database indexes on common query patterns (user_id, date ranges, categories)
- Pagination for large result sets
- Selective field loading for list views
- Background aggregation for expensive calculations

**Data Consistency Patterns:**
- **Eventual Consistency**: Account balances calculated asynchronously for reporting
- **Strong Consistency**: Transaction creation requires immediate balance validation
- **Compensating Actions**: Failed imports can be rolled back through stored metadata
- **Idempotent Operations**: Import jobs can be safely retried without duplicates

## Integration Testing

**Repository Testing:**
- Test intent-based methods against real database
- Verify complex queries return expected domain objects
- Ensure repository implementations honor domain constraints
- Validate performance characteristics of key operations

**Adapter Testing:**
- Mock external services for unit tests
- Integration tests against real service sandboxes
- Circuit breaker behavior during service outages
- Retry logic and error handling validation

## Sources
- "Firefly III Architecture" — https://docs.firefly-iii.org/how-to/firefly-iii/features/ — retrieved 2025-01-09
- "Firefly III Performance Guide" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09

<!-- ai-docs-metadata
{"last_audit":"2025-01-09","fingerprints":{"sources":{"https://docs.firefly-iii.org/how-to/firefly-iii/features/":"","https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/":""},"sections":{"repositories-adapters":""}}}
-->