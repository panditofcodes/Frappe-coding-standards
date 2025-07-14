# ✅ Frappe & ERPNext Customization Coding Standards and Best Practices

> A Complete Developer Handbook with Code Examples, Naming Rules, and Implementation Tips

Whether you're building custom apps on the **Frappe Framework** or tailoring **ERPNext** to meet business needs, following proper coding standards is essential to maintain **clean**, **upgrade-safe**, and **collaborative** code.

---

## 📘 Table of Contents
  * [✅ 1. General Coding Standards](#--1-general-coding-standards)
  * [✅ 2. Frappe App Structure Guidelines](#--2-frappe-app-structure-guidelines)
  * [✅ 3. Doctype Customization](#--3-doctype-customization)
  * [✅ 4. Client-Side Scripting Standards](#--4-client-side-scripting-standards)
  * [✅ 5. Server-Side Scripting Standards](#--5-server-side-scripting-standards)
  * [✅ 6. Hooks & Document Events](#--6-hooks---document-events)
  * [✅ 7. Permission Handling](#--7-permission-handling)
  * [✅ 8. REST API & Whitelisted Functions](#--8-rest-api---whitelisted-functions)
  * [✅ 9. Reports & Dashboards](#--9-reports---dashboards)
  * [✅ 10. Testing & Debugging](#--10-testing---debugging)
  * [✅ 11. Fixtures & Migrations](#--11-fixtures---migrations)
  * [✅ 12. Git & Bench Best Practices](#--12-git---bench-best-practices)
  * [✅ 13. Naming Conventions](#--13-naming-conventions)
  * [✅ 14. UI/UX Best Practices](#--14-ui-ux-best-practices)
  * [✅ 15. Performance Tips](#--15-performance-tips)
  * [✅ 16. Good vs. Bad Naming Examples](#--16-good-vs-bad-naming-examples)

---

## ✅ 1. General Coding Standards

* Use **Python 3** and follow **PEP-8**.
* Use **4-space indentation**, not tabs.
* Use **CamelCase** for class names, **snake\_case** for variables and functions.
* Use **meaningful and descriptive names**.
* Include **docstrings** for all classes and functions.
* Avoid hardcoding values — fetch from `frappe.conf` or constants.

✅ Example:

```python
def calculate_discounted_amount(base_amount, discount_percent):
    discount = base_amount * (discount_percent / 100)
    return base_amount - discount
```

---

## ✅ 2. Frappe App Structure Guidelines

* Keep custom apps **modular and isolated** from ERPNext/Frappe core.
* Organize code into proper folders: `doctype`, `page`, `report`, `dashboard_chart`, etc.
* Avoid modifying core files; use **hooks**, **fixtures**, and **patches** instead.

---

## ✅ 3. Doctype Customization

* Use **Custom Fields** or **Property Setters** when changing standard doctypes.
* Follow consistent naming: `posting_date`, `status`, `company`.
* Choose proper field types: **Link**, **Select**, **Table**, **Currency**, etc.

✅ Field Naming Example:

| Label          | Fieldname       | Fieldtype            |
| -------------- | --------------- | -------------------- |
| Customer Name  | `customer_name` | Data                 |
| Posting Date   | `posting_date`  | Date                 |
| Linked Invoice | `sales_invoice` | Link (Sales Invoice) |

---

## ✅ 4. Client-Side Scripting Standards

* Use `frappe.ui.form.on("Doctype", { ... })` pattern.
* Use `frappe.call()` with error handling.
* Avoid direct DOM access; use `frm` methods.
* Break code into reusable functions.
* Use `async/await` if supported (v14+).

✅ Example:

```javascript
frappe.ui.form.on('Sales Invoice', {
  refresh(frm) {
    if (!frm.doc.__islocal) {
      frm.add_custom_button('Fetch Orders', () => {
        frappe.call({
          method: 'my_app.api.fetch_orders',
          args: { customer: frm.doc.customer },
          callback(r) {
            if (r.message) {
              frappe.msgprint('Orders fetched!');
            }
          }
        });
      });
    }
  }
});
```

---

## ✅ 5. Server-Side Scripting Standards

* Import `frappe` at the top.
* Sanitize inputs and validate all arguments.
* Use ORM: `frappe.get_doc()`, `frappe.db.get_value()`, `frappe.get_all()`.
* Use `frappe.throw()` instead of `raise`.

✅ Example:

```python
@frappe.whitelist()
def get_customer_balance(customer):
    balance = frappe.db.get_value("Customer", customer, "outstanding_amount")
    return balance or 0.0
```

---

## ✅ 6. Hooks & Document Events

✅ Sample `hooks.py`:

```python
doc_events = {
    "Sales Invoice": {
        "on_submit": "my_app.events.sales_invoice.on_submit"
    }
}
fixtures = ["Custom Field", "Property Setter", "Custom Script"]
```

✅ Event Method:

```python
def validate(self):
    if not self.customer:
        frappe.throw("Customer is required.")
```

---

## ✅ 7. Permission Handling

* Prefer **Role-Based Permissions** from UI.
* Use `has_permission` for advanced rules.
* Use **custom roles**, not `System Manager` for everything.

---

## ✅ 8. REST API & Whitelisted Functions

* Decorate API functions with `@frappe.whitelist()`.
* Use `allow_guest=True` only if public access is intended.
* Sanitize all user inputs.

---

## ✅ 9. Reports & Dashboards

* Use **Query Report** for SQL-based reports.
* Use **Script Report** for Python-generated data.
* Avoid `SELECT *`; use specific columns.

✅ Example:

```sql
SELECT
  name,
  customer,
  posting_date,
  grand_total
FROM `tabSales Invoice`
WHERE docstatus = 1
```

---

## ✅ 10. Testing & Debugging

* Use `frappe.logger()` instead of `print()` for logs.
* Use `frappe.db.rollback()` in test cases.
* Use `frappe.tests.utils.FrappeTestCase`.

---

## ✅ 11. Fixtures & Migrations

* Export with:
  `bench export-fixtures`
* Define in `hooks.py`:

```python
fixtures = ["Custom Field", "Property Setter"]
```

✅ Patch Example:

```python
def execute():
    for doc in frappe.get_all("Customer"):
        frappe.db.set_value("Customer", doc.name, "loyalty_points", 0)
```

---

## ✅ 12. Git & Bench Best Practices

* Use Git for version control.
* Use feature branches: `feature/new-module`.
* Run before pushing:

  * `bench update --patch`
  * `bench build`
  * `bench migrate`
* Keep `sites/common_site_config.json` in `.gitignore`.

---

## ✅ 13. Naming Conventions

| Element     | Style                  | Example                          |
| ----------- | ---------------------- | -------------------------------- |
| Variable    | `snake_case`           | `total_amount`, `invoice_date`   |
| Function    | `snake_case`           | `calculate_tax()`, `get_items()` |
| Class Name  | `CamelCase`            | `SalesInvoice`, `PaymentEntry`   |
| Constant    | `UPPER_CASE`           | `MAX_LIMIT`, `DEFAULT_CURRENCY`  |
| JS Variable | `camelCase`            | `invoiceTotal`, `itemList`       |
| JS Function | `camelCase`            | `refreshForm()`, `submitForm()`  |
| App Name    | `lowercase_underscore` | `my_custom_app`                  |
| Git Branch  | `feature/<name>`       | `feature/add-tax-field`          |

---

## ✅ 14. UI/UX Best Practices

* Use `depends_on`, `read_only`, `mandatory` wisely.
* Organize fields into **sections** and **columns**.
* Keep form interfaces minimal and clean.

---

## ✅ 15. Performance Tips

* Avoid DB queries in loops.
* Use Redis cache: `frappe.cache()`.
* Index commonly queried fields.
* Use background jobs (`frappe.enqueue`) for long operations.

---

## ✅ 16. Good vs. Bad Naming Examples

| ❌ Bad Name  | ✅ Good Name           |
| ----------- | --------------------- |
| `custName`  | `customer_name`       |
| `calcTax`   | `calculate_tax()`     |
| `refNo`     | `reference_number`    |
| `getInfo()` | `get_customer_info()` |
| `MyApp`     | `my_app`              |

---
