---
name: odoo-13-security
description: Complete reference for Odoo 13 security covering access rights (ACL), record rules, field-level access, security pitfalls, SQL injection prevention, XSS prevention, and safe coding practices.
globs: "**/*.{py,xml,csv}"
topics:
  - Access rights (ir.model.access.csv)
  - Record rules (ir.rule)
  - Field-level access (groups attribute)
  - Security pitfalls (SQL injection, XSS, eval)
  - User groups and categories
  - ACL vs Record Rules interaction
  - Public/Portal user security
when_to_use:
  - Configuring security for new models
  - Setting up access rights CSV
  - Creating record rules
  - Preventing security vulnerabilities
  - Understanding multi-company security
  - Implementing field-level permissions
---

# Odoo 13 Security Guide

Complete reference for Odoo 13 security: access rights, record rules, field access, and preventing security pitfalls.

## Table of Contents

1. [Security Overview](#security-overview)
2. [User Groups](#user-groups)
3. [Access Rights (ACL)](#access-rights-acl)
4. [Record Rules](#record-rules)
5. [Field-Level Access](#field-level-access)
6. [Security Pitfalls](#security-pitfalls)

---

## Security Overview

### Two-Layer Security

Odoo provides two main data-driven security mechanisms:

| Layer | Mechanism | Purpose |
|-------|-----------|---------|
| 1 | Access Rights (ACL) | Grants access to an entire model for operations |
| 2 | Record Rules | Restricts which specific records can be accessed |

Both are linked to users through **groups**.

---

## User Groups

Groups are the foundation of Odoo security.

```xml
<record id="group_trip_manager" model="res.groups">
    <field name="name">Trip Manager</field>
    <field name="category_id" ref="base.module_category_trip_management"/>
    <field name="implied_ids" eval="[(4, ref('base.group_user'))]"/>
</record>
```

### Checking Groups in Code

```python
# Check if user has group
if self.env.user.has_group('my_module.group_manager'):
    pass
```

---

## Access Rights (ACL)

### ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_trip_user,trip.user,model_business_trip,base.group_user,1,0,0,0
access_trip_manager,trip.manager,model_business_trip,group_trip_manager,1,1,1,1
```

---

## Record Rules

### ir.rule - Record-Level Security

```xml
<record id="trip_personal_rule" model="ir.rule">
    <field name="name">Personal Trips</field>
    <field name="model_id" ref="model_business_trip"/>
    <field name="domain_force">[('user_id', '=', user.id)]</field>
    <field name="groups" eval="[(4, ref('base.group_user'))]"/>
</record>
```

### Domain Force Variables (Odoo 13)

Available variables in `domain_force`:

| Variable | Description |
|----------|-------------|
| `user` | Current user record |
| `company_id` | Current company ID |
| `company_ids` | Allowed company IDs |

---

## Field-Level Access

Restrict access to specific fields using the `groups` parameter:

```python
class BusinessTrip(models.Model):
    _name = 'business.trip'

    secret_code = fields.Char(groups='my_module.group_manager')
```

---

## Security Pitfalls

### 1. SQL Injection

**Problem**: String concatenation in SQL queries.

```python
# VERY BAD: SQL injection vulnerability
query = "SELECT id FROM table WHERE name = '%s'" % user_input
self.env.cr.execute(query)

# GOOD: Use parameterized queries
self.env.cr.execute("SELECT id FROM table WHERE name = %s", (user_input,))
```

### 2. Unescaped Content (XSS)

**Problem**: Using `t-raw` with user-provided content.

```xml
<!-- BAD: t-raw with user content -->
<div t-raw="info_message"/>

<!-- GOOD: t-esc auto-escapes -->
<div t-esc="info_message"/>
```

### 3. sudo() Overuse

**Problem**: `sudo()` bypasses all security. Use it only when absolutely necessary to bypass access rights for specific technical operations.

---

## Security Debugging (Odoo 13)

```python
# Check access rights
model.check_access_rights('read')
model.check_access_rights('write', raise_exception=False)

# Check record access
record.check_access_rule('read')
```

---

**For more Odoo 13 guides, see [SKILL.md](../SKILL.md)**
