---
name: odoo-13-javascript
description: Complete reference for Odoo 13 JavaScript framework. Covers odoo.define, Widget class, AbstractAction, web.rpc, web.core, QWeb templates, and classic inheritance patterns.
globs: "**/static/src/**/*.js"
topics:
  - Module definition (odoo.define)
  - Widget class and lifecycle (init, willStart, start, destroy)
  - AbstractAction for client actions
  - Calling Python (web.rpc, _rpc)
  - QWeb templates and XML declaration
  - Event handling (events object)
  - Core services (web.core, _t, qweb)
  - Registries (field_registry, action_registry)
when_to_use:
  - Creating custom widgets for form views
  - Implementing client actions
  - Building interactive UI components in Odoo 13
  - Extending existing JavaScript functionality
---

# Odoo 13 JavaScript Guide

Complete reference for the Odoo 13 classic JavaScript framework: Widgets, RPC, and QWeb.

## Table of Contents

1. [Module Definition](#module-definition)
2. [Widget Class](#widget-class)
3. [Widget Lifecycle](#widget-lifecycle)
4. [QWeb Templates](#qweb-templates)
5. [RPC and Data](#rpc-and-data)
6. [Client Actions](#client-actions)
7. [Core Services](#core-services)
8. [Registries](#registries)
9. [Event Handling](#event-handling)
10. [Common Patterns](#common-patterns)

---

## Module Definition

In Odoo 13, all JavaScript files must be wrapped in `odoo.define`.

```javascript
odoo.define('my_module.MyWidget', function (require) {
"use strict";

var core = require('web.core');
var Widget = require('web.Widget');

var MyWidget = Widget.extend({
    // Implementation
});

return MyWidget;
});
```

### Module Naming Convention
- `addon_name.FileName` or `addon_name.ComponentName`

---

## Widget Class

`web.Widget` is the base class for all UI components.

### Basic Structure

```javascript
var MyWidget = Widget.extend({
    template: 'MyTemplate', // Name of the QWeb template
    events: {
        'click .my_button': '_onButtonClick',
    },

    init: function (parent, options) {
        this._super.apply(this, arguments);
        this.options = options || {};
    },

    willStart: function () {
        var self = this;
        return this._super.apply(this, arguments).then(function() {
            // Async setup: return a promise
            return self._loadData();
        });
    },

    start: function () {
        // DOM is now rendered and available as this.$el
        this.$('.my_title').text('Hello Odoo 13');
        return this._super.apply(this, arguments);
    },

    _onButtonClick: function (ev) {
        ev.preventDefault();
        console.log("Button clicked!");
    },
});
```

---

## Widget Lifecycle

### Lifecycle Order

1. `init(parent, ...)`
   - Synchronous constructor. Sets up initial state.
2. `willStart()`
   - Asynchronous. Should return a promise. Used for loading data before rendering.
3. `renderElement()` (Internal)
   - Renders the `template` and sets `this.$el`.
4. `start()`
   - Asynchronous. Called after rendering. Ideal for DOM manipulation and third-party library initialization.
5. `destroy()`
   - Cleanup. Removes DOM elements and unbinds events.

---

## QWeb Templates

In Odoo 13, templates are defined in XML files in `static/src/xml/`.

**XML file (`static/src/xml/my_templates.xml`)**:
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates>
    <t t-name="MyTemplate">
        <div class="my_widget">
            <h3 class="my_title">Loading...</h3>
            <button class="my_button btn btn-primary">Click Me</button>
            <ul>
                <t t-foreach="items" t-as="item">
                    <li><t t-esc="item.name"/></li>
                </t>
            </ul>
        </div>
    </t>
</templates>
```

**Note**: In Odoo 13, templates must be registered in the manifest's `qweb` key:
```python
'qweb': ['static/src/xml/my_templates.xml'],
```

---

## RPC and Data

### Calling Python Methods (RPC)

Widgets have a helper `_rpc` method (via `web.mixins.common`).

```javascript
// Calling a model method
this._rpc({
    model: 'res.partner',
    method: 'search_read',
    domain: [['is_company', '=', true]],
    fields: ['name', 'email'],
}).then(function (result) {
    console.log(result);
});

// Calling a controller route
this._rpc({
    route: '/my/custom/route',
    params: {
        id: 123,
    },
}).then(function (data) {
    // ...
});
```

---

## Client Actions

Client actions are widgets that take over the main content area.

```javascript
var AbstractAction = require('web.AbstractAction');
var core = require('web.core');

var MyClientAction = AbstractAction.extend({
    template: 'MyClientActionTemplate',

    start: function () {
        this.$el.append($('<div>').text('Welcome to the Client Action'));
        return this._super.apply(this, arguments);
    },
});

core.action_registry.add('my_client_action', MyClientAction);

return MyClientAction;
```

---

## Core Services

### web.core

Access registries, translations, and the global event bus.

```javascript
var core = require('web.core');
var _t = core._t; // Translation function
var qweb = core.qweb; // QWeb engine

// Translation example
var message = _t("Are you sure?");

// Manual template rendering
var html = qweb.render('MyTemplate', {items: []});
```

---

## Registries

Used to register views, widgets, and actions.

```javascript
var field_registry = require('web.field_registry');
var action_registry = require('web.action_registry');

// Register a field widget
field_registry.add('my_custom_field', MyFieldWidget);

// Register a client action
action_registry.add('my_action_tag', MyActionWidget);
```

---

## Event Handling

### events Object
Uses jQuery event delegation. The key is `'event selector'`, and the value is the method name.

```javascript
events: {
    'click .btn_save': '_onSave',
    'change input[name="name"]': '_onNameChange',
},
```

### Manual Binding
Sometimes you need to bind events to the window or elements outside the widget.

```javascript
start: function() {
    $(window).on('resize.my_widget', this._onResize.bind(this));
    return this._super.apply(this, arguments);
},

destroy: function() {
    $(window).off('resize.my_widget');
    this._super.apply(this, arguments);
},
```

---

## Common Patterns

### Asynchronous Data Loading in willStart

```javascript
willStart: function () {
    var self = this;
    var def = this._rpc({
        model: 'res.partner',
        method: 'read',
        args: [[this.partnerId], ['name']],
    }).then(function (data) {
        self.partnerData = data[0];
    });

    return Promise.all([this._super.apply(this, arguments), def]);
},
```

### Passing Data to Templates

When `template` is defined, the widget automatically renders it during `renderElement`. To pass data, you must override `renderElement` or use `start`. However, the standard way to provide data to the initial render is by defining a `renderElement` or using a widget that supports a `state` object (less common in base Widget).

Standard pattern:
```javascript
var MyWidget = Widget.extend({
    template: 'MyTemplate',
    
    willStart: function() {
        var self = this;
        return this._rpc(...).then(function(data) {
            self.data = data;
        });
    },

    renderElement: function() {
        // Provide 'widget' to the template context
        this._super.apply(this, arguments);
        // OR manually render:
        // this.replaceElement(core.qweb.render(this.template, {widget: this}));
    },
});
```

In the template:
```xml
<t t-name="MyTemplate">
    <div>
        <t t-esc="widget.data.name"/>
    </div>
</t>
```

---

**For more Odoo 13 guides, see [SKILL.md](../SKILL.md)**
