# Lumber — Customer Success Project

> Source: [Lumber](https://www.notion.so/28c1d464528f801bac72d6c064d30db2)
> Sub-pages:
> - [Workforce Junction](https://www.notion.so/2f71d464528f80428a68dd67e47451a0)
> - [Workforce Junction - Schema for MDV](https://www.notion.so/28e1d464528f80238c9cc5eced5161fc)
> - [Schema of Lumber Postgres](https://www.notion.so/2f71d464528f8092b0dce634ef55bfb7)
> - [Job Costing Report](https://www.notion.so/2fb1d464528f808b987cce2583b7fb8a)
> - [Payroll reporting](https://www.notion.so/3041d464528f80c4bc54c355306a4129)
> - [Payroll tables documentation](https://www.notion.so/3041d464528f8000b005f648dcf2bef6)
> - [Payroll reports](https://www.notion.so/3041d464528f80ae954cc7968a6373fd)

---

## Workspace

- **Workspace ID:** `iaEj4QYboi`
- **Snowflake project:** `https://remix-dev.remixlabs.com/e/edit/snowflake`
- **Lumber reporting:** `https://remix-india.remixlabs.com/e/edit/lumber_reporting`
- **Web catalog:** `remix.app/run?_rmx_url=https://agt.files.remix.app/iaEj4QYboi/_rmx_files/apps/catalog.remix`
- **Table configurator:** `https://agt.files.remix.app/iaEj4QYboi/_rmx_files/apps/component_table.remix`

---

## Two Database Domains

### 1. Job Costing Schema (PostgreSQL — `lfi_` prefix)

6 core tables for construction job costing:

| Table                               | Purpose                                                  |
|-------------------------------------|----------------------------------------------------------|
| `lfi_company`                       | Master company/org data (74 sample rows)                 |
| `lfi_project`                       | Job sites/work assignments per company                   |
| `lfi_cost_code_hierarchical`        | Hierarchical cost codes (parent/child)                   |
| `lfi_project_cost_code_association` | Links projects to cost codes with estimated hours        |
| `lfi_timesheet`                     | Worker clock in/out entries by project + cost code       |
| `lfi_worker_compensation_detail`    | Computed pay from timesheets (REG/OT/DOT rates + travel) |

**Data flow:** company → project → timesheet → worker_compensation_detail, with cost codes categorizing work.

**Key function:** `fnc_cost_code_report_for_all_projects(p_company_id, p_start_date, p_end_date)` — aggregates across all projects, returns one row per cost code with period actuals + all-time totals.

### 2. Workforce Junction Schema (SQL Server — `h_` prefix)

Employee benefits management system for MDV. Core tables:

| Table                                       | Purpose                                          |
|---------------------------------------------|--------------------------------------------------|
| `h_employee_personalInfo`                   | PK: (CompanyId, PersonId). Full employee details |
| `h_dependent_personalInfo`                  | Employee dependents                              |
| `h_employeeInvalidEnrollmentInfo`           | Benefits enrollment records                      |
| `h_dependent_enrollment_info`               | Dependent enrollment                             |
| `h_Documents` / `h_EmployeeEnrollDocuments` | Document storage + enrollment docs               |
| `h_EmployeeEmergencyContacts`               | Emergency contacts                               |
| `H_EmployeeSettings`                        | Notification preferences                         |

> Full DDL: fetch from [Workforce Junction - Schema for MDV](https://www.notion.so/28e1d464528f80238c9cc5eced5161fc)

---

## Payroll Schema (PostgreSQL — `prl_` prefix)

38 tables in three groups:

| Group                  | Tables | Purpose                                                             |
|------------------------|--------|---------------------------------------------------------------------|
| **Payroll Processing** | 16     | Payroll runs, employee items, earnings, benefits, taxes, deductions |
| **Journal Entries**    | 14     | Accounting journals at company + employee level                     |
| **Chart of Accounts**  | 8      | GL accounts, classes, groups, mappings                              |

### Key Tables

- `prl_payroll` — Central payroll run (type: REGULAR/OFF_CYCLE, status: DRAFT/APPROVED/PAID)
- `prl_payroll_employee_item` — One per employee per payroll, links to line items
- `prl_payroll_employee_earning` — Earnings by type (REGULAR/OVERTIME/DOUBLE_OVERTIME) with project + cost code
- `prl_payroll_totals` — Aggregate payroll totals (gross, net, benefits, taxes, cash requirement)
- `prl_journal` — One journal per payroll, with company-level and employee-level breakdowns

### Data Flow

```
lfi_timesheet → prl_payroll_timesheet_map → prl_payroll
  → prl_payroll_employee_item
    → earnings / benefits / taxes / deductions / reimbursements
  → prl_payroll_totals
  → prl_journal → journal line items (company + employee level)
    → prl_chart_of_account_mapping → prl_chart_of_account
```

> Full table docs: fetch from [Payroll tables documentation](https://www.notion.so/3041d464528f8000b005f648dcf2bef6)

---

## Payroll Report Functions (50+)

Organized into groups. All return TABLE types (parameterized views):

| Group              | Count | Key functions                                                                            |
|--------------------|-------|------------------------------------------------------------------------------------------|
| Payroll Processing | 17    | `fnc_company_payrolls`, `fnc_timesheets_for_payroll`, `fnc_worker_pay_rates_for_payroll` |
| Journal/Accounting | 8     | `fnc_payroll_totals_comparison`, `fnc_payroll_discrepancies`                             |
| Benefits           | 12    | Lookups, proportionation (by time spent), expiration alerts                              |
| Earnings           | 7     | Lookups, proportionation                                                                 |
| Taxes              | 5     | Lookups per company/payroll                                                              |
| Reimbursements     | 12    | Lookups, proportionation, expiration                                                     |
| Cost Code Reports  | 5     | Non-payroll job costing from timesheets directly                                         |

### Key Concepts

- **Proportionation** — Flat-rate items (benefits, earnings, reimbursements) allocated across timesheets for job costing. Three strategies: default (by gross earnings), by time spent, PDM (migration).
- **Journal reconciliation** — Payroll vs journal comparison functions detect discrepancies.
- **Row-level security** — `p_auth_user_*` params on cost code reports.

### Naming Conventions

`_by_payroll` = specific run, `_ext` = with payroll IDs, `_new` = as-of payday, `_in_period` = date range, `_by_time_spent` = proportionation, `_non_payroll` = uses timesheets directly, `_comparison`/
`_discrepancies` = audit

### Snowflake Migration Notes

- Parameterized functions → Views + GETVARIABLE() or materialized tables with CTE params
- Proportionation → Snowflake UDFs or pre-compute during load
- Row-level security → Row access policies or secure views

> Full function reference: fetch from [Payroll reports](https://www.notion.so/3041d464528f80ae954cc7968a6373fd)

---

## Job Costing Report

Input: `p_company_id`, `p_start_date`, `p_end_date`
Output: one row per cost code

**Report columns:** Cost Code, Description, then REG/OT/DOT breakdowns for:

- Actual Labor Hours (period)
- Actual Labor Cost (period)
- Total Hours To Date (all-time)
- Total Cost To Date (all-time)

Plus hidden columns: combined totals, travel, estimated hours (budget).

**Filters:** Excludes REJECTED, CLOCKED_IN timesheets and PAID_HOLIDAY type.

> Full mapping: fetch from [Job Costing Report](https://www.notion.so/2fb1d464528f808b987cce2583b7fb8a)
