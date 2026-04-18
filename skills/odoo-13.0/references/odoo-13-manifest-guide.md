---
name: odoo-13-manifest
description: Complete reference for Odoo 13 module manifest (__manifest__.py) covering all fields, dependencies, assets loading (XML templates), external dependencies, hooks, auto_install, and module structure.
globs: "**/__manifest__.py"
topics:
  - All __manifest__.py fields
  - Module dependencies and loading order
  - Assets loading (web.assets_backend, etc.) via XML
  - External dependencies (python, bin)
  - Hooks (pre_init, post_init, uninstall)
  - auto_install behavior
  - Module categories
  - License types
when_to_use:
  - Creating new Odoo 13 modules
  - Configuring module dependencies
  - Setting up assets (JS, CSS, SCSS) via templates
  - Declaring external Python/binary dependencies
  - Using module hooks for initialization
---

# Odoo 13 Module Manifest Guide

Complete reference for Odoo 13 `__manifest__.py`: all fields, dependencies, asset loading, hooks, and configuration.

## Table of Contents

1. [Manifest Basics](#manifest-basics)
2. [Core Fields](#core-fields)
3. [Dependencies](#dependencies)
4. [Data Loading](#data-loading)
5. [Assets (XML Templates)](#assets)
6. [External Dependencies](#external-dependencies)
7. [Hooks](#hooks)
8. [Complete Example](#complete-example)

---

## Manifest Basics

### File Name and Location

```
my_module/
├── __init__.py
├── __manifest__.py  # <-- Manifest file
├── models/
├── views/
└── ...
```

### File Name

- Odoo 13: `__manifest__.py`
- Odoo 12 and older: `__openerp__.py` (deprecated)

### Basic Structure

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

{
    'name': 'My Module',
    'version': '13.0.1.0.0',
    'category': 'Tools',
    'summary': 'Short description',
    'description': """
        Long Description
    """,
    'author': 'Author Name',
    'website': 'https://www.example.com',
    'license': 'LGPL-3',
    'depends': ['base'],
    'data': [
        'views/my_views.xml',
        'views/assets.xml',  # Assets are loaded via XML in Odoo 13
    ],
    'qweb': [
        'static/src/xml/my_templates.xml',
    ],
    'demo': [
        'demo/my_demo.xml',
    ],
    'installable': True,
    'application': True,
    'auto_install': False,
}
```

---

## Core Fields

### Required Fields

Only `name` is truly required, but `version` and `depends` should always be specified.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `name` | str | - | Human-readable module name (required) |
| `version` | str | - | Module version (should use semantic versioning) |
| `description` | str | - | Extended description in RST |
| `author` | str | - | Module author name |
| `website` | str | - | Author website URL |
| `license` | str | LGPL-3 | Distribution license |
| `category` | str | Uncategorized | Module category |
| `depends` | list(str) | - | Required modules |
| `data` | list(str) | - | Data files to load |
| `qweb` | list(str) | - | Client-side XML templates |
| `installable` | bool | True | Whether module can be installed |

### name (Required)

```python
'name': 'My Module',
```

The display name shown in Apps menu.

### version

```python
'version': '13.0.1.0.0',
```

In Odoo 13, it's common to prefix with the Odoo version:
- `13.0.MAJOR.MINOR.PATCH`

```python
# Examples
'version': '13.0.1.0.0',      # First release
'version': '13.0.1.1.0',      # Added feature
'version': '13.0.1.1.1',      # Bug fix
```

---

## Dependencies

### depends

```python
'depends': ['base', 'mail', 'web'],
```

Modules that must be loaded before this one.

#### auto_install

```python
'auto_install': True,
```

Automatically install if all dependencies are installed.

---

## Assets (XML Templates)

In Odoo 13, assets (JS, CSS, SCSS) are **NOT** declared in the manifest `assets` key (which was introduced in v15). Instead, they are loaded by inheriting the base asset templates in an XML file (usually `views/assets.xml`).

### Loading Backend Assets

Inherit `web.assets_backend`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="assets_backend" name="my_module assets" inherit_id="web.assets_backend">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/my_module/static/src/js/my_script.js"></script>
            <link rel="stylesheet" type="text/scss" href="/my_module/static/src/scss/my_style.scss"/>
        </xpath>
    </template>
</odoo>
```

### Loading Frontend Assets (Website)

Inherit `web.assets_frontend`:

```xml
<template id="assets_frontend" inherit_id="web.assets_frontend">
    <xpath expr="." position="inside">
        <script type="text/javascript" src="/my_module/static/src/js/portal.js"></script>
    </xpath>
</template>
```

### Loading Common Assets

Inherit `web.assets_common`:

```xml
<template id="assets_common" inherit_id="web.assets_common">
    <xpath expr="." position="inside">
        <link rel="stylesheet" type="text/css" href="/my_module/static/src/css/common.css"/>
    </xpath>
</template>
```

### Client-side XML (QWeb)

Client-side templates used by JavaScript are declared in the `qweb` key of the manifest:

```python
'qweb': [
    'static/src/xml/my_widget_template.xml',
],
```

---

## External Dependencies

### external_dependencies

```python
'external_dependencies': {
    'python': [
        'requests',
    ],
    'bin': [
        'wkhtmltopdf',
    ],
}
```

---

## Hooks

### Hook Functions

```python
'pre_init_hook': 'module_pre_init',
'post_init_hook': 'module_post_init',
'uninstall_hook': 'module_uninstall',
```

These functions must be defined in the module's `__init__.py`.

---

## Complete Example

### Full Manifest (Odoo 13)

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

{
    'name': 'Business Trip Management',
    'version': '13.0.1.0.0',
    'category': 'Tools/Trip Management',
    'summary': 'Manage business trips and expenses',
    'description': """
Business Trip Management for Odoo 13
====================================
* Create and manage business trips
* Track expenses
    """,
    'author': 'My Company',
    'website': 'https://www.example.com',
    'license': 'LGPL-3',

    'depends': [
        'base',
        'mail',
        'web',
        'portal',
    ],

    'data': [
        'security/business_trip_security.xml',
        'security/ir.model.access.csv',
        'views/assets.xml',             # Asset loading template
        'views/business_trip_views.xml',
        'data/business_trip_data.xml',
        'report/trip_report_views.xml',
    ],

    'qweb': [
        'static/src/xml/trip_templates.xml',
    ],

    'demo': [
        'demo/business_trip_demo.xml',
    ],

    'external_dependencies': {
        'python': ['requests'],
        'bin': ['wkhtmltopdf'],
    },

    'installable': True,
    'application': True,
    'auto_install': False,
}
```

---

**For more Odoo 13 guides, see [SKILL.md](../SKILL.md)**
