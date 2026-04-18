---
name: odoo-13-field
description: Complete reference for Odoo 13 field types, parameters, and when to use each. Covers simple fields, relational fields, computed fields, and Odoo 13 specific parameters like group_operator and compute_sudo.
globs: "**/models/**/*.py"
topics:
  - Simple fields (Char, Text, Html, Boolean, Integer, Float, Monetary, Date, Datetime, Binary, Selection, Reference)
  - Relational fields (Many2one, One2many, Many2many)
  - Computed fields (compute, store, search, inverse)
  - Related fields
  - Field parameters (index, default, copy, store, groups, company_dependent, tracking)
  - Float and Date utilities in Odoo 13
when_to_use:
  - Defining new model fields for Odoo 13
  - Choosing appropriate field types
  - Configuring computed fields
  - Setting up relational fields
  - Optimizing field parameters
---

# Odoo 13 Field Guide

Complete reference for Odoo 13 field types, parameters, and when to use each.

## Table of Contents

1. [Simple Fields](#simple-fields)
2. [Relational Fields](#relational-fields)
3. [Computed Fields](#computed-fields)
4. [Related Fields](#related-fields)
5. [Field Parameters](#field-parameters)

---

## Simple Fields

### Char - String Field

```python
name = fields.Char(
    string='Name',
    required=True,
    size=100,  # Max length
    translate=True,
    default='',
)
```

### Text - Long Text Field

```python
description = fields.Text(string='Description', translate=True)
```

### Boolean - True/False Field

```python
active = fields.Boolean(string='Active', default=True)
```

### Integer - Whole Number Field

```python
quantity = fields.Integer(string='Quantity', default=1)
```

### Float - Decimal Number Field

```python
price = fields.Float(string='Price', digits='Product Price')
weight = fields.Float(string='Weight', digits=(16, 3))
```

#### Float Utilities (Odoo 13)
In Odoo 13, use `odoo.tools.float_utils`:

```python
from odoo.tools import float_compare, float_is_zero, float_round

# Round to precision
rounded = float_round(value, precision_rounding=0.01)

# Check if zero
is_zero = float_is_zero(value, precision_rounding=0.01)

# Compare values
res = float_compare(val1, val2, precision_rounding=0.01)
```

### Monetary - Currency Field

```python
amount = fields.Monetary(string='Amount', currency_field='currency_id')
currency_id = fields.Many2one('res.currency', string='Currency')
```

### Date / Datetime

```python
date = fields.Date(string='Date', default=fields.Date.context_today)
datetime = fields.Datetime(string='DateTime', default=fields.Datetime.now)
```

#### Date Helpers (Odoo 13)
In Odoo 13, use `relativedelta` for date math:

```python
from dateutil.relativedelta import relativedelta

# Add 1 month
next_month = fields.Date.from_string(fields.Date.today()) + relativedelta(months=1)
```

### Binary - File/Attachment Field

```python
file = fields.Binary(string='File', attachment=True)
```

### Image (Introduced in Odoo 13)

```python
image_1920 = fields.Image("Image", max_width=1920, max_height=1920)
image_128 = fields.Image("Image 128", related="image_1920", max_width=128, max_height=128, store=True)
```

---

## Relational Fields

### Many2one - Many-to-One

```python
partner_id = fields.Many2one(
    'res.partner', 
    string='Partner', 
    ondelete='cascade', 
    required=True
)
```

### One2many - One-to-Many

```python
line_ids = fields.One2many(
    'my.model.line', 
    'parent_id', 
    string='Lines'
)
```

### Many2many - Many-to-Many

```python
tag_ids = fields.Many2many(
    'my.tag', 
    'my_model_tag_rel', 
    'model_id', 
    'tag_id', 
    string='Tags'
)
```

---

## Computed Fields

```python
amount_total = fields.Float(compute='_compute_total', store=True)

@api.depends('line_ids.price')
def _compute_total(self):
    for record in self:
        record.amount_total = sum(record.line_ids.mapped('price'))
```

---

## Field Parameters

| Parameter | Description |
|-----------|-------------|
| `string` | Display label |
| `required` | Validation |
| `readonly` | UI read-only |
| `index` | Database index |
| `default` | Default value |
| `copy` | Copy on duplicate |
| `store` | Store in DB |
| `groups` | XML IDs for access |
| `company_dependent` | Property field |
| `tracking` | Track in chatter (v13+) |
| `group_operator` | SQL aggregation (e.g., 'sum') |
| `compute_sudo` | Compute as superuser |

---

**For more Odoo 13 guides, see [SKILL.md](../SKILL.md)**
