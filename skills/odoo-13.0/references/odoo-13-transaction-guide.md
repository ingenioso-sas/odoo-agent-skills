---
name: odoo-13-transaction
description: Complete guide for handling database transactions, UniqueViolation errors, savepoints, and commit operations in Odoo 13.
globs: "**/*.{py,xml}"
topics:
  - Transaction states and error handling
  - UniqueViolation (duplicate key) errors
  - Savepoint usage patterns
  - commit() and rollback() best practices
  - InFailedSqlTransaction errors
  - Serialization errors
when_to_use:
  - Handling duplicate key errors
  - Working with savepoints for error isolation
  - Understanding transaction abort states
  - Preventing serialization conflicts
---

# Odoo 13 Transaction Guide

Complete guide for handling database transactions, UniqueViolation errors, savepoints, and commit operations in Odoo 13.

## Table of Contents

1. [Transaction States](#transaction-states)
2. [UniqueViolation Errors](#uniqueviolation-errors)
3. [Savepoint Usage](#savepoint-usage)
4. [commit() and rollback()](#commit-and-rollback)
5. [Transaction Aborted Errors](#transaction-aborted-errors)
6. [Serialization Errors](#serialization-errors)

---

## Transaction States

### PostgreSQL Transaction Isolation

Odoo 13 uses `REPEATABLE READ` isolation level by default.

**What this means**:
- Transactions operate on snapshots taken at the first query.
- Concurrent updates are detected and may cause serialization errors.
- Changes from other transactions are not visible during your transaction.

### Transaction State Flow

```
Normal → [Error] → Aborted → [rollback] → Normal
                    ↓
                 [commit] → ERROR! (cannot commit aborted transaction)
```

**Key Point**: Once a transaction enters the "aborted" state due to an error, **all subsequent commands will fail** until you execute `ROLLBACK` (or roll back to a savepoint).

---

## UniqueViolation Errors

### What is UniqueViolation?

PostgreSQL error code `23505` (UniqueViolation) occurs when inserting or updating data violates a unique constraint.

```python
# Example: Trying to create a duplicate record
try:
    with self.env.cr.savepoint():
        self.create({'email': 'test@example.com'})
except psycopg2.errors.UniqueViolation:
    _logger.info("Email already exists")
```

### Handling UniqueViolation Correctly

```python
# GOOD: Use savepoint to isolate the error
with self.env.cr.savepoint():
    try:
        record = self.create({'email': email})
    except psycopg2.errors.UniqueViolation:
        # Savepoint rolled back, transaction still valid
        pass

# GOOD: Check for existence first
existing = self.search([('email', '=', email)], limit=1)
if not existing:
    record = self.create({'email': email})
```

---

## Savepoint Usage

### What is a Savepoint?

A savepoint creates a nested transaction that can be rolled back without affecting the outer transaction.

### Basic Savepoint Pattern

```python
# Savepoint context manager
with self.env.cr.savepoint():
    # Any error here rolls back to the savepoint
    record = self.create({'name': 'test'})
    # ... logic that might fail ...
# Transaction continues normally after the savepoint
```

---

## commit() and rollback()

### When to Use commit()

**WARNING**: Manual `commit()` is rarely needed in Odoo and can be dangerous if used incorrectly!

Odoo automatically commits at the end of HTTP requests. Only use manual commit for:
1. **Long-running batch jobs** (to release locks periodically).
2. **Multi-transaction operations** (certain cron jobs).

```python
# GOOD: Batch commit for long operations
for i in range(0, len(records), 100):
    batch = records[i:i+100]
    batch.write({'processed': True})
    self.env.cr.commit()  # Release locks and persist changes
```

### rollback() Usage

```python
# GOOD: Rollback on error in batch operation
try:
    # ... complex logic ...
except Exception:
    self.env.cr.rollback()
    _logger.error("Operation failed, rolled back")
```

---

## Transaction Aborted Errors

### InFailedSqlTransaction

After any database error, PostgreSQL enters an "aborted transaction" state.

```python
# ERROR: current transaction is aborted, commands ignored until end of transaction block
```

### Proper Error Recovery

**Always use a savepoint** if you expect an operation might fail and you want to continue the transaction.

```python
with self.env.cr.savepoint():
    try:
        # Potentially failing DB operation
        self.env.cr.execute("INSERT ...")
    except psycopg2.Error:
        # Transaction is rolled back to savepoint and remains usable
        pass
```

---

## Serialization Errors

### What is Serialization Error?

PostgreSQL error code `40001` (serialization_error) occurs when concurrent transactions conflict.

### Solution: Record Locking with FOR UPDATE NOWAIT

Standard Odoo pattern for concurrency control:

```python
from psycopg2.errors import LockNotAvailable

try:
    with self.env.cr.savepoint():
        self.env.cr.execute(
            'SELECT id FROM %s WHERE id = %%s FOR UPDATE NOWAIT' % self._table,
            (record.id,)
        )
        # Record is locked, safe to proceed
        record.write({'state': 'processing'})
except LockNotAvailable:
    # Another transaction is already processing this record
    _logger.info("Record %s is locked, skipping", record.id)
```

---

## Best Practices

1. **Always use `with cr.savepoint()`** for operations that might fail (like `create` or `write` on models with constraints).
2. **Avoid manual `commit()`** in business logic; it breaks atomicity and can lead to inconsistent data if a later part of the request fails.
3. **Check for duplicates** using `search` before attempting to `create` to avoid unnecessary transaction failures.
4. **Use `FOR UPDATE NOWAIT`** for cron jobs to prevent multiple workers from processing the same records simultaneously.
5. **Flush the ORM cache** before executing direct SQL queries using `self.flush()`.
