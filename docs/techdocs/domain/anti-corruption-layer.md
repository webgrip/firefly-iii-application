---
title: "Anti-Corruption Layer"
description: "Boundary patterns protecting domain model integrity."
tags: [ddd, acl, integration]
search: { boost: 2, exclude: false }
icon: material/shield-check
author: "<Infrastructure/Platform Team>"
date: 2025-01-09
---

**Purpose:** Document translation boundaries that protect domain integrity.

**Contents**
- [ACL Implementations](#acl-implementations)
- [Translation Patterns](#translation-patterns)
- [Boundary Protection](#boundary-protection)
- [Sources](#sources)

## ACL Implementations

**Data Import ACL:**
Protects internal transaction model from external bank format variations.

| External Format | Internal Model | Translation Responsibility | Source |
|-----------------|----------------|----------------------------|--------|
| CSV Bank Export | Transaction Group | CSV mapper validates and transforms field mappings | "Firefly III Data Import" — https://docs.firefly-iii.org/how-to/data-importer/ — retrieved 2025-01-09 |
| OFX Files | Transaction Journal | OFX parser extracts transactions and accounts | "Firefly III Data Import" — https://docs.firefly-iii.org/how-to/data-importer/ — retrieved 2025-01-09 |
| Nordigen API | Account/Transaction | API adapter handles pagination and rate limiting | "Firefly III Data Import" — https://docs.firefly-iii.org/how-to/data-importer/ — retrieved 2025-01-09 |
| Spectre API | Financial Data | API client manages authentication and data transformation | "Firefly III Data Import" — https://docs.firefly-iii.org/how-to/data-importer/ — retrieved 2025-01-09 |

**API Presentation ACL:**
Shields internal domain models from REST API concerns.

| Domain Aggregate | API Resource | Translation Layer | Source |
|------------------|--------------|-------------------|--------|
| Transaction Group | TransactionResource | Laravel Resource with field mapping | "Firefly III API" — https://docs.firefly-iii.org/how-to/firefly-iii/features/api/ — retrieved 2025-01-09 |
| Account | AccountResource | Account type enumeration to string conversion | "Firefly III API" — https://docs.firefly-iii.org/how-to/firefly-iii/features/api/ — retrieved 2025-01-09 |
| Budget | BudgetResource | Period calculation and amount formatting | "Firefly III API" — https://docs.firefly-iii.org/how-to/firefly-iii/features/api/ — retrieved 2025-01-09 |
| Bill | BillResource | Pattern matching rules to API representation | "Firefly III API" — https://docs.firefly-iii.org/how-to/firefly-iii/features/api/ — retrieved 2025-01-09 |

**Database Persistence ACL:**
Protects domain objects from database schema concerns.

| Domain Model | Database Schema | ORM Mapping | Source |
|--------------|-----------------|-------------|--------|
| Money Value Object | amount + currency_code columns | Laravel Eloquent casting | "Firefly III Database" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| Account Hierarchy | accounts table with type enum | Single table inheritance pattern | "Firefly III Database" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| Transaction Group | transactions + transaction_journals tables | One-to-many relationship mapping | "Firefly III Database" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |
| Rule Logic | rules + rule_triggers + rule_actions tables | Polymorphic associations | "Firefly III Database" — https://docs.firefly-iii.org/how-to/firefly-iii/installation/self-hosted/ — retrieved 2025-01-09 |

## Translation Patterns

**Import Translation Pipeline:**
1. **Format Detection**: Identify file type and structure
2. **Schema Mapping**: Map external fields to internal concepts
3. **Validation**: Ensure data meets domain invariants
4. **Transformation**: Convert to domain objects
5. **Persistence**: Save through domain repositories

**API Translation Patterns:**
```php
// Example: Transaction Resource transformation
class TransactionResource extends JsonResource
{
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'type' => $this->transaction_type->type, // Enum to string
            'attributes' => [
                'amount' => $this->amount->value,    // Money object to string
                'currency_code' => $this->amount->currency_code,
                'description' => $this->description,
                'date' => $this->date->format('Y-m-d'), // Date object to ISO
            ]
        ];
    }
}
```

**Database Translation Examples:**
```php
// Money Value Object casting
class Transaction extends Model
{
    protected $casts = [
        'amount' => MoneyValueObject::class,
        'date' => 'date',
    ];
}

// Account type enumeration
class Account extends Model
{
    public function getTypeAttribute($value)
    {
        return AccountType::from($value);
    }
}
```

## Boundary Protection

**Import Boundary Guards:**
- **Field Validation**: All imported fields validated against domain rules
- **Account Resolution**: External account names mapped to internal accounts
- **Currency Handling**: Foreign currencies converted using exchange rates
- **Duplicate Detection**: Prevent duplicate transaction imports

**API Boundary Guards:**
- **Input Validation**: Request data validated before domain operations
- **Output Sanitization**: Sensitive data filtered from API responses
- **Rate Limiting**: Protect against API abuse
- **Authentication**: Ensure user can only access owned data

**Database Boundary Guards:**
- **Query Scoping**: Automatic user-based data filtering
- **Migration Safety**: Database schema changes preserve data integrity
- **Constraint Enforcement**: Foreign key and check constraints prevent invalid data
- **Audit Logging**: Track data modifications for compliance

**Error Translation:**
- Domain exceptions mapped to appropriate HTTP status codes
- Database constraint violations translated to user-friendly messages
- Import errors provide actionable feedback for data correction
- API errors follow consistent JSON:API error format

## Integration Testing

**ACL Validation:**
- Test external format changes don't break internal models
- Verify API responses remain stable despite domain changes
- Ensure database migrations preserve ACL functionality
- Validate error handling across all translation boundaries

## Sources
- "Firefly III Data Import Architecture" — https://docs.firefly-iii.org/how-to/data-importer/ — retrieved 2025-01-09
- "Firefly III API Documentation" — https://docs.firefly-iii.org/how-to/firefly-iii/features/api/ — retrieved 2025-01-09

<!-- ai-docs-metadata
{"last_audit":"2025-01-09","fingerprints":{"sources":{"https://docs.firefly-iii.org/how-to/data-importer/":"","https://docs.firefly-iii.org/how-to/firefly-iii/features/api/":""},"sections":{"anti-corruption-layer":""}}}
-->