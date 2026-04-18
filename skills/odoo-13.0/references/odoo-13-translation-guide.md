---
name: odoo-13-translation
description: Complete guide for Odoo 13 translations and localization. Covers Python translations with _() and _lt(), JavaScript translations with _t(), QWeb template translations, field translations with translate=True, and PO file structure.
globs: "**/*.{py,js,xml}"
topics:
  - Python translations (_ and _lt)
  - JavaScript translations (_t)
  - QWeb template translations
  - Field translations (translate=True)
  - PO file structure
  - Translation export/import
when_to_use:
  - Adding translatable strings to Python code
  - Adding translations to JavaScript widgets
  - Creating translatable QWeb templates
  - Setting up translated fields
---

# Odoo 13 Translation & Localization Guide

Complete guide for translating and localizing Odoo 13 modules.

## Quick Reference

| Context | Function | Example |
|---------|----------|---------|
| Python code | `_()` | `_("Hello World")` |
| Python constants | `_lt()` | `TITLE = _lt("Module Title")` |
| JavaScript (Classic) | `_t()` | `_t("Hello World")` |
| Field definition | `translate=True` | `name = fields.Char(translate=True)` |

---

## Table of Contents

1. [Python Translations](#python-translations)
2. [Field Translations](#field-translations)
3. [QWeb Template Translations](#qweb-template-translations)
4. [JavaScript Translations](#javascript-translations)
5. [Module Translation Structure](#module-translation-structure)
6. [Translation Export/Import](#translation-exportimport)
7. [Best Practices](#best-practices)

---

## Python Translations

### Standard Translation Function

```python
from odoo import _

# Simple translation
message = _("Hello World")

# With formatting
message = _("Hello %s") % user.name

# With named arguments
message = _("Hello %(name)s") % {'name': user.name}
```

### Lazy Translation

For constants defined at the module level:

```python
from odoo import _, _lt

# Translated when used, not when defined
DESCRIPTION = _lt("This is a lazy translation")
```

---

## Field Translations

### Simple Field Translation

```python
class MyModel(models.Model):
    _name = 'my.model'

    name = fields.Char(string='Name', translate=True)
```

**Note**: In Odoo 13, translations for fields are stored in the `ir_translation` table, not as JSONB in the model's table.

### Accessing Translations in Code

To get a field's value in a specific language:

```python
# Use with_context to change the language
record_fr = self.with_context(lang='fr_FR').browse(record_id)
print(record_fr.name)  # Output in French
```

---

## QWeb Template Translations

### Basic QWeb Translation

Text inside QWeb tags is automatically extracted for translation.

```xml
<template id="my_template">
    <div>
        <p>This text is translatable</p>
        <span t-esc="_t('This is also translatable')"/>
    </div>
</template>
```

### Translatable Attributes

Common attributes like `title`, `alt`, `placeholder`, and `string` are automatically translatable.

```xml
<input placeholder="Search..."/>
<img src="..." alt="Company Logo"/>
```

---

## JavaScript Translations

In Odoo 13 (Classic JS), use the `web.core` module to access `_t`.

```javascript
odoo.define('my_module.my_script', function (require) {
    "use strict";

    var core = require('web.core');
    var _t = core._t;

    // Usage
    var message = _t("Operation successful");
});
```

---

## Module Translation Structure

### Directory Structure

```
my_module/
├── i18n/
│   ├── my_module.pot    # Translation template (source strings)
│   ├── es.po           # Spanish translations
│   └── fr.po           # French translations
```

### Generating the POT File

You can generate the template file using the Odoo CLI:

```bash
./odoo-bin -c odoo.conf -d my_db --i18n-export=my_module/i18n/my_module.pot --modules=my_module
```

---

## Translation Export/Import

### Via User Interface

1. Activate **Developer Mode**.
2. Go to **Settings > Translations > Export Translation**.
3. Select the language and the module.
4. Download the `.po` file, translate it, and place it in the `i18n/` folder.
5. To load new translations, use **Import Translation** or update the module.

---

## Best Practices

1. **Don't concatenate strings**: `_("Hello ") + name` is bad. Use `_("Hello %s") % name`.
2. **Use complete sentences**: Avoid splitting phrases into multiple `_()` calls.
3. **Lazy translations for constants**: Use `_lt()` for global variables or field defaults.
4. **Translate attributes**: Ensure `placeholder`, `title`, and `string` are used for user-facing text.
5. **Keep PO files updated**: Always export a new POT file when adding new translatable strings.
