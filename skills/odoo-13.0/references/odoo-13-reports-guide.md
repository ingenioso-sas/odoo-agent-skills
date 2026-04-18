---
name: odoo-13-reports
description: Complete reference for Odoo 13 QWeb reports covering PDF/HTML reports, report templates, paper formats, custom reports, translatable templates, and report actions.
globs: "**/*.{py,xml}"
topics:
  - QWeb reports (qweb-pdf, qweb-html)
  - Report templates and layouts
  - Paper formats (report.paperformat)
  - Custom reports with _get_report_values
  - Translatable reports (t-lang)
  - Barcodes in reports
  - Report actions and bindings
when_to_use:
  - Creating PDF reports for models
  - Designing report templates
  - Creating translatable reports
  - Implementing barcode support
  - Customizing report rendering
---

# Odoo 13 Reports Guide

Complete reference for Odoo 13 QWeb reports: PDF/HTML reports, templates, paper formats, and custom reports.

## Table of Contents

1. [Report Basics](#report-basics)
2. [Report Templates](#report-templates)
3. [Report Actions](#report-actions)
4. [Paper Formats](#paper-formats)
5. [Custom Reports](#custom-reports)
6. [Translatable Reports](#translatable-reports)
7. [Barcodes](#barcodes)

---

## Report Basics

### QWeb Reports

Reports in Odoo are written in HTML/QWeb and rendered to PDF using `wkhtmltopdf`.

#### Report Types

| Type | Description |
|------|-------------|
| `qweb-pdf` | PDF report (most common) |
| `qweb-html` | HTML report (for web viewing) |

### Report Declaration (Shortcut)

```xml
<report
    id="action_report_invoice"
    model="account.move"
    string="Invoices"
    report_type="qweb-pdf"
    name="account.report_invoice"
    file="account.report_invoice"
    print_report_name="'Invoice_%s' % (object.name)"
/>
```

---

## Report Templates

### Minimal Template

```xml
<template id="report_invoice">
    <t t-call="web.html_container">
        <t t-foreach="docs" t-as="o">
            <t t-call="web.external_layout">
                <div class="page">
                    <h2>Invoice</h2>
                    <p>Invoice Number: <span t-field="o.name"/></p>
                    <p>Amount: <span t-field="o.amount_total"/></p>
                </div>
            </t>
        </t>
    </t>
</template>
```

### Template Structure

```
web.html_container
    └── web.external_layout (header + footer)
            └── div.page (your content)
```

### Common QWeb Directives

```xml
<t t-foreach="docs" t-as="doc">
    <p t-field="doc.name"/>
</t>

<p t-if="doc.state == 'draft'">Draft Document</p>

<span t-field="o.date_order"/>  <!-- Formatted date -->
<span t-field="o.amount_total"/>  <!-- Formatted currency -->
```

---

## Report Actions (XML Record)

```xml
<record id="action_report_my_model" model="ir.actions.report">
    <field name="name">My Report</field>
    <field name="model">my.model</field>
    <field name="report_type">qweb-pdf</field>
    <field name="report_name">my_module.my_report_template</field>
    <field name="binding_model_id" ref="model_my_model"/>
    <field name="binding_type">report</field>
</record>
```

---

## Paper Formats

Define custom paper sizes and margins.

```xml
<record id="paperformat_euro" model="report.paperformat">
    <field name="name">European A4</field>
    <field name="default" eval="True"/>
    <field name="format">A4</field>
    <field name="orientation">Portrait</field>
    <field name="margin_top">40</field>
    <field name="margin_bottom">20</field>
    <field name="margin_left">7</field>
    <field name="margin_right">7</field>
    <field name="header_spacing">35</field>
    <field name="dpi">90</field>
</record>
```

---

## Custom Reports

For additional data in reports, create a custom report model:

```python
from odoo import api, models

class ReportMyModel(models.AbstractModel):
    _name = 'report.my_module.my_report_template'

    @api.model
    def _get_report_values(self, docids, data=None):
        docs = self.env['my.model'].browse(docids)
        return {
            'doc_ids': docids,
            'doc_model': 'my.model',
            'docs': docs,
            'custom_data': 'Some extra value',
        }
```

---

## Translatable Reports

Use `t-lang` to set the language of the report based on a record field (e.g., partner language).

```xml
<template id="report_saleorder">
    <t t-call="web.html_container">
        <t t-foreach="docs" t-as="doc">
            <t t-call="sale.report_saleorder_document" t-lang="doc.partner_id.lang"/>
        </t>
    </t>
</template>
```

---

## Barcodes

Barcodes can be embedded in reports using the report barcode controller.

```xml
<!-- QR code -->
<img t-att-src="'/report/barcode/QR/%s' % 'My text'"/>

<!-- EAN-13 barcode -->
<img t-att-src="'/report/barcode/?type=%s&amp;value=%s&amp;width=%s&amp;height=%s' % ('EAN13', o.barcode, 600, 100)"/>
```

---

**For more Odoo 13 guides, see [SKILL.md](../SKILL.md)**
