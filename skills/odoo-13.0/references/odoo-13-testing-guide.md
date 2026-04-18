---
name: odoo-13-testing
description: Comprehensive guide for testing Odoo 13 modules, including
  TransactionCase, SavepointCase, HttpCase, browser testing, and best practices.
globs: "**/tests/**/*.py"
topics:
  - Test case types (TransactionCase, SavepointCase, HttpCase)
  - Test decorators (tagged, users)
  - Form testing and fixtures
  - Browser testing (browser_js)
  - Mocking and patching
when_to_use:
  - Writing module tests
  - Adding regression coverage
  - Testing UI flows or JS
  - Mocking external services
---

# Odoo 13 Testing Guide

Comprehensive guide for testing Odoo 13 modules, covering test classes, decorators, mocking, form testing, browser testing, and best practices.

## Table of Contents

1. [Base Test Classes](#base-test-classes)
2. [Test Decorators](#test-decorators)
3. [Mocking and Patching](#mocking-and-patching)
4. [Form Testing](#form-testing)
5. [Browser Testing](#browser-testing)
6. [Setup and Teardown](#setup-and-teardown)
7. [Assert Methods](#assert-methods)
8. [Test Data Helpers](#test-data-helpers)
9. [Running Tests](#running-tests)
10. [Best Practices](#best-practices)

---

## Base Test Classes

### Location

Test infrastructure is located in `/odoo/tests/`:

- `common.py` - Base test classes and utilities

### Class Hierarchy

```
BaseCase (abstract)
├── TransactionCase
├── SavepointCase
└── HttpCase (extends TransactionCase)
```

### TransactionCase

**Purpose**: The most common test class. Each test method runs in its own transaction and is rolled back at the end.

**Key Features**:
- Each test method starts with a fresh transaction.
- `setUp()` runs before each test method.
- Use when tests need full isolation and don't share a large amount of setup data.

```python
from odoo.tests import TransactionCase

class TestMyModel(TransactionCase):
    def setUp(self):
        super(TestMyModel, self).setUp()
        self.product = self.env['product.product'].create({
            'name': 'Test Product',
        })

    def test_01_product_exists(self):
        self.assertTrue(self.product)
        self.assertEqual(self.product.name, 'Test Product')
```

### SavepointCase

**Purpose**: For tests that share a large amount of setup data. It uses a single transaction for the whole class and savepoints for each test method.

**Key Features**:
- `setUpClass()` runs once per class.
- Each test method runs in a savepoint and is rolled back to that savepoint.
- Significantly faster than `TransactionCase` when `setUpClass` is expensive.

```python
from odoo.tests import SavepointCase

class TestMyModel(SavepointCase):
    @classmethod
    def setUpClass(cls):
        super(TestMyModel, cls).setUpClass()
        # Expensive setup done once for all tests
        cls.product = cls.env['product.product'].create({
            'name': 'Test Product',
        })

    def test_01_product_exists(self):
        self.assertTrue(self.product)
```

### HttpCase

**Purpose**: For HTTP/browser-based testing with headless Chrome support.

**Key Features**:
- Extends `TransactionCase`.
- Provides `url_open()` for HTTP requests.
- Provides `browser_js()` for JavaScript testing.
- Provides `start_tour()` for tour testing.

```python
from odoo.tests import HttpCase

class TestMyUI(HttpCase):
    def test_tour(self):
        """Run Odoo tour."""
        self.start_tour('/web', 'my_tour_name', login="admin")
```

---

## Test Decorators

### @tagged

Tag test classes or methods for selective execution.

```python
from odoo.tests import tagged

@tagged('-at_install', 'post_install')
class TestMyFeature(TransactionCase):
    pass
```

**Built-in Tags**:
- `standard`: Default tag for regular tests.
- `at_install`: Run during module installation (default).
- `post_install`: Run after installation.

---

## Mocking and Patching

### unittest.mock

Odoo 13 uses standard `unittest.mock` for patching.

```python
from unittest.mock import patch

def test_external_api(self):
    with patch('requests.post') as mock_post:
        mock_post.return_value.status_code = 200
        # ...
```

---

## Form Testing

The `Form` class simulates form view behavior, including onchanges.

```python
from odoo.tests import Form

def test_sale_order_form(self):
    with Form(self.env['sale.order']) as f:
        f.partner_id = self.partner
        with f.order_line.new() as line:
            line.product_id = self.product
    order = f.save()
```

---

## Browser Testing

### browser_js()

Execute JavaScript in headless Chrome.

```python
def test_js_code(self):
    self.browser_js("/web", "console.log('test')", login="admin")
```

### start_tour()

Run Odoo JavaScript tours.

```python
def test_tour(self):
    self.start_tour("/web", 'tour_name', login="admin")
```

---

## Assert Methods

### assertRecordValues()

Compare a recordset with a list of expected values.

```python
self.assertRecordValues(records, [
    {'name': 'Record 1', 'value': 10},
    {'name': 'Record 2', 'value': 20},
])
```

### assertQueryCount()

Verify the number of SQL queries executed.

```python
with self.assertQueryCount(5):
    self.env['res.partner'].search([])
```

---

## Running Tests

```bash
# Run tests for a module
./odoo-bin -c odoo.conf -d my_db -i my_module --test-enable

# Run tests with tags
./odoo-bin -c odoo.conf -d my_db --test-enable --test-tags=post_install
```

---

## Best Practices

1. **Prefer `SavepointCase`** for complex tests to improve performance.
2. **Use `Form`** to test complex onchange logic.
3. **Always use `@tagged`** to categorize tests (e.g., `slow`, `post_install`).
4. **Mock external services** to avoid flaky tests and dependencies.
5. **Use `assertRecordValues`** for clean and readable record state verification.
