---
name: odoo-13-model
description: Complete reference for Odoo 13 ORM model methods, CRUD operations, domain syntax, and recordset handling. Use this guide when writing model methods, ORM queries, search operations, or working with recordsets.
globs: "**/models/**/*.py"
topics:
  - Recordset basics (browse, exists, empty)
  - Search methods (search, search_read, search_count)
  - Aggregation (read_group)
  - CRUD operations (create, read, write, unlink)
  - Domain syntax (operators, logical, relational)
  - Environment context (with_context, sudo)
  - Recordset iteration patterns (@api.multi)
when_to_use:
  - Writing ORM queries in Odoo 13
  - Performing CRUD operations
  - Building domain filters
  - Using read_group() for aggregations
  - Iterating over recordsets with @api.multi
  - Using environment context
---

# Odoo 13 Model Guide

Complete reference for Odoo 13 ORM model methods, CRUD operations, and recordset handling.

## Table of Contents

1. [Recordset Basics](#recordset-basics)
2. [Search Methods](#search-methods)
3. [CRUD Operations](#crud-operations)
4. [Domain Syntax](#domain-syntax)
5. [Environment Context](#environment-context)

---

## Recordset Basics

### browse() - Retrieve Records by ID

```python
# Single record
record = self.browse(1)

# Multiple records
records = self.browse([1, 2, 3])

# Empty recordset
empty = self.browse()
```

**Important**: `browse()` always returns a recordset. Use `.exists()` to ensure the records actually exist in the database.

```python
records = self.browse([1, 999])
valid_records = records.exists()  # Filters out ID 999 if it doesn't exist
```

### Empty Recordset Pattern

```python
if not records:
    return

# Use filtered() for conditional operations
active_records = records.filtered(lambda r: r.active)
```

---

## Search Methods

### search() - Find Records

```python
# Basic search - returns recordset
records = self.search([('state', '=', 'draft')])

# With limit and order
records = self.search(
    [('state', '=', 'draft')],
    limit=10,
    order='id desc'
)
```

### search_read() - Find and Read

```python
# Returns list of dicts
data = self.search_read(
    [('state', '=', 'done')],
    ['name', 'date', 'amount']
)
```

### search_count() - Count Records

```python
count = self.search_count([('state', '=', 'draft')])
```

### read_group() - Aggregation

In Odoo 13, `read_group` is the primary method for aggregations.

```python
# Group by category_id and sum amount_total
result = self.read_group(
    domain=[('state', '=', 'draft')],
    fields=['amount_total'],
    groupby=['category_id']
)
# Result: [{'category_id': (1, 'Category A'), 'amount_total': 1500.0, 'category_id_count': 5}]
```

---

## CRUD Operations

### create() - Create New Records

In Odoo 13, `create()` typically accepts a single dictionary, but it's optimized for batch creation when passed a list of dictionaries.

```python
@api.model
def create(self, vals):
    # Standard single create
    return super(MyModel, self).create(vals)

# Batch creation (Odoo 13+)
records = self.create([
    {'name': 'Rec 1'},
    {'name': 'Rec 2'},
])
```

#### Relational Commands (Tuples)

When creating or writing to relational fields (O2M, M2M), use classic tuples:

| Command | Syntax | Description |
|---------|--------|-------------|
| **CREATE** | `(0, 0, vals)` | Create a new record and link it |
| **UPDATE** | `(1, id, vals)` | Update an existing linked record |
| **DELETE** | `(2, id, 0)` | Remove relation and DELETE record from DB |
| **UNLINK** | `(3, id, 0)` | Remove relation (don't delete record) |
| **LINK** | `(4, id, 0)` | Link an existing record |
| **CLEAR** | `(5, 0, 0)` | Unlink all records |
| **SET** | `(6, 0, ids)` | Replace all existing relations with these IDs |

Example:
```python
self.write({
    'line_ids': [
        (0, 0, {'name': 'New Line'}),        # Create
        (6, 0, [10, 11, 12]),               # Replace all
    ]
})
```

### write() - Update Records

```python
@api.multi
def action_done(self):
    return self.write({'state': 'done'})
```

### unlink() - Delete Records

In Odoo 13, validations before deletion are done by overriding `unlink()`.

```python
@api.multi
def unlink(self):
    for record in self:
        if record.state == 'done':
            raise UserError("Cannot delete records in 'done' state.")
    return super(MyModel, self).unlink()
```

---

## Domain Syntax

### Basic Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `=` | equals | `[('state', '=', 'draft')]` |
| `!=` | not equals | `[('state', '!=', 'draft')]` |
| `>` | greater than | `[('amount', '>', 100)]` |
| `in` | in list | `[('id', 'in', [1, 2, 3])]` |
| `ilike` | contains (case-insensitive) | `[('name', 'ilike', 'test')]` |
| `child_of` | is child (in hierarchy) | `[('category_id', 'child_of', category_id)]` |

### Logical Operators

```python
# OR
domain = ['|', ('state', '=', 'draft'), ('state', '=', 'open')]

# AND (Implicit)
domain = [('state', '=', 'open'), ('partner_id', '=', 1)]
```

---

## Environment Context

### sudo() - Bypass Security

```python
# Run as OdooBot (superuser)
self.sudo().write({'active': False})
```

### with_context() - Modify Context

```python
# Pass custom flags
self.with_context(mail_auto_subscribe_no_notifications=True).write(...)

# Change language context
records.with_context(lang='es_ES').read(['name'])
```

### SQL (Classic)

In Odoo 13, direct SQL is performed using `self._cr`.

```python
query = "SELECT id, name FROM res_partner WHERE customer_rank > %s"
self.env.cr.execute(query, (0,))
results = self.env.cr.dictfetchall()
# Returns: [{'id': 1, 'name': 'Partner A'}, ...]
```

---

## Recordset Iteration (@api.multi)

In Odoo 13, `@api.multi` was the standard decorator for methods that act on a recordset.

```python
@api.multi
def my_method(self):
    for record in self:
        print(record.name)
```

- `@api.multi`: `self` is a recordset.
- `@api.model`: `self` is the model (no IDs).
- `@api.one`: `self` is a single record (deprecated, use `@api.multi` + loop).

---

## Recordset Utility Methods

### mapped()

```python
# Get a list of names
names = self.mapped('name')

# Get a recordset of partners
partners = self.mapped('line_ids.partner_id')
```

### filtered()

```python
# Filter records in memory
done_lines = self.line_ids.filtered(lambda l: l.state == 'done')
```

### sorted()

```python
# Sort in memory
sorted_records = self.sorted(key=lambda r: r.date, reverse=True)
```

---

**For more Odoo 13 guides, see [SKILL.md](../SKILL.md)**
