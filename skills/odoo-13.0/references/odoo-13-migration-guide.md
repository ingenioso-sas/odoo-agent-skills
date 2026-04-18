---
name: odoo-13-migration
description: Comprehensive guide for upgrading modules and data to Odoo 13,
  including migration scripts, upgrade hooks, and best practices.
globs: "**/migrations/**/*.py"
topics:
  - Migration script structure (pre/post/end)
  - Module upgrade hooks (pre_init, post_init, uninstall)
  - Data migration with SQL and ORM (v12 to v13)
  - Version checks and upgrade tooling
when_to_use:
  - Creating or reviewing migration scripts
  - Upgrading modules from Odoo 12 to Odoo 13
  - Handling data migration and cleanup
---

# Odoo 13 Migration Guide

Comprehensive guide for migrating modules and data to Odoo 13, covering migration scripts, upgrade hooks, and best practices.

## Table of Contents

1. [Migration Script Structure](#migration-script-structure)
2. [Module Upgrade Hooks](#module-upgrade-hooks)
3. [Migration Stages](#migration-stages)
4. [Migration Best Practices](#migration-best-practices)
5. [Real-World Examples (v12 to v13)](#real-world-examples)
6. [Version Management](#version-management)

---

## Migration Script Structure

### Directory Layout

Migration scripts are organized in versioned directories within your module:

```python
<module>/
├── __init__.py
├── __manifest__.py
├── models/
├── migrations/
│   ├── 12.0.1.0/          # Migrating from v12
│   ├── 13.0.1.0/          # Odoo 13.0 specific
│   └── 0.0.0/             # Runs on any version change
```

### Migration Script Function Signature

```python
# File: migrations/13.0.1.0/pre-migrate_data.py

def migrate(cr, version):
    """
    Migration script for Odoo 13.0

    Args:
        cr: Database cursor (SQL operations)
        version: Previously installed version (None for new installs)
    """
    if version is None:
        return  # New installation, skip migration

    # Your migration code here
    cr.execute("""
        UPDATE your_model
        SET field_name = 'new_value'
        WHERE condition = true
    """)
```

---

## Module Upgrade Hooks

### Manifest Hooks (`__manifest__.py`)

```python
# File: __manifest__.py

{
    'name': 'My Module',
    'version': '13.0.1.0',

    # Hooks executed during module lifecycle
    'pre_init_hook': 'pre_init_function',      # Before installation
    'post_init_hook': 'post_init_function',     # After installation
    'uninstall_hook': 'uninstall_function',     # Before uninstallation
}
```

### Hook Function Signatures (in `__init__.py`)

```python
# pre_init_hook(cr)
def pre_init_function(cr):
    # SQL only
    pass

# post_init_hook(cr, registry)
def post_init_function(cr, registry):
    from odoo import api, SUPERUSER_ID
    env = api.Environment(cr, SUPERUSER_ID, {})
    # ORM available
    pass
```

---

## Migration Stages

### Pre-Stage (`pre-*.py`)

Runs **before** module initialization:
- Models are not loaded.
- Use raw SQL for schema changes or data preparation.

```python
def migrate(cr, version):
    # Add new column if it doesn't exist
    cr.execute("ALTER TABLE my_table ADD COLUMN IF NOT EXISTS new_col INTEGER")
```

### Post-Stage (`post-*.py`)

Runs **after** module initialization:
- Models are loaded.
- Can use ORM (`api.Environment`).

```python
def migrate(cr, version):
    from odoo import api, SUPERUSER_ID
    env = api.Environment(cr, SUPERUSER_ID, {})
    # Use env['my.model']
```

---

## Migration Best Practices

### 1. Use Raw SQL for Large Datasets
ORM can be slow for thousands of records during migration.

### 2. Check for `version is None`
Always ensure your script handles new installations gracefully.

### 3. Idempotency
Ensure scripts can be run multiple times without failing (e.g., `IF NOT EXISTS`).

---

## Real-World Examples (v12 to v13)

### Example: Migrating from `api.one` (Legacy) to `api.multi` logic
In Odoo 13, all methods are recordset-based. Migration scripts might need to fix data that was incorrectly stored due to old API usage.

```python
def migrate(cr, version):
    # Example: Ensure all records have a required field that was added in v13
    cr.execute("""
        UPDATE account_move
        SET type = 'entry'
        WHERE type IS NULL
    """)
```

---

## Version Management

### Version Parsing

```python
from odoo.tools.parse_version import parse_version

if parse_version(version) < parse_version('13.0.1.0'):
    # Do something
    pass
```

---

**For more Odoo 13 guides, see [SKILL.md](../SKILL.md)**
