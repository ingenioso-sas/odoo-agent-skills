# Odoo 13 Documentation - AI Agents Setup

Setup guide for using Odoo 13 documentation with AI coding assistants (Cursor, Claude Code, Windsurf, Aider, etc.).

## Quick Start

### Install via skills.sh (Recommended)

```bash
# Add Odoo 13 skill to your project
npx skills add unclecatvn/agent-skills
```

Visit [https://skills.sh/](https://skills.sh/) for more installation options.

### Cursor IDE - Remote Rule

Configure once in Cursor settings:
- `Settings` → `Rules` → `Add Remote Rule`
- Source: `Git Repository`
- URL: `git@github.com:unclecatvn/agent-skills.git`
- Branch: `13.0`
- Subfolder: `skills/odoo-13.0/`

---

## Documentation Structure

```
skills/odoo-13.0/
├── SKILL.md                       # Master index (all agents)
├── references/                    # Development guides (18 files)
│   ├── odoo-13-actions-guide.md     # ir.actions.*, cron, bindings
│   ├── odoo-13-controller-guide.md  # HTTP, routing, controllers
│   ├── odoo-13-data-guide.md        # XML/CSV data files, records
│   ├── odoo-13-decorator-guide.md   # @api decorators
│   ├── odoo-13-development-guide.md # Manifest, wizards (overview)
│   ├── odoo-13-field-guide.md       # Field types, parameters
│   ├── odoo-13-manifest-guide.md    # __manifest__.py reference
│   ├── odoo-13-mixins-guide.md      # mail.thread, activities, etc.
│   ├── odoo-13-model-guide.md       # ORM, CRUD, search, domain
│   ├── odoo-13-migration-guide.md   # Migration scripts, hooks
│   ├── odoo-13-javascript-guide.md  # Classic JS Widgets, RPC
│   ├── odoo-13-performance-guide.md # N+1 prevention, optimization
│   ├── odoo-13-reports-guide.md     # QWeb reports, PDF/HTML
│   ├── odoo-13-security-guide.md    # ACL, record rules, security
│   ├── odoo-13-testing-guide.md     # Test classes, decorators, mocking
│   ├── odoo-13-transaction-guide.md # Savepoints, errors
│   ├── odoo-13-translation-guide.md # Translations, i18n
│   └── odoo-13-view-guide.md        # XML views, QWeb
├── CLAUDE.md                      # Claude Code specific
└── AGENTS.md                      # THIS FILE - setup guide
```

---

## Guide Reference

| File | Purpose | When to Use |
|------|---------|-------------|
| `SKILL.md` | Master index for all guides | Find the right guide for your task |
| `references/odoo-13-actions-guide.md` | Actions (window, URL, server, cron) | Creating actions, menus, scheduled jobs |
| `references/odoo-13-controller-guide.md` | HTTP controllers, routing | Writing endpoints |
| `references/odoo-13-data-guide.md` | XML/CSV data files, records | Creating data files |
| `references/odoo-13-decorator-guide.md` | @api decorators usage | Using @api decorators |
| `references/odoo-13-development-guide.md` | Module structure, wizards | Creating new modules |
| `references/odoo-13-field-guide.md` | Field types, parameters | Defining model fields |
| `references/odoo-13-manifest-guide.md` | __manifest__.py reference | Configuring module manifest |
| `references/odoo-13-mixins-guide.md` | mail.thread, activities, mixins | Adding messaging, activities |
| `references/odoo-13-model-guide.md` | ORM methods, CRUD, domains | Writing model methods |
| `references/odoo-13-migration-guide.md` | Migration scripts, hooks | Upgrading modules |
| `references/odoo-13-javascript-guide.md` | Classic JS Widgets, RPC | Building JS UI |
| `references/odoo-13-performance-guide.md` | Performance optimization | Fixing slow code |
| `references/odoo-13-reports-guide.md` | QWeb reports, templates | Creating reports |
| `references/odoo-13-security-guide.md` | ACL, record rules, security | Configuring security |
| `references/odoo-13-testing-guide.md` | Test classes, decorators, mocking | Writing tests |
| `references/odoo-13-transaction-guide.md` | Database transactions, error handling | Savepoints, UniqueViolation |
| `references/odoo-13-translation-guide.md` | Translations, localization, i18n | Adding translations |
| `references/odoo-13-view-guide.md` | XML views, actions, menus | Writing view XML |

---

## AI Agent Configuration

### Cursor IDE

| Setting | Value |
|---------|-------|
| Source | Git Repository |
| URL | `git@github.com:unclecatvn/agent-skills.git` |
| Branch | `13.0` |
| Subfolder | `skills/odoo-13.0/` |

**Globs patterns used by Cursor:**

| File | globs Pattern |
|------|---------------|
| `SKILL.md` | `**/*.{py,xml}` |
| `references/odoo-13-actions-guide.md` | `**/*.{py,xml}` |
| `references/odoo-13-controller-guide.md` | `**/controllers/**/*.py` |
| `references/odoo-13-data-guide.md` | `**/*.{xml,csv}` |
| `references/odoo-13-decorator-guide.md` | `**/models/**/*.py` |
| `references/odoo-13-development-guide.md` | `**/*.{py,xml,csv}` |
| `references/odoo-13-field-guide.md` | `**/models/**/*.py` |
| `references/odoo-13-manifest-guide.md` | `**/__manifest__.py` |
| `references/odoo-13-mixins-guide.md` | `**/models/**/*.py` |
| `references/odoo-13-model-guide.md` | `**/models/**/*.py` |
| `references/odoo-13-migration-guide.md` | `**/migrations/**/*.py` |
| `references/odoo-13-javascript-guide.md` | `static/src/**/*.js` |
| `references/odoo-13-performance-guide.md` | `**/*.{py,xml}` |
| `references/odoo-13-reports-guide.md` | `**/report/**/*.xml` |
| `references/odoo-13-security-guide.md` | `**/security/**/*.{csv,xml}` |
| `references/odoo-13-testing-guide.md` | `**/tests/**/*.py` |
| `references/odoo-13-transaction-guide.md` | `**/models/**/*.py` |
| `references/odoo-13-translation-guide.md` | `**/*.{py,js,xml}` |
| `references/odoo-13-view-guide.md` | `**/views/**/*.xml` |

### Claude Code

```bash
# Install via skills.sh
npx skills add unclecatvn/agent-skills
```

Claude Code reads:
- `CLAUDE.md` - Project overview and quick reference
- `SKILL.md` - Master index for all guides
- Individual guides in `references/` - Detailed information

### Other Agents

| Agent | Setup |
|-------|-------|
| Windsurf | Same as Cursor (uses `.mdc` files) |
| Continue | Place `CLAUDE.md` or `SKILL.md` in root |
| Aider | Place `CLAUDE.md` or add to prompt |
| OpenCode | Copy skill folder to project - no additional config needed |

---

## Cursor / Claude Skills Folder

After installing via `npx skills add unclecatvn/agent-skills`, the skill is placed at:

```
.cursor/skills/
└── odoo-13/
    └── SKILL.md

.claude/skills/
└── odoo-13/
    └── SKILL.md
```

---

## Key Odoo 13 Conventions

| Change | Pattern (Odoo 13) |
|--------|-------------------|
| List view tag | `<tree>` |
| Dynamic attributes | `attrs="{'invisible': [...]}"` |
| Delete validation | Override `unlink()` |
| Field aggregation | `group_operator=` |
| SQL queries | `cr.execute(query, params)` |

---

## Repository

**URL**: `git@github.com:unclecatvn/agent-skills.git`
**Branch**: `13.0`
**License**: MIT
