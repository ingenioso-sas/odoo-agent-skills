---
name: odoo-13-actions
description: Complete reference for Odoo 13 actions (ir.actions.*). Covers window actions, URL actions, server actions, report actions, client actions, scheduled actions, and action bindings.
globs: "**/*.{py,xml}"
topics:
  - Window actions (ir.actions.act_window)
  - URL actions (ir.actions.act_url)
  - Server actions (ir.actions.server)
  - Report actions (ir.actions.report)
  - Client actions (ir.actions.client)
  - Scheduled actions (ir.cron)
  - Action bindings (binding_model_id, binding_type)
when_to_use:
  - Creating menu items and action buttons in Odoo 13
  - Defining window actions for models
  - Setting up scheduled/cron jobs
  - Configuring server actions for automation
  - Creating report actions
  - Implementing client-side actions
---

# Odoo 13 Actions Guide

Complete reference for Odoo 13 actions: window, URL, server, report, client, and scheduled actions with bindings.

## Table of Contents

1. [Action Basics](#action-basics)
2. [Window Actions](#window-actions)
3. [URL Actions](#url-actions)
4. [Server Actions](#server-actions)
5. [Report Actions](#report-actions)
6. [Client Actions](#client-actions)
7. [Scheduled Actions](#scheduled-actions)
8. [Action Bindings](#action-bindings)

---

## Action Basics

### What are Actions?

Actions define the behavior of the system in response to user actions: login, action button, selection of an invoice, etc.

### Common Action Attributes

All actions share these mandatory attributes:

| Attribute | Type | Description |
|-----------|------|-------------|
| `type` | string | Category of the action (determines available fields) |
| `name` | string | Short user-readable description |

---

## Window Actions

### `ir.actions.act_window` - Most Common Action

The most common action type, used to present visualizations of a model through views.

#### Window Action Fields

| Field | Type | Description |
|-------|------|-------------|
| `res_model` | string | Model to present views for (required) |
| `view_mode` | string | Comma-separated view types: `tree,form,kanban` |
| `res_id` | int | Record to load for form views (optional) |
| `target` | string | Where to open: `current`, `new`, `main` |
| `context` | dict | Additional context data for views |
| `domain` | list | Filtering domain for search queries |

#### View Types (Odoo 13)

| Type | Description |
|------|-------------|
| `tree` | List/Table view |
| `form` | Form view |
| `kanban` | Kanban view |
| `graph` | Graph view |
| `pivot` | Pivot view |
| `calendar` | Calendar view |
| `search` | Search view |

### Window Action Examples

#### Basic Tree and Form Views

```xml
<record id="action_customer" model="ir.actions.act_window">
    <field name="name">Customers</field>
    <field name="res_model">res.partner</field>
    <field name="view_mode">tree,form</field>
    <field name="domain">[('customer_rank', '>', 0)]</field>
</record>
```

#### Using Dictionary (Python)

```python
return {
    "type": "ir.actions.act_window",
    "res_model": "res.partner",
    "view_mode": "tree,form",
    "domain": [["customer_rank", ">", 0]],
}
```

#### Open Specific Record in Dialog

```python
return {
    "type": "ir.actions.act_window",
    "res_model": "product.product",
    "view_mode": "form",
    "res_id": product_id,
    "target": "new",
}
```

---

## URL Actions

### `ir.actions.act_url` - Open Web Pages

Allow opening a URL (website/web page) via an Odoo action.

```xml
<record id="action_open_documentation" model="ir.actions.act_url">
    <field name="name">Documentation</field>
    <field name="url">https://www.odoo.com</field>
    <field name="target">new</field>
</record>
```

---

## Server Actions

### `ir.actions.server` - Execute Python Code

Allow triggering complex server code from any valid action location.

#### Server Action Example (Code)

```xml
<record model="ir.actions.server" id="action_mark_done">
    <field name="name">Mark as Done</field>
    <field name="model_id" ref="model_my_model"/>
    <field name="state">code</field>
    <field name="code">
        records.write({'state': 'done'})
    </field>
</record>
```

### Evaluation Context

Available variables in server action code:
- `model`: Model object
- `record` / `records`: Current record(s)
- `env`: Environment
- `UserError`: For raising errors

---

## Report Actions

### `ir.actions.report` - Print Reports

```xml
<record id="report_sale_order" model="ir.actions.report">
    <field name="name">Quotation / Order</field>
    <field name="model">sale.order</field>
    <field name="report_type">qweb-pdf</field>
    <field name="report_name">sale.report_saleorder</field>
    <field name="binding_model_id" ref="model_sale_order"/>
</record>
```

---

## Scheduled Actions

### `ir.cron` - Automated Actions

```xml
<record id="ir_cron_cleanup" model="ir.cron">
    <field name="name">Cleanup Task</field>
    <field name="model_id" ref="model_my_model"/>
    <field name="state">code</field>
    <field name="code">model._action_cleanup()</field>
    <field name="interval_number">1</field>
    <field name="interval_type">days</field>
    <field name="numbercall">-1</field>
</record>
```

---

## Action Bindings

In Odoo 13, use `binding_model_id` to add actions to the "Action" or "Print" menus.

```xml
<record id="action_server_binding" model="ir.actions.server">
    <field name="name">Batch Process</field>
    <field name="model_id" ref="model_my_model"/>
    <field name="binding_model_id" ref="model_my_model"/>
    <field name="state">code</field>
    <field name="code">records.do_something()</field>
</record>
```

---

**For more Odoo 13 guides, see [SKILL.md](../SKILL.md)**
