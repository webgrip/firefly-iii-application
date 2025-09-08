---
title: "Use Cases"
description: "Application service orchestration and business workflow coordination."
tags: [ddd, use-cases, application-services]
search: { boost: 2, exclude: false }
icon: material/cog
author: "<Infrastructure/Platform Team>"
date: 2025-01-09
---

**Purpose:** Document application layer orchestration patterns.

**Contents**
- [Use Case Patterns](#use-case-patterns)
- [Key Use Cases](#key-use-cases)
- [Orchestration Strategy](#orchestration-strategy)
- [Sources](#sources)

## Use Case Patterns

**Single Responsibility Principle:**
Each use case represents one business operation with clear input/output boundaries.

**Orchestration Only:**
Use cases coordinate domain objects and infrastructure services without implementing business rules.

**Example Use Case Structure:**
```php
class CreateTransactionUseCase
{
    public function __construct(
        private TransactionRepositoryInterface $transactions,
        private AccountRepositoryInterface $accounts,
        private RuleEngineService $ruleEngine,
        private EventDispatcher $events
    ) {}
    
    public function execute(CreateTransactionCommand $command): TransactionResult
    {
        // 1. Validate inputs
        // 2. Load domain objects  
        // 3. Execute domain operations
        // 4. Persist changes
        // 5. Publish events
        // 6. Return result
    }
}
```

## Key Use Cases

**Transaction Management:**

| Use Case | Input | Output | Domain Objects | Events Published | Source |
|----------|--------|--------|----------------|------------------|--------|
| CreateTransaction | TransactionCommand | TransactionResult | Transaction, Account | TransactionCreated | "Firefly III Transactions" — https://docs.firefly-iii.org/how-to/firefly-iii/features/transactions/ — retrieved 2025-01-09 |
| UpdateTransaction | UpdateTransactionCommand | TransactionResult | Transaction | TransactionUpdated | "Firefly III Transactions" — https://docs.firefly-iii.org/how-to/firefly-iii/features/transactions/ — retrieved 2025-01-09 |
| DeleteTransaction | DeleteTransactionCommand | SuccessResult | Transaction | TransactionDeleted | "Firefly III Transactions" — https://docs.firefly-iii.org/how-to/firefly-iii/features/transactions/ — retrieved 2025-01-09 |
| SplitTransaction | SplitTransactionCommand | TransactionGroupResult | TransactionGroup | TransactionSplit | "Firefly III Transactions" — https://docs.firefly-iii.org/how-to/firefly-iii/features/transactions/ — retrieved 2025-01-09 |

**Account Management:**

| Use Case | Input | Output | Domain Objects | Events Published | Source |
|----------|--------|--------|----------------|------------------|--------|
| CreateAccount | CreateAccountCommand | AccountResult | Account | AccountCreated | "Firefly III Accounts" — https://docs.firefly-iii.org/how-to/firefly-iii/features/accounts/ — retrieved 2025-01-09 |
| UpdateAccountBalance | UpdateBalanceCommand | BalanceResult | Account, Transaction | BalanceUpdated | "Firefly III Accounts" — https://docs.firefly-iii.org/how-to/firefly-iii/features/accounts/ — retrieved 2025-01-09 |
| CloseAccount | CloseAccountCommand | SuccessResult | Account | AccountClosed | "Firefly III Accounts" — https://docs.firefly-iii.org/how-to/firefly-iii/features/accounts/ — retrieved 2025-01-09 |

**Budget Operations:**

| Use Case | Input | Output | Domain Objects | Events Published | Source |
|----------|--------|--------|----------------|------------------|--------|
| CreateBudget | CreateBudgetCommand | BudgetResult | Budget | BudgetCreated | "Firefly III Budgets" — https://docs.firefly-iii.org/how-to/firefly-iii/features/budgets/ — retrieved 2025-01-09 |
| SetBudgetLimit | SetLimitCommand | BudgetLimitResult | Budget, BudgetLimit | BudgetLimitSet | "Firefly III Budgets" — https://docs.firefly-iii.org/how-to/firefly-iii/features/budgets/ — retrieved 2025-01-09 |
| CalculateBudgetProgress | CalculateProgressQuery | BudgetProgressResult | Budget, Transaction | BudgetProgressCalculated | "Firefly III Budgets" — https://docs.firefly-iii.org/how-to/firefly-iii/features/budgets/ — retrieved 2025-01-09 |

**Rule Engine:**

| Use Case | Input | Output | Domain Objects | Events Published | Source |
|----------|--------|--------|----------------|------------------|--------|
| CreateRule | CreateRuleCommand | RuleResult | Rule, RuleTrigger, RuleAction | RuleCreated | "Firefly III Rules" — https://docs.firefly-iii.org/how-to/firefly-iii/features/rules/ — retrieved 2025-01-09 |
| ExecuteRules | ExecuteRulesCommand | RuleExecutionResult | Rule, Transaction | RuleExecuted | "Firefly III Rules" — https://docs.firefly-iii.org/how-to/firefly-iii/features/rules/ — retrieved 2025-01-09 |
| TestRule | TestRuleQuery | RuleTestResult | Rule | RuleTested | "Firefly III Rules" — https://docs.firefly-iii.org/how-to/firefly-iii/features/rules/ — retrieved 2025-01-09 |

**Data Import:**

| Use Case | Input | Output | Domain Objects | Events Published | Source |
|----------|--------|--------|----------------|------------------|--------|
| ImportTransactions | ImportCommand | ImportResult | ImportJob, Transaction | ImportStarted, TransactionImported | "Firefly III Data Import" — https://docs.firefly-iii.org/how-to/data-importer/ — retrieved 2025-01-09 |
| ValidateImportData | ValidateDataCommand | ValidationResult | ImportMapping | DataValidated | "Firefly III Data Import" — https://docs.firefly-iii.org/how-to/data-importer/ — retrieved 2025-01-09 |
| ConfigureImport | ConfigureImportCommand | ImportConfigResult | ImportConfiguration | ImportConfigured | "Firefly III Data Import" — https://docs.firefly-iii.org/how-to/data-importer/ — retrieved 2025-01-09 |

## Orchestration Strategy

**Error Handling Pattern:**
```php
public function execute(CreateTransactionCommand $command): TransactionResult
{
    try {
        DB::beginTransaction();
        
        // Domain operations
        $sourceAccount = $this->accounts->findById($command->sourceAccountId);
        $destinationAccount = $this->accounts->findById($command->destinationAccountId);
        
        $transaction = Transaction::create(
            $sourceAccount,
            $destinationAccount, 
            $command->amount,
            $command->description
        );
        
        $this->transactions->store($transaction);
        
        // Apply rules
        $this->ruleEngine->processTransaction($transaction);
        
        DB::commit();
        
        // Publish events after successful persistence
        $this->events->dispatch(new TransactionCreated($transaction));
        
        return TransactionResult::success($transaction);
        
    } catch (DomainException $e) {
        DB::rollback();
        return TransactionResult::failure($e->getMessage());
    }
}
```

**Input Validation:**
- Commands validated at application boundary
- Domain invariants enforced by aggregates
- Cross-aggregate validation in use cases
- External service validation through adapters

**Event Publishing:**
- Events published after successful persistence
- Event handlers run asynchronously where possible
- Event publishing failures logged but don't break primary operation
- Events carry minimal payload (IDs + basic data)

**Query vs Command Separation:**
- **Commands**: Modify state, return success/failure status
- **Queries**: Read data, return domain objects or DTOs
- **Query handlers**: Optimized for read performance
- **Command handlers**: Focus on consistency and validation

**Transaction Boundaries:**
- Use cases define transaction boundaries
- One database transaction per use case execution
- Compensating actions for cross-boundary operations
- Event publication outside transaction boundaries

**Dependency Coordination:**
```php
public function execute(ImportTransactionsCommand $command): ImportResult
{
    $importJob = ImportJob::start($command->fileName, $command->userId);
    
    // 1. Parse file
    $transactions = $this->fileParser->parse($command->filePath);
    
    // 2. Validate data
    $validationResult = $this->validator->validate($transactions);
    if ($validationResult->hasErrors()) {
        return ImportResult::validationFailure($validationResult->getErrors());
    }
    
    // 3. Import transactions
    foreach ($transactions as $transactionData) {
        $result = $this->createTransactionUseCase->execute(
            CreateTransactionCommand::fromImport($transactionData)
        );
        
        $importJob->recordResult($result);
    }
    
    // 4. Finalize import
    $importJob->complete();
    
    $this->events->dispatch(new ImportCompleted($importJob));
    
    return ImportResult::success($importJob);
}
```

## Testing Strategy

**Use Case Testing:**
- Mock repository and adapter interfaces
- Test successful execution paths
- Test error conditions and rollback behavior
- Verify event publication
- Test input validation

**Integration Testing:**
- Test use case orchestration with real dependencies
- Verify transaction boundary behavior
- Test cross-use case workflows
- Validate event handler interactions

## Sources
- "Firefly III Application Architecture" — https://docs.firefly-iii.org/how-to/firefly-iii/features/ — retrieved 2025-01-09
- "Firefly III Workflow Documentation" — https://docs.firefly-iii.org/how-to/firefly-iii/features/transactions/ — retrieved 2025-01-09

<!-- ai-docs-metadata
{"last_audit":"2025-01-09","fingerprints":{"sources":{"https://docs.firefly-iii.org/how-to/firefly-iii/features/":"","https://docs.firefly-iii.org/how-to/firefly-iii/features/transactions/":""},"sections":{"use-cases":""}}}
-->