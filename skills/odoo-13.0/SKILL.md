---
name: odoo-13
description: >
  Odoo 13 development reference for Python models and ORM (search, domain, read_group, compute fields),
  XML/CSV data and views, classic JS framework (Widgets), QWeb reports, security (ACL, record rules, groups),
  cron and server actions, migrations and module upgrades, tests, i18n, and performance.
  Use this skill whenever work involves Odoo 13 or custom addons—even if the user only pastes a traceback,
  mentions addons/ or __manifest__.py, describes form/list/kanban/XML errors, HTTP controllers, or
  business rules on models—including building features, fixing bugs, refactoring, or reviewing addon code.
globs: "**/*.{py,xml,csv,js}"
---

# Odoo 13 Skill - Master Index

Master index for all Odoo 13 development guides. Read the appropriate guide from `references/` based on your task.

## Quick Reference

| Topic | File | When to Use |
|-------|------|-------------|
| Actions | `references/odoo-13-actions-guide.md` | Creating actions, menus, scheduled jobs, server actions |
| API Decorators | `references/odoo-13-decorator-guide.md` | Using @api decorators, compute fields, validation |
| Controllers | `references/odoo-13-controller-guide.md` | Writing HTTP endpoints, routes, web controllers |
| Data Files | `references/odoo-13-data-guide.md` | XML/CSV data files, records, shortcuts |
| Development | `references/odoo-13-development-guide.md` | Creating modules, manifest, reports, security, wizards |
| Field Types | `references/odoo-13-field-guide.md` | Defining model fields, choosing field types |
| Manifest | `references/odoo-13-manifest-guide.md` | __manifest__.py configuration, dependencies, hooks |
| Migration | `references/odoo-13-migration-guide.md` | Upgrading modules, data migration, version changes |
| Mixins | `references/odoo-13-mixins-guide.md` | mail.thread, activities, email aliases, tracking |
| Model Methods | `references/odoo-13-model-guide.md` | Writing ORM queries, CRUD operations, domain filters |
| JS Framework | `references/odoo-13-javascript-guide.md` | Building Widgets, Actions, and classic JS components |
| Performance | `references/odoo-13-performance-guide.md` | Optimizing queries, fixing slow code, preventing N+1 |
| Reports | `references/odoo-13-reports-guide.md` | QWeb reports, PDF/HTML, templates, paper formats |
| Security | `references/odoo-13-security-guide.md` | Access rights, record rules, field permissions |
| Testing | `references/odoo-13-testing-guide.md` | Writing tests, mocking, assertions, browser testing |
| Transactions | `references/odoo-13-transaction-guide.md` | Handling database errors, savepoints, UniqueViolation |
| Translation | `references/odoo-13-translation-guide.md` | Adding translations, localization, i18n |
| Views & XML | `references/odoo-13-view-guide.md` | Writing XML views, actions, menus, QWeb templates |

## File Structure

```
skills/odoo-13.0/
├── SKILL.md                          # This file - master index
└── references/                       # Development guides
    ├── odoo-13-actions-guide.md
    ├── odoo-13-controller-guide.md
    ├── odoo-13-data-guide.md
    ├── odoo-13-decorator-guide.md
    ├── odoo-13-development-guide.md
    ├── odoo-13-field-guide.md
    ├── odoo-13-manifest-guide.md
    ├── odoo-13-migration-guide.md
    ├── odoo-13-mixins-guide.md
    ├── odoo-13-model-guide.md
    ├── odoo-13-javascript-guide.md
    ├── odoo-13-performance-guide.md
    ├── odoo-13-reports-guide.md
    ├── odoo-13-security-guide.md
    ├── odoo-13-testing-guide.md
    ├── odoo-13-transaction-guide.md
    ├── odoo-13-translation-guide.md
    └── odoo-13-view-guide.md
```

## Base Code Reference (Odoo 13)

All guides are based on analysis of Odoo 13 source code:
- `odoo/models.py` - ORM implementation
- `odoo/fields.py` - Field types
- `odoo/api.py` - Decorators
- `odoo/http.py` - HTTP layer
- `odoo/exceptions.py` - Exception types
- `odoo/tools/translate.py` - Translation system
- `odoo/addons/base/models/res_lang.py` - Language model
- `addons/web/static/src/js/core/translation.js` - JS translations

## External Documentation

- [Odoo 13 Official Documentation](https://www.odoo.com/documentation/13.0/)
- [Odoo 13 Developer Reference](https://www.odoo.com/documentation/13.0/developer/reference/orm.html)
