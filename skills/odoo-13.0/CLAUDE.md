# Odoo 13 Development Guide

This file provides guidance to AI agents when working with Odoo 13 code in this repository.

> **For setup instructions with different AI IDEs, see [AGENTS.md](./AGENTS.md)**

## Documentation Structure

The `skills/odoo-13.0/references/` directory contains modular guides for Odoo 13 development:

```
skills/odoo-13.0/
├── SKILL.md                       # Master index
├── references/                    # Development guides (18 files)
│   ├── odoo-13-actions-guide.md     # ir.actions.*, cron, bindings
│   ├── odoo-13-controller-guide.md  # HTTP, routing, controllers
│   ├── odoo-13-data-guide.md        # XML/CSV data files, records
│   ├── odoo-13-decorator-guide.md   # @api decorators
│   ├── odoo-13-development-guide.md # Manifest, wizards (overview)
│   ├── odoo-13-field-guide.md       # Field types, parameters
│   ├── odoo-13-manifest-guide.md    # __manifest__.py reference
│   ├── odoo-13-mixins-guide.md      # mail.thread, activities, etc.
│   ├── odoo-13-model-guide.md       # ORM, CRUD, search, domain
│   ├── odoo-13-migration-guide.md   # Migration scripts, hooks
│   ├── odoo-13-javascript-guide.md  # Classic JS Widgets, RPC
│   ├── odoo-13-performance-guide.md # N+1 prevention, optimization
│   ├── odoo-13-reports-guide.md     # QWeb reports, PDF/HTML
│   ├── odoo-13-security-guide.md    # ACL, record rules, security
│   ├── odoo-13-testing-guide.md     # Test classes, decorators
│   ├── odoo-13-transaction-guide.md # Savepoints, errors
│   ├── odoo-13-translation-guide.md # Translations, i18n
│   └── odoo-13-view-guide.md        # XML views, QWeb
├── CLAUDE.md                      # This file
└── AGENTS.md                      # AI agents setup
```

## Which Guide to Use

| Task | Guide |
|------|-------|
| Creating actions, menus, cron jobs | `references/odoo-13-actions-guide.md` |
| Creating a new module | `references/odoo-13-development-guide.md` |
| Configuring __manifest__.py | `references/odoo-13-manifest-guide.md` |
| Creating XML/CSV data files | `references/odoo-13-data-guide.md` |
| Writing ORM queries/search | `references/odoo-13-model-guide.md` |
| Defining model fields | `references/odoo-13-field-guide.md` |
| Using @api decorators | `references/odoo-13-decorator-guide.md` |
| Writing XML views | `references/odoo-13-view-guide.md` |
| Fixing slow code/N+1 queries | `references/odoo-13-performance-guide.md` |
| Handling database errors | `references/odoo-13-transaction-guide.md` |
| Creating HTTP endpoints | `references/odoo-13-controller-guide.md` |
| Building JS Widgets/UI | `references/odoo-13-javascript-guide.md` |
| Upgrading modules/migrating data | `references/odoo-13-migration-guide.md` |
| Using mail.thread, activities, mixins | `references/odoo-13-mixins-guide.md` |
| Creating QWeb reports | `references/odoo-13-reports-guide.md` |
| Configuring security (ACL, rules) | `references/odoo-13-security-guide.md` |
| Writing tests | `references/odoo-13-testing-guide.md` |
| Adding translations/localization | `references/odoo-13-translation-guide.md` |

## Key Odoo 13 Conventions

| Feature | Pattern (Odoo 13) | Notes |
|---------|-------------------|-------|
| List view tag | `<tree>` | Use `tree` (v18 uses `list`) |
| Dynamic attributes | `attrs="{'invisible': [...]}"` | REQUIRED (v18 uses direct attributes) |
| Delete validation | Override `unlink()` | No `@api.ondelete` decorator |
| Field aggregation | `group_operator=` | Use `group_operator` (v18 uses `aggregator`) |
| SQL queries | `cr.execute(query, params)` | No `SQL` class |
| Batch create | `create([vals1, vals2])` | Supported since v12 |
| JS Framework | Backbone-style Widgets | No OWL (introduced later) |
| Relation commands | `(0, 0, vals)` | Use tuplas (v18 uses `odoo.fields.Command`) |

## Critical Anti-Patterns

| Anti-Pattern | Why Bad | Correct Approach |
|--------------|---------|------------------|
| `invisible="partner_id == False"` | Direct attributes not supported | Use `attrs="{'invisible': [('partner_id', '=', False)]}"` |
| `@api.depends('partner_id')` then accessing `partner_id.email` | N queries per record | Add `@api.depends('partner_id.email')` |
| `search()` inside loop | N+1 queries | Use `search()` with `IN` domain or `read_group()` |
| `create()` in loop | N INSERT statements | Batch: `create([{...}, {...}])` |
| Using `odoo.fields.Command` | Module doesn't exist in v13 | Use tuplas: `(0, 0, vals)`, `(6, 0, ids)` |
| Using `<list>` tag | Invalid tag in v13 | Use `<tree>` instead |

## @api Decorator Decision Tree

```
Need to define field behavior?
├── Field computed from other fields → @api.depends
│   └── CAN use dotted paths: `@api.depends('partner_id.email')`
├── Validate data → @api.constrains
│   └── CANNOT use dotted paths: only simple field names
└── Update form UI → @api.onchange
    └── NO CRUD operations allowed

Need to define method behavior?
├── Method-level, doesn't depend on self → @api.model
├── Multi-record operations (v13 standard) → @api.multi (optional but common)
└── Normal record method → no decorator needed
```

## Common Patterns Reference

### N+1 Query Prevention

```python
# BAD: search in loop
for order in orders:
    payments = self.env['payment'].search([('order_id', '=', order.id)])

# GOOD: single query
payments = self.env['payment'].search_read([('order_id', 'in', orders.ids)])
```

### List View (Odoo 13)

```xml
<tree string="Records" editable="bottom" multi_edit="1">
    <field name="state" decoration-success="state == 'done'"/>
    <field name="phone" optional="show"/>
</tree>
```

### Delete Validation (Odoo 13)

```python
def unlink(self):
    if any(rec.state != 'draft' for rec in self):
        raise UserError("Cannot delete non-draft records")
    return super(MyModel, self).unlink()
```

## Module Structure

```
my_module/
├── __init__.py
├── __manifest__.py
├── models/
│   ├── __init__.py
│   └── my_model.py
├── views/
│   ├── my_model_views.xml
│   └── templates.xml (for assets)
├── security/
│   ├── ir.model.access.csv
│   └── my_module_security.xml
├── data/
│   └── my_module_data.xml
├── migrations/
│   └── 13.0.1.0/
│       └── post-migrate_data.py
├── tests/
│   ├── __init__.py
│   └── test_my_model.py
├── wizard/
│   ├── __init__.py
│   └── my_wizard.py
├── controllers/
│   ├── __init__.py
│   └── my_controller.py
└── static/
    └── src/
        ├── js/
        │   └── my_widget.js
        ├── xml/
        │   └── my_template.xml
        └── scss/
            └── my_styles.scss
```

## Base Code Reference

The guides are based on Odoo 13 source code. Reference these files in your Odoo installation:
- `odoo/models.py` - ORM implementation
- `odoo/fields.py` - Field types
- `odoo/api.py` - Decorators
- `odoo/http.py` - HTTP layer
- `odoo/exceptions.py` - Exception types
