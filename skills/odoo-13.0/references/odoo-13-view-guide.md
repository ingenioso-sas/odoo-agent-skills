---
name: odoo-13-view
description: Complete reference for Odoo 13 XML views, actions, menus, and QWeb templates. Covers tree, form, search, kanban, graph, pivot, calendar views and Odoo 13 conventions.
globs: "**/views/**/*.xml"
topics:
  - View types (tree, form, search, kanban, graph, pivot, calendar, activity)
  - Tree view features (editable, decoration, optional fields, widgets)
  - Form view structure (sheet, button box, notebook, chatter)
  - Search view features (fields, filters, group by)
  - Kanban view (templates, cards)
  - Actions (window, server, client, report)
  - Menus (structure, attributes)
  - View inheritance (xpath, position)
  - QWeb templates
when_to_use:
  - Writing XML views for Odoo 13
  - Creating actions and menus
  - Implementing view inheritance via XPath
  - Building QWeb templates
---

# Odoo 13 View Guide

Complete reference for Odoo 13 XML views, actions, menus, and QWeb templates.

## Table of Contents

1. [View Types](#view-types)
2. [Tree View (List)](#tree-view-list)
3. [Form View](#form-view)
4. [Search View](#search-view)
5. [Kanban View](#kanban-view)
6. [Graph & Pivot Views](#graph--pivot-views)
7. [Calendar View](#calendar-view)
8. [Actions](#actions)
9. [Menus](#menus)
10. [View Inheritance](#view-inheritance)

---

## View Types

| Type | XML Tag | Use For |
|------|---------|---------|
| `tree` | `<tree>` | Table/List view |
| `form` | `<form>` | Single record edit/view |
| `search` | `<search>` | Search panel and filters |
| `kanban` | `<kanban>` | Card-based view |
| `graph` | `<graph>` | Bar/line/pie charts |
| `pivot` | `<pivot>` | Pivot table |
| `calendar` | `<calendar>` | Calendar view |
| `activity` | `<activity>` | Activity/messaging view |

---

## Tree View (List)

### Basic Tree View

```xml
<record id="view_my_model_tree" model="ir.ui.view">
    <field name="name">my.model.tree</field>
    <field name="model">my.model</field>
    <field name="arch" type="xml">
        <tree string="My Records">
            <field name="name"/>
            <field name="date"/>
            <field name="state"/>
        </tree>
    </field>
</record>
```

### Tree View Features

```xml
<tree string="My Records"
      multi_edit="1"          # Enable inline edit
      editable="bottom"       # Edit mode (top/bottom)
      default_order="date desc">

    <!-- Decoration (row styling) -->
    <field name="state"
           decoration-success="state == 'done'"
           decoration-danger="state == 'cancel'"
           decoration-muted="not active"/>

    <!-- Optional fields (Introduced in Odoo 13) -->
    <field name="phone" optional="show"/>    <!-- shown by default -->
    <field name="mobile" optional="hide"/>    <!-- hidden by default -->

    <!-- Special widgets -->
    <field name="image" widget="image"/>
    <field name="category_id" widget="many2many_tags" options="{'color_field': 'color'}"/>
    <field name="sequence" widget="handle"/>   <!-- drag to reorder -->
</tree>
```

---

## Form View

### Basic Form View

```xml
<record id="view_my_model_form" model="ir.ui.view">
    <field name="name">my.model.form</field>
    <field name="model">my.model</field>
    <field name="arch" type="xml">
        <form string="My Record">
            <header>
                <button name="action_confirm" string="Confirm" type="object" class="oe_highlight"/>
                <field name="state" widget="statusbar"/>
            </header>
            <sheet>
                <group>
                    <group>
                        <field name="name"/>
                        <field name="date"/>
                    </group>
                    <group>
                        <field name="user_id"/>
                    </group>
                </group>
            </sheet>
            <div class="oe_chatter">
                <field name="message_follower_ids" widget="mail_followers"/>
                <field name="activity_ids" widget="mail_activity"/>
                <field name="message_ids" widget="mail_thread"/>
            </div>
        </form>
    </field>
</record>
```

### Dynamic Attributes (attrs)

In Odoo 13, dynamic properties **MUST** use the `attrs` attribute.

```xml
<field name="is_company"/>
<field name="email" attrs="{'required': [('is_company', '=', True)], 'invisible': [('is_company', '=', False)]}"/>
<field name="code" attrs="{'readonly': [('state', '!=', 'draft')]}"/>
<button name="action" string="Action" type="object" attrs="{'invisible': [('state', '=', 'done')]}"/>
```

---

## Search View

### Search View Features

```xml
<search string="Search My Model">
    <field name="name" filter_domain="['|', ('name', 'ilike', self), ('code', 'ilike', self)]"/>
    <field name="partner_id"/>
    
    <filter string="My Records" name="my_records" domain="[('user_id', '=', uid)]"/>
    <separator/>
    <filter string="Draft" name="draft" domain="[('state', '=', 'draft')]"/>

    <group expand="0" string="Group By">
        <filter string="State" name="state" context="{'group_by': 'state'}"/>
    </group>

    <!-- Search Panel (Introduced in Odoo 13) -->
    <searchpanel>
        <field name="category_id" icon="fa-filter"/>
        <field name="state" select="multi" icon="fa-list"/>
    </searchpanel>
</search>
```

---

## View Inheritance

In Odoo 13, you must use the full `xpath` syntax to target elements.

```xml
<record id="view_res_partner_form_inherit" model="ir.ui.view">
    <field name="name">res.partner.form.inherit</field>
    <field name="model">res.partner</field>
    <field name="inherit_id" ref="base.view_partner_form"/>
    <field name="arch" type="xml">
        <!-- Insert after existing field -->
        <xpath expr="//field[@name='email']" position="after">
            <field name="my_field"/>
        </xpath>

        <!-- Modify attributes -->
        <xpath expr="//field[@name='phone']" position="attributes">
            <attribute name="required">1</attribute>
            <attribute name="attrs">{'readonly': [('state', '=', 'done')]}</attribute>
        </xpath>
    </field>
</record>
```

---

## Actions

### Window Action (ir.actions.act_window)

```xml
<record id="action_my_model" model="ir.actions.act_window">
    <field name="name">My Model</field>
    <field name="res_model">my.model</field>
    <field name="view_mode">tree,form</field>
    <field name="help" type="html">
        <p class="o_view_nocontent_smiling_face">
            Create your first record!
        </p>
    </field>
</record>
```

---

**For more Odoo 13 guides, see [SKILL.md](../SKILL.md)**
