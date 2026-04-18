---
name: odoo-13-mixins
description: Complete reference for Odoo 13 mixins and useful classes. Covers mail.thread (messaging, chatter, field tracking), mail.alias.mixin, mail.activity.mixin, utm.mixin, website.published.mixin, website.seo.metadata, and rating.mixin.
globs: "**/models/**/*.py"
topics:
  - mail.thread (messaging, chatter, followers)
  - mail.alias.mixin (email aliases)
  - mail.activity.mixin (activities)
  - utm.mixin (campaign tracking)
  - website.published.mixin (website visibility)
  - website.seo.metadata (SEO)
  - rating.mixin (customer ratings)
when_to_use:
  - Adding messaging/chatter to models
  - Setting up email aliases
  - Adding activities to models
  - Tracking marketing campaigns
  - Creating website-publishable content
  - Implementing customer ratings
---

# Odoo 13 Mixins Guide

Complete reference for Odoo 13 mixins: messaging, email, activities, tracking, website features, and ratings.

## Table of Contents

1. [mail.thread - Messaging](#mailthread---messaging)
2. [mail.alias.mixin - Email Aliases](#mailaliasmixin---email-aliases)
3. [mail.activity.mixin - Activities](#mailactivitymixin---activities)
4. [utm.mixin - Campaign Tracking](#utmmixin---campaign-tracking)
5. [website.published.mixin - Website Visibility](#websitepublishedmixin---website-visibility)
6. [website.seo.metadata - SEO](#websiteseometadata---seo)
7. [rating.mixin - Customer Ratings](#ratingmixin---customer-ratings)

---

## mail.thread - Messaging

### Basic Messaging Integration

The `mail.thread` mixin provides full messaging capabilities: chatter, followers, messages, and field tracking.

#### Minimal Setup

```python
from odoo import models, fields

class BusinessTrip(models.Model):
    _name = 'business.trip'
    _inherit = ['mail.thread']
    _description = 'Business Trip'

    name = fields.Char()
    partner_id = fields.Many2one('res.partner', 'Responsible')
```

#### Form View Integration (Odoo 13)

In Odoo 13, you must manually add the chatter fields at the end of the form.

```xml
<record id="business_trip_form" model="ir.ui.view">
    <field name="name">business.trip.form</field>
    <field name="model">business.trip</field>
    <field name="arch" type="xml">
        <form string="Business Trip">
            <sheet>
                <group>
                    <field name="name"/>
                    <field name="partner_id"/>
                </group>
            </sheet>
            <!-- Chatter integration -->
            <div class="oe_chatter">
                <field name="message_follower_ids" widget="mail_followers"/>
                <field name="activity_ids" widget="mail_activity"/>
                <field name="message_ids" widget="mail_thread"/>
            </div>
        </form>
    </field>
</record>
```

### Field Tracking

Automatically log field changes in the chatter. In Odoo 13, you use `tracking=True`.

```python
class BusinessTrip(models.Model):
    _name = 'business.trip'
    _inherit = ['mail.thread']

    name = fields.Char(tracking=True)  # Track changes
    state = fields.Selection([
        ('draft', 'New'),
        ('confirmed', 'Confirmed'),
    ], tracking=True)
```

### Posting Messages

#### message_post() - Post a Message

```python
def send_notification(self):
    self.message_post(
        body='Trip has been confirmed!',
        subject='Trip Confirmation',
        subtype='mail.mt_comment',
    )
```

#### message_post() with Attachments

```python
def send_with_attachment(self):
    self.message_post(
        body='Please review attached document',
        attachments=[
            ('document.pdf', pdf_content),
        ]
    )
```

---

## mail.alias.mixin - Email Aliases

### Alias Basics

Aliases allow creating records via email.

#### Required Overrides

```python
from odoo import models, fields

class BusinessTrip(models.Model):
    _name = 'business.trip'
    _inherit = ['mail.thread', 'mail.alias.mixin']

    name = fields.Char()
    alias_id = fields.Many2one('mail.alias', required=True, ondelete="restrict")

    def _get_alias_model_name(self, vals):
        return 'business.expense'

    def _get_alias_values(self):
        values = super(BusinessTrip, self)._get_alias_values()
        values['alias_defaults'] = {'trip_id': self.id}
        return values
```

---

## mail.activity.mixin - Activities

### Activity Integration

Activities are actions users need to take.

```python
class BusinessTrip(models.Model):
    _name = 'business.trip'
    _inherit = ['mail.thread', 'mail.activity.mixin']
```

#### Kanban View Integration

```xml
<kanban>
    <field name="activity_ids"/>
    <field name="activity_state"/>
    <templates>
        <t t-name="kanban-box">
            <div class="oe_kanban_global_click">
                <field name="name"/>
                <div class="oe_kanban_bottom_right">
                    <field name="activity_ids" widget="kanban_activity"/>
                </div>
            </div>
        </t>
    </templates>
</kanban>
```

---

## utm.mixin - Campaign Tracking

Track marketing campaigns (campaign, source, medium).

```python
class Lead(models.Model):
    _name = 'crm.lead'
    _inherit = ['utm.mixin']
```

| Field | Description |
|-------|-------------|
| `campaign_id` | UTM Campaign |
| `source_id` | UTM Source |
| `medium_id` | UTM Medium |

---

## website.published.mixin - Website Visibility

Control whether records are visible on the website.

```python
class BlogPost(models.Model):
    _name = 'blog.post'
    _inherit = ['website.published.mixin']

    website_url = fields.Char(compute='_compute_website_url')

    def _compute_website_url(self):
        for post in self:
            post.website_url = "/blog/%s" % post.id
```

---

## website.seo.metadata - SEO

Inject metadata into frontend pages.

```python
class BlogPost(models.Model):
    _name = 'blog.post'
    _inherit = ['website.seo.metadata', 'website.published.mixin']
```

---

## rating.mixin - Customer Ratings

Allow sending rating requests.

```python
class ProjectTask(models.Model):
    _name = 'project.task'
    _inherit = ['rating.mixin', 'mail.thread']

    partner_id = fields.Many2one('res.partner', 'Customer')
```

---

## Quick Reference

| Mixin | Purpose |
|-------|---------|
| `mail.thread` | Messaging / Chatter |
| `mail.alias.mixin` | Email Aliases |
| `mail.activity.mixin` | Activities |
| `utm.mixin` | UTM Tracking |
| `website.published.mixin` | Website Publishing |
| `website.seo.metadata` | SEO Metadata |
| `rating.mixin` | Customer Ratings |

---

**For more Odoo 13 guides, see [SKILL.md](../SKILL.md)**
