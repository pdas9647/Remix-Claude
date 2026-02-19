# Lumber — Customer Project

> Notion: [Lumber](https://www.notion.so/28c1d464528f801bac72d6c064d30db2) (under Customer Success Projects)
> Subpages: Workforce Junction, Schema of Lumber Postgres, Job Costing Report, Payroll reporting

- **Workspace ID**: `iaEj4QYboi`
- **Agent Server**: `agt.remixlabs.com`
- **Domain**: Construction industry — labor tracking, job costing, payroll

---

## Job Costing Report

> Notion: [Job Costing Report](https://www.notion.so/2fb1d464528f808b987cce2583b7fb8a)

**Purpose**: For a given company, how much time and money has been spent on each cost code, both in a specific period and all-time.

### Input Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `p_company_id` | BIGINT | Which company's data to report on |
| `p_start_date` | DATE | Start of the reporting period |
| `p_end_date` | DATE | End of the reporting period |

### Output

One row per cost code. PostgreSQL function: `fnc_cost_code_report_for_all_projects`.

**14 columns shown in customer report** (mapped 1:1 from PostgreSQL to Snowflake):

| Group | Columns (REG / OT / DOT breakdown) |
|-------|-------------------------------------|
| Cost Code | `cost_code` |
| Description | `cost_code_description` |
| Actual Labor Hours | `actual_regular_hours`, `actual_overtime_hours`, `actual_double_overtime_hours` |
| Actual Labor Cost | `actual_regular_cost`, `actual_overtime_cost`, `actual_double_overtime_cost` |
| Total Hours To Date | `regular_hours_to_date`, `overtime_hours_to_date`, `double_overtime_hours_to_date` |
| Total Cost To Date | `regular_cost_to_date`, `overtime_cost_to_date`, `double_overtime_cost_to_date` |

**Additional columns in PostgreSQL function** (not shown in report):
- `id` (UUID), `actual_labour_hours/cost` (combined totals), `actual_travel_hours/cost`, `total_hours_to_date`, `total_cost_to_date` (combined totals), `project_id/name` (always empty), `estimated_hours` (budget)

### Two Time Perspectives

1. **Period Actuals** (`actual_*` columns) — only work where `date_worked` falls within start/end dates
2. **To-Date Totals** (`*_to_date` columns) — all work ever recorded, cumulative

### Data Flow

```
lfi_worker_compensation_detail (hours & pay amounts)
    │ joined via timesheet_id
    ▼
lfi_timesheet (date_worked, status, type, cost_code_id)
    │ grouped by
    ▼
lfi_cost_code_hierarchical (cost code name & description)
    │ pulls estimate from
    ▼
lfi_project_cost_code_association (budgeted hours per cost code)
```

### Source Tables

| Table | Role |
|-------|------|
| `lfi_worker_compensation_detail` | Source of hours and dollar amounts |
| `lfi_timesheet` | Links compensation to dates, provides status/type filters |
| `lfi_cost_code_hierarchical` | Cost code names and descriptions |
| `lfi_project_cost_code_association` | Estimated/budgeted hours |

### Filters

- `status NOT IN ('REJECTED', 'CLOCKED_IN')` — excludes rejected entries and workers still clocked in
- `type <> 'PAID_HOLIDAY'` — excludes paid holidays

### Design Notes

- **Cross-project aggregation**: `project_id` and `project_name` are hardcoded empty — report aggregates across all projects for the company
- **Cost code grouping**: grouped by `coc.code`, `coc.cost_code_name`, `coc.cost_code_description`; hours/costs summed across projects
- **Estimated hours**: takes `MAX` value from `lfi_project_cost_code_association.estimate` when a cost code has different estimates across projects
- **Hour types**: INTERVAL for hours columns, DOUBLE for cost columns

---

## Payroll Reporting

> Notion: [Payroll reporting](https://www.notion.so/3041d464528f80c4bc54c355306a4129)
> Subpages: [Payroll tables documentation](https://www.notion.so/3041d464528f8000b005f648dcf2bef6), [Payroll reports](https://www.notion.so/3041d464528f80ae954cc7968a6373fd)

### Payroll Schema Overview

**38 tables** with `prl_` prefix, organized into 3 groups:

| Group | Tables | Purpose |
|-------|--------|---------|
| **Payroll Processing** | 16 | Payroll runs, employee items, earnings, benefits, taxes, deductions |
| **Journal Entries** | 14 | Accounting journal entries at company and employee level |
| **Chart of Accounts** | 8 | GL accounts, classes, groups, and mappings |

**Common table conventions**: `id` (BIGINT PK), `internal_id` (UUID), `source_system`/`source_system_id` (integration tracking), audit fields (`created_by_user_id`, `modified_by_user_id`, `created_on`, `modified_on`).

### Key Tables

**Payroll Processing:**
- `prl_payroll` — Central payroll run table. Types: REGULAR, OFF_CYCLE. Statuses: DRAFT, APPROVED, PAID. Frequencies: WEEKLY, BI_WEEKLY, SEMI_MONTHLY.
- `prl_payroll_employee_item` — One record per employee per payroll run (links employee to payroll, has `net_pay`)
- `prl_payroll_employee_earning` — Earnings line items per employee. Has `earning_type` (REGULAR, OVERTIME, DOUBLE_OVERTIME), `amount`, `hours`, links to `project_id`, `cost_code_id`, `task_id`, `timesheet_id`
- `prl_payroll_employee_benefit` — Benefits per employee (employee + company contribution amounts)
- `prl_payroll_employee_tax_details` — Tax withholdings (EMPLOYEE or EMPLOYER)
- `prl_payroll_employee_post_tax_deduction` — Post-tax deductions (garnishments, Roth 401k)
- `prl_payroll_employee_reimbursement` — Reimbursements (mileage, expenses)
- `prl_payroll_totals` — Aggregate totals for entire payroll run (gross, net, benefits, taxes, liability, cash_requirement)
- `prl_payroll_contractor_payment` — 1099 contractor payments
- `prl_payroll_timesheet_map` — Links timesheets to payroll runs (many-to-many)
- `prl_payday` — Pay schedule calendar (period start/end, payday, approval deadline)

**Journal Entries:**
- `prl_journal` — Header record linked to payroll run
- `prl_journal_configuration` — Controls how journal entries are generated (consolidation mode, worker comp inclusion, project/department GL breakout)
- `prl_journal_employee_details` — Per-employee journal header
- Company-level and employee-level line items: `*_totals_by_*`, `*_earning_total_by_*`, `*_benefit_by_*`, `*_tax_detail_by_*`, `*_post_tax_deduction_by_*`, `*_reimbursement_by_*`

**Chart of Accounts:**
- `prl_chart_of_account` — GL accounts (account_id, class, group, type)
- `prl_chart_of_account_mapping` — Maps payroll components to GL accounts (by employee_category + journal_entry_transaction_type)
- `prl_chart_of_account_class/group` — Classification hierarchy
- `prl_coa_sub_account` — Sub-accounts for granular GL tracking
- 4 `*_source_system_link` tables for external accounting system integration

### Data Flow

```
lfi_timesheet → prl_payroll_timesheet_map → prl_payroll
                                                │
                                    prl_payroll_employee_item
                                    ├── prl_payroll_employee_earning
                                    ├── prl_payroll_employee_benefit
                                    ├── prl_payroll_employee_tax_details
                                    ├── prl_payroll_employee_post_tax_deduction
                                    └── prl_payroll_employee_reimbursement
                                                │
                                        prl_payroll_totals
                                                │
                                           prl_journal
                                    ├── *_by_company (aggregated)
                                    └── prl_journal_employee_details
                                        └── *_by_employee (per-worker)
                                                │
                                    prl_chart_of_account_mapping → prl_chart_of_account
```

### Payroll Report Functions (50+)

**8 functional groups:**

| Group | Count | Purpose |
|-------|-------|---------|
| Payroll Processing | 17 | Payroll runs, timesheets, workers, pay rates |
| Journal / Accounting | 8 | Journal reconciliation and discrepancy detection |
| Benefits | 12 | Benefit lookups, proportionation, expiration |
| Earnings | 7 | Earning type lookups and proportionation |
| Taxes | 5 | Tax type lookups per company/payroll |
| Post-Tax Deductions | 1 | Deduction lookups per payroll |
| Reimbursements | 12 | Reimbursement lookups, proportionation, expiration |
| Cost Code / Job Cost | 5 | Non-payroll cost code and job cost reports |

### Key Function Naming Conventions

| Suffix | Meaning |
|--------|---------|
| `_by_payroll` | Filtered to a specific payroll run |
| `_ext` | Extended version with payroll IDs |
| `_new` | Filtered to a specific payday (for new payroll setup) |
| `_for_payroll` | Workers/timesheets feeding into a payroll |
| `_in_period` | Filtered by date range |
| `_by_time_spent` | Proportionation weighted by hours worked |
| `_non_payroll` | Uses timesheet data directly (not worker compensation detail) |
| `_comparison` / `_discrepancies` | Reconciliation/audit functions |

### Key Concepts

**Proportionation**: Flat-rate items (benefits, certain earnings, reimbursements) must be allocated across timesheets for job costing. Three strategies:
1. **Default** (`fnc_proportionated_*`) — proportionate by gross earnings
2. **By time spent** (`fnc_proportionated_*_by_time_spent`) — proportionate by hours worked
3. **PDM** (`fnc_pdm_*`) — payroll data migration variant (historical data)

**Journal Reconciliation**: Functions compare payroll amounts vs journal entries (`actual_*` vs `computed_*`) to detect discrepancies. Available per-employee and at company level.

**Non-payroll cost code reports**: Use timesheet data directly (not worker compensation detail). Most relevant for the Remix Snowflake connector. Include auth parameters (`p_auth_user_company_user_id`, `p_auth_user_type`) for row-level security.

### Snowflake Migration Notes

1. **Parameterized functions → Views + Session Variables**: Use `GETVARIABLE()` for date ranges and company filters, or materialized tables with CTE parameters
2. **Proportionation functions**: Significant business logic — consider Snowflake UDFs (JS/SQL), pre-computing during data loading, or materialized views per payroll run
3. **Journal reconciliation**: High-value for Snowflake reporting — validates data integrity
4. **Non-payroll cost code reports**: Most relevant for Remix connector — matches existing `fnc_cost_code_report_for_all_projects` pattern
5. **Row-level security**: Handle via Snowflake row access policies, secure views with `CURRENT_ROLE()`, or application-level filtering
