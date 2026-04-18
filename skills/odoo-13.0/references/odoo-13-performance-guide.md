---
name: odoo-13-performance
description: Complete guide for writing performant Odoo 13 code, focusing on N+1 query prevention, batch operations, and optimization patterns.
globs: "**/*.{py,xml}"
topics:
  - Prefetch mechanism (how it works, understanding groups)
  - N+1 query prevention patterns
  - Batch operations (create, write, unlink)
  - Field selection optimization (search_read, load, bin_size)
  - Aggregation optimization (read_group)
  - Compute field optimization (store, avoiding recursion)
  - SQL optimization (when to use, cr.execute)
  - Clean code patterns (mapped, filtered, sorted)
when_to_use:
  - Optimizing slow code
  - Preventing N+1 queries
  - Writing batch operations
  - Optimizing computed fields
  - Using read_group() for aggregations
  - Using direct SQL for aggregations
---

# Odoo 13 Performance Guide

Complete guide for writing performant Odoo 13 code, focusing on N+1 query prevention and clean patterns.

## Table of Contents

1. [Prefetch Mechanism](#prefetch-mechanism)
2. [N+1 Query Prevention](#n1-query-prevention)
3. [Batch Operations](#batch-operations)
4. [Field Selection Optimization](#field-selection-optimization)
5. [Compute Field Optimization](#compute-field-optimization)
6. [SQL Optimization](#sql-optimization)

---

## Prefetch Mechanism

### How Prefetch Works

Odoo automatically prefetches records in batches to minimize queries.

**How it works**:
1. When you access a field on a recordset, Odoo loads that field for ALL records in the recordset.
2. Related records are also prefetched up to a limit (usually 1000).

```python
# GOOD: Automatic prefetch
orders = self.search([('state', '=', 'done')])  # 1 query for orders
for order in orders:
    print(order.name)  # 1 query for all names
    print(order.partner_id.name)  # 1 query for all partners
# Total: 3 queries regardless of number of orders
```

---

## N+1 Query Prevention

### Pattern 1: Search Inside Loop (BAD)

```python
# BAD: N+1 query
for order in orders:
    payments = self.env['payment.transaction'].search([
        ('order_id', '=', order.id)
    ])
    order.payment_count = len(payments)
# Result: 1 + N queries

# GOOD: Use read_group or search with IN domain
order_ids = orders.ids
all_payments = self.env['payment.transaction'].search_read(
    [('order_id', 'in', order_ids)],
    ['order_id']
)
# Count by order using a dictionary
from collections import defaultdict
payment_counts = defaultdict(int)
for payment in all_payments:
    payment_counts[payment['order_id'][0]] += 1

for order in orders:
    order.payment_count = payment_counts.get(order.id, 0)
# Result: 1 query
```

---

## Batch Operations

### Batch Create (Odoo 12+)

Odoo 13 supports batch creation, which is much faster than creating in a loop.

```python
# GOOD: Batch create
records = self.create([
    {'name': 'Record %s' % i, 'state': 'draft'}
    for i in range(100)
])

# BAD: Create in loop
for i in range(100):
    self.create({'name': 'Record %s' % i})
```

### Batch Write

```python
# GOOD: Write on recordset
self.search([('state', '=', 'draft')]).write({'state': 'cancel'})

# BAD: Write in loop
for order in self.search([('state', '=', 'draft')]):
    order.write({'state': 'cancel'})
```

---

## Field Selection Optimization

### Use search_read for Specific Fields

```python
# GOOD: search_read when you need dicts, not recordsets
data = self.search_read(
    [('state', '=', 'done')],
    ['name', 'amount_total', 'date']
)
```

### Use read_group() for Aggregations

```python
# GOOD: read_group() returns a list of dictionaries
data = self.read_group(
    domain=[('state', '=', 'done')],
    fields=['amount_total'],
    groupby=['partner_id']
)
# Result: [{'partner_id': (1, 'Partner Name'), 'amount_total': 100, 'partner_id_count': 5}, ...]
```

---

## Compute Field Optimization

### Store Expensive Computations

```python
# GOOD: Store expensive aggregations
amount_total = fields.Float(
    string='Total',
    compute='_compute_amount_total',
    store=True,
)

@api.depends('line_ids.price_subtotal')
def _compute_amount_total(self):
    for order in self:
        order.amount_total = sum(order.line_ids.mapped('price_subtotal'))
```

---

## SQL Optimization

### When to Use Direct SQL

Use SQL for complex aggregations or bulk data migration.

```python
def get_statistics(self):
    """Direct SQL for complex aggregation"""
    self.env.cr.execute("""
        SELECT
            state,
            COUNT(*) as count,
            SUM(amount_total) as total
        FROM sale_order
        WHERE create_date >= %s
        GROUP BY state
    """, (fields.Date.today(),))
    return self.env.cr.dictfetchall()
```

---

## Clean Code Performance Patterns

### Use Mapped() and Filtered()

```python
# GOOD: Use mapped() for field access
partner_ids = orders.mapped('partner_id.id')

# GOOD: Filter before processing
done_orders = orders.filtered(lambda o: o.state == 'done')
```

---

## Performance Checklist

- [ ] Avoid `search()` inside loops.
- [ ] Use `mapped()` instead of list comprehension for field access.
- [ ] Use `search_read()` when you need dicts, not recordsets.
- [ ] Use `read_group()` for aggregations.
- [ ] Store expensive computed fields.
- [ ] Batch create/write/unlink operations.
- [ ] Add indexes on frequently searched fields.
- [ ] Use `filtered()` before operations.
- [ ] Don't use SQL for writes (use ORM).

---

**For more Odoo 13 guides, see [SKILL.md](../SKILL.md)**
