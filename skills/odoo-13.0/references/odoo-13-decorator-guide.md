---
name: odoo-13-decorator
description: "Complete reference for Odoo 13 API decorators (@api.model, @api.multi, @api.depends, @api.constrains, @api.onchange, @api.returns) and their proper usage patterns."
globs: "**/models/**/*.py"
topics:
  - api.model (model-level methods)
  - api.multi (recordset methods)
  - api.depends (computed fields)
  - api.depends_context (context-dependent computed fields)
  - api.constrains (data validation)
  - api.onchange (form UI updates)
  - api.returns (return type specification)
  - Decorator combinations and decision tree
when_to_use:
  - Writing computed fields
  - Implementing data validation
  - Creating form onchange handlers
  - Preventing record deletion (unlink override)
  - Defining model methods
---

# Odoo 13 Decorator Guide

Complete reference for Odoo 13 API decorators and their proper usage.

## Table of Contents

1. [@api.model](#api-model)
2. [@api.multi](#api-multi)
3. [@api.depends](#api-depends)
4. [@api.depends_context](#api-depends_context)
5. [@api.constrains](#api-constrains)
6. [@api.onchange](#api-onchange)
7. [@api.returns](#api-returns)
8. [Overriding unlink (Deletion Validation)](#overriding-unlink)

---

## @api.model

**Purpose**: Decorate methods where `self` is a recordset, but the actual records don't matter - only the model class.

```python
from odoo import api, models

class SaleOrder(models.Model):
    _name = 'sale.order'

    @api.model
    def get_default_values(self):
        """Return default values for new orders"""
        return {
            'state': 'draft',
            'date_order': fields.Datetime.now(),
        }

    @api.model
    def create(self, vals):
        """Standard create override"""
        # vals is a dict
        return super(SaleOrder, self).create(vals)
```

**When to use**:
- Factory methods that create records
- Methods that don't depend on `self` content
- Utility methods for the model

---

## @api.multi

**Purpose**: Explicitly decorate methods that act on a recordset. In Odoo 13, all methods are "multi" by default, but this decorator is often kept for clarity and backward compatibility.

```python
from odoo import api, models

class SaleOrder(models.Model):
    _name = 'sale.order'

    @api.multi
    def action_confirm(self):
        """Action on multiple records"""
        for order in self:
            order.state = 'sale'
        return True
```

**When to use**:
- Methods called from buttons in tree/form views
- Actions that process multiple records at once
- Standard practice for most recordset methods in v13

---

## @api.depends

**Purpose**: Declare dependencies for computed fields. The method is re-computed when any dependency changes.

```python
from odoo import api, fields, models

class SaleOrder(models.Model):
    _name = 'sale.order'

    amount_untaxed = fields.Float(string='Untaxed Amount')
    tax_amount = fields.Float(string='Tax Amount')

    # Basic depends
    amount_total = fields.Float(
        string='Total',
        compute='_compute_amount_total',
        store=True,
    )

    @api.depends('amount_untaxed', 'tax_amount')
    def _compute_amount_total(self):
        for order in self:
            order.amount_total = order.amount_untaxed + order.tax_amount
```

**Relational field dependencies**:
```python
@api.depends('partner_id.name', 'partner_id.email')
def _compute_partner_display(self):
    for order in self:
        if order.partner_id:
            order.partner_display = "%s <%s>" % (order.partner_id.name, order.partner_id.email)
        else:
            order.partner_display = ''
```

**Important rules**:
1. **Must list all dependencies** - missed dependencies cause stale values
2. **Dot notation for relations** - `partner_id.name` not just `partner_id`
3. **No dotted path in @constrains** - only @api.depends supports dotted paths

---

## @api.depends_context

**Purpose**: Make computed field depend on context values. Field recomputed when context changes (introduced in Odoo 13).

```python
from odoo import api, fields, models

class ProductProduct(models.Model):
    _name = 'product.product'

    # Price depends on pricelist in context
    price = fields.Float(
        string='Price',
        compute='_compute_price',
    )

    @api.depends_context('pricelist')
    def _compute_price(self):
        pricelist_id = self.env.context.get('pricelist')
        if pricelist_id:
            pricelist = self.env['product.pricelist'].browse(pricelist_id)
            for product in self:
                product.price = pricelist.get_product_price(product, 1.0)
        else:
            for product in self:
                product.price = product.list_price
```

**Built-in context keys**:
- `company`
- `uid`
- `lang`

---

## @api.constrains

**Purpose**: Validate data integrity. Raise `ValidationError` if validation fails.

```python
from odoo import api, models
from odoo.exceptions import ValidationError

class SaleOrder(models.Model):
    _name = 'sale.order'

    @api.constrains('date_order', 'date_validity')
    def _check_dates(self):
        for order in self:
            if order.date_validity and order.date_order > order.date_validity:
                raise ValidationError("Order date cannot be after validity date.")
```

---

## @api.onchange

**Purpose**: Update form fields dynamically when another field changes.

```python
from odoo import api, models

class SaleOrderLine(models.Model):
    _name = 'sale.order.line'

    product_id = fields.Many2one('product.product', string='Product')
    price_unit = fields.Float(string='Unit Price')

    @api.onchange('product_id')
    def _onchange_product_id(self):
        if self.product_id:
            self.price_unit = self.product_id.list_price
        else:
            self.price_unit = 0.0
```

**Return warning**:
```python
@api.onchange('discount')
def _onchange_discount(self):
    if self.discount > 50:
        return {
            'warning': {
                'title': "High Discount",
                'message': "Discount over 50% requires approval.",
            }
        }
```

---

## @api.returns

**Purpose**: Specify the return model of a method for API compatibility.

```python
from odoo import api, models

class SaleOrder(models.Model):
    _name = 'sale.order'

    @api.returns('res.partner')
    def get_partner(self):
        """Returns partner record(s)"""
        return self.mapped('partner_id')
```

---

## Overriding unlink (Deletion Validation)

In Odoo 13, to prevent record deletion, you override the `unlink()` method.

```python
from odoo import api, models
from odoo.exceptions import UserError

class SaleOrder(models.Model):
    _name = 'sale.order'

    @api.multi
    def unlink(self):
        for order in self:
            if order.state not in ('draft', 'cancel'):
                raise UserError("You can only delete draft or cancelled orders.")
        return super(SaleOrder, self).unlink()
```

---

## @api.model_create_multi

**Purpose**: Decorate batch create method. Recommended for performance when creating multiple records.

```python
from odoo import api

@api.model_create_multi
def create(self, vals_list):
    """Batch create - receives list of vals, returns recordset"""
    for vals in vals_list:
        vals.setdefault('state', 'draft')
    return super(SaleOrder, self).create(vals_list)
```

---

## All API Decorators Reference (Odoo 13)

| Decorator | Purpose |
|----------|---------|
| `@api.model` | Model-level method (self not relevant) |
| `@api.multi` | Recordset method (standard for v13) |
| `@api.depends` | Computed field dependencies |
| `@api.depends_context` | Context dependencies |
| `@api.constrains` | Data validation |
| `@api.onchange` | Form UI updates |
| `@api.returns` | Return type specification |
| `@api.model_create_multi` | Batch create (v12+) |

---

## Decorator Decision Tree (Odoo 13)

```
Need to define field behavior?
├── Field value comes from other fields → @api.depends
│   └── Depends on context → also @api.depends_context
├── Validate data integrity → @api.constrains
├── Form UI update → @api.onchange
│
Need method behavior?
├── Doesn't depend on self records → @api.model
├── Acts on recordset → @api.multi
├── Returns specific model → @api.returns
└── Normal record method → @api.multi (recommended)
```
