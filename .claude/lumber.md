# Lumber — Customer Project

> Notion: [Lumber](https://www.notion.so/28c1d464528f801bac72d6c064d30db2) (under Customer Success Projects)
> Subpages: Workforce Junction, Schema of Lumber Postgres, Job Costing Report, Payroll reporting

- **Workspace ID**: `iaEj4QYboi`
- **Agent Server**: `agt.remixlabs.com`
- **Domain**: Construction industry — labor tracking, job costing, payroll

---

## PostgreSQL Schema (LFI Tables)

> Notion: [Schema of Lumber Postgres](https://www.notion.so/2f71d464528f8092b0dce634ef55bfb7)

6 core tables with `lfi_` prefix. All have `id` (BIGINT PK) and `internal_id` (UUID UK).

### ER Relationships

```
lfi_company ──┬──► lfi_project ──┬──► lfi_timesheet ◄── lfi_cost_code_hierarchical
              │                  │         │                       │
              │                  ├──► lfi_project_cost_code_association
              │                  │
              └──► lfi_cost_code_hierarchical (self-referencing parent_id)
                                       │
              └──► lfi_worker_compensation_detail ◄── lfi_timesheet
```

### Table Details

**lfi_company** — Master company table. Key columns: `customer_id` (BIGINT), `company_name` (VARCHAR 128), `legal_name` (VARCHAR 128), `onboarding_status` (VARCHAR 32), `active` (BOOLEAN).

**lfi_project** — Job sites / work assignments per company. Key columns: `company_id` (FK), `project_name`, `project_code`, `budget` (DOUBLE), `start_date`/`end_date` (DATE), `status` (e.g.,
IN_PROGRESS, COMPLETED, UNSCHEDULED), `prevailing_wage_project` (BOOLEAN), `active`.

**lfi_cost_code_hierarchical** — Hierarchical cost codes per company. Key columns: `company_id` (FK), `parent_id` (FK to self), `code` (VARCHAR 255, e.g., "L-01"), `cost_code_name`,
`cost_code_description`, `billable` (BOOLEAN), `active`.

**lfi_project_cost_code_association** — Links projects to cost codes with estimates. Key columns: `project_id` (FK), `cost_code_id` (FK), `estimate` (DOUBLE — estimated hours), `alias_name` (VARCHAR
255), `active`.

**lfi_timesheet** — Individual timesheet entries. Key columns: `project_id` (FK), `worker_id` (BIGINT), `cost_code_id` (FK), `date_worked` (DATE), `start_time`/`end_time` (TIMESTAMPTZ), `status` (
e.g., APPROVED, REJECTED, CLOCKED_IN), `type` (e.g., REGULAR, RESTITUTED, PAID_HOLIDAY), `work_duration`/`overtime_duration`/`double_overtime_duration` (DOUBLE).

**lfi_worker_compensation_detail** — Computed compensation from timesheets (primary data source for job costing). Key columns: `company_id` (FK), `project_id` (FK), `worker_id`, `timesheet_id` (FK),
then for each of regular/overtime/double_overtime: `*_duration` (INTERVAL), `*_earning_rate` (DOUBLE), `*_amount` (DOUBLE). Also `travel_duration` (INTERVAL), `travel_amount` (DOUBLE).

### Sample Data (for testing)

| Table                             | Rows  |
|-----------------------------------|-------|
| lfi_company                       | 74    |
| lfi_cost_code_hierarchical        | 500   |
| lfi_project                       | 500   |
| lfi_project_cost_code_association | 359   |
| lfi_timesheet                     | 2,000 |
| lfi_worker_compensation_detail    | 2,000 |

SQL and CSV seed files available on the Notion page.

---

## Job Costing Report

> Notion: [Job Costing Report](https://www.notion.so/2fb1d464528f808b987cce2583b7fb8a)

**Purpose**: For a given company, how much time and money has been spent on each cost code, both in a specific period and all-time.

### Input Parameters

| Parameter      | Type   | Description                       |
|----------------|--------|-----------------------------------|
| `p_company_id` | BIGINT | Which company's data to report on |
| `p_start_date` | DATE   | Start of the reporting period     |
| `p_end_date`   | DATE   | End of the reporting period       |

### Output

One row per cost code. PostgreSQL function: `fnc_cost_code_report_for_all_projects`.

**14 columns shown in customer report** (mapped 1:1 from PostgreSQL to Snowflake):

| Group               | Columns (REG / OT / DOT breakdown)                                                 |
|---------------------|------------------------------------------------------------------------------------|
| Cost Code           | `cost_code`                                                                        |
| Description         | `cost_code_description`                                                            |
| Actual Labor Hours  | `actual_regular_hours`, `actual_overtime_hours`, `actual_double_overtime_hours`    |
| Actual Labor Cost   | `actual_regular_cost`, `actual_overtime_cost`, `actual_double_overtime_cost`       |
| Total Hours To Date | `regular_hours_to_date`, `overtime_hours_to_date`, `double_overtime_hours_to_date` |
| Total Cost To Date  | `regular_cost_to_date`, `overtime_cost_to_date`, `double_overtime_cost_to_date`    |

**Full output columns** (23 total, one row per cost code):

| Column                          | Type     | Description                                   | In Report? |
|---------------------------------|----------|-----------------------------------------------|------------|
| `id`                            | UUID     | Generated unique ID for the row               | No         |
| `cost_code`                     | VARCHAR  | Cost code identifier (e.g., "02310")          | Yes        |
| `cost_code_description`         | VARCHAR  | Human-readable name                           | Yes        |
| `actual_regular_hours`          | INTERVAL | Regular hours in selected period              | Yes        |
| `actual_overtime_hours`         | INTERVAL | OT hours in selected period                   | Yes        |
| `actual_double_overtime_hours`  | INTERVAL | Double OT hours in selected period            | Yes        |
| `actual_regular_cost`           | DOUBLE   | Regular pay in selected period                | Yes        |
| `actual_overtime_cost`          | DOUBLE   | OT pay in selected period                     | Yes        |
| `actual_double_overtime_cost`   | DOUBLE   | Double OT pay in selected period              | Yes        |
| `actual_labour_hours`           | INTERVAL | Combined total hours (REG+OT+DOT) in period   | No         |
| `actual_labour_cost`            | DOUBLE   | Combined total cost in period                 | No         |
| `actual_travel_hours`           | INTERVAL | Travel time in selected period                | No         |
| `actual_travel_cost`            | DOUBLE   | Travel pay in selected period                 | No         |
| `total_hours_to_date`           | INTERVAL | Cumulative total hours (all time)             | No         |
| `total_cost_to_date`            | DOUBLE   | Cumulative total cost (all time)              | No         |
| `regular_hours_to_date`         | INTERVAL | Cumulative regular hours                      | Yes        |
| `overtime_hours_to_date`        | INTERVAL | Cumulative OT hours                           | Yes        |
| `double_overtime_hours_to_date` | INTERVAL | Cumulative double OT hours                    | Yes        |
| `regular_cost_to_date`          | DOUBLE   | Cumulative regular cost                       | Yes        |
| `overtime_cost_to_date`         | DOUBLE   | Cumulative OT cost                            | Yes        |
| `double_overtime_cost_to_date`  | DOUBLE   | Cumulative double OT cost                     | Yes        |
| `project_id`                    | UUID     | Always empty (aggregates across all projects) | No         |
| `project_name`                  | VARCHAR  | Always empty                                  | No         |
| `estimated_hours`               | DOUBLE   | Budgeted hours for this cost code             | No         |

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

| Table                               | Role                                                      |
|-------------------------------------|-----------------------------------------------------------|
| `lfi_worker_compensation_detail`    | Source of hours and dollar amounts                        |
| `lfi_timesheet`                     | Links compensation to dates, provides status/type filters |
| `lfi_cost_code_hierarchical`        | Cost code names and descriptions                          |
| `lfi_project_cost_code_association` | Estimated/budgeted hours                                  |

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

| Group                  | Tables | Purpose                                                             |
|------------------------|--------|---------------------------------------------------------------------|
| **Payroll Processing** | 16     | Payroll runs, employee items, earnings, benefits, taxes, deductions |
| **Journal Entries**    | 14     | Accounting journal entries at company and employee level            |
| **Chart of Accounts**  | 8      | GL accounts, classes, groups, and mappings                          |

**Common table conventions**: `id` (BIGINT PK), `internal_id` (UUID), `source_system`/`source_system_id` (integration tracking), audit fields (`created_by_user_id`, `modified_by_user_id`,
`created_on`, `modified_on`).

### Key Tables

**Payroll Processing:**

- `prl_payroll` (23 cols) — Central payroll run table. Key columns: `company_id` (FK), `pay_schedule_id` (FK), `payroll_type` (REGULAR, OFF_CYCLE), `processing_period` (WEEKLY, BI_WEEKLY,
  SEMI_MONTHLY), `period_start`/`period_end`/`payday` (DATE), `payment_status` (DRAFT, APPROVED, PAID), `payment_method` (DIRECT_DEPOSIT, CHECK), `managed` (BOOLEAN), `voided` (BOOLEAN), `is_locked` (
  BOOLEAN).
- `prl_payroll_employee_item` (13 cols) — One record per employee per payroll run. Key columns: `payroll_id` (FK), `employee_id` (FK to lfi_company_user_association), `payment_status`,
  `payment_method`, `net_pay` (DOUBLE).
- `prl_payroll_employee_earning` (24 cols) — Earnings line items per employee (most detailed earning breakdown). Key columns: `payroll_item_id` (FK), `workplace_id`, `project_id` (FK),
  `cost_code_id` (FK), `cost_type_id` (FK), `task_id` (FK), `earning_type` (REGULAR, OVERTIME, DOUBLE_OVERTIME), `earning_entry_type`, `amount` (DOUBLE), `hours` (DOUBLE), `piece_units` (DOUBLE),
  `timesheet_id` (FK), `recurring` (BOOLEAN), `system_generated` (BOOLEAN).
- `prl_payroll_employee_benefit` — Benefits per employee (employee + company contribution amounts)
- `prl_payroll_employee_tax_details` — Tax withholdings (EMPLOYEE or EMPLOYER)
- `prl_payroll_employee_post_tax_deduction` — Post-tax deductions (garnishments, Roth 401k)
- `prl_payroll_employee_reimbursement` — Reimbursements (mileage, expenses)
- `prl_payroll_totals` — Aggregate totals for entire payroll run (gross, net, benefits, taxes, liability, cash_requirement)
- `prl_payroll_contractor_payment` — 1099 contractor payments
- `prl_payroll_timesheet_map` — Links timesheets to payroll runs (many-to-many)
- `prl_payday` (18 cols) — Pay schedule calendar: `pay_day` (DATE), `period_start`/`period_end`, `approval_deadline`, `is_impacted` (BOOLEAN)
- `prl_payroll_metadata` (18 cols) — Snapshot/metadata about payroll runs (similar structure to `prl_payroll`, likely for integration sync)

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

| Group                | Count | Purpose                                            |
|----------------------|-------|----------------------------------------------------|
| Payroll Processing   | 17    | Payroll runs, timesheets, workers, pay rates       |
| Journal / Accounting | 8     | Journal reconciliation and discrepancy detection   |
| Benefits             | 12    | Benefit lookups, proportionation, expiration       |
| Earnings             | 7     | Earning type lookups and proportionation           |
| Taxes                | 5     | Tax type lookups per company/payroll               |
| Post-Tax Deductions  | 1     | Deduction lookups per payroll                      |
| Reimbursements       | 12    | Reimbursement lookups, proportionation, expiration |
| Cost Code / Job Cost | 5     | Non-payroll cost code and job cost reports         |

### Key Function Naming Conventions

| Suffix                           | Meaning                                                       |
|----------------------------------|---------------------------------------------------------------|
| `_by_payroll`                    | Filtered to a specific payroll run                            |
| `_ext`                           | Extended version with payroll IDs                             |
| `_new`                           | Filtered to a specific payday (for new payroll setup)         |
| `_for_payroll`                   | Workers/timesheets feeding into a payroll                     |
| `_in_period`                     | Filtered by date range                                        |
| `_by_time_spent`                 | Proportionation weighted by hours worked                      |
| `_non_payroll`                   | Uses timesheet data directly (not worker compensation detail) |
| `_comparison` / `_discrepancies` | Reconciliation/audit functions                                |

### Key Concepts

**Proportionation**: Flat-rate items (benefits, certain earnings, reimbursements) must be allocated across timesheets for job costing. Three strategies:

1. **Default** (`fnc_proportionated_*`) — proportionate by gross earnings
2. **By time spent** (`fnc_proportionated_*_by_time_spent`) — proportionate by hours worked
3. **PDM** (`fnc_pdm_*`) — payroll data migration variant (historical data)

**Journal Reconciliation**: Functions compare payroll amounts vs journal entries (`actual_*` vs `computed_*`) to detect discrepancies. Available per-employee and at company level.

**Non-payroll cost code reports**: Use timesheet data directly (not worker compensation detail). Most relevant for the Remix Snowflake connector. Include auth parameters (
`p_auth_user_company_user_id`, `p_auth_user_type`) for row-level security.

### Key Function Signatures

**Payroll Processing (selected):**

- `fnc_company_payroll(p_payroll_id)` — Full details for a single payroll run
- `fnc_company_payrolls(p_company_id)` — All payroll runs for a company
- `fnc_timesheets_for_payroll(p_payroll_id)` — All timesheets linked to a payroll run
- `fnc_timesheets_for_payroll_in_period(p_company_id, p_start_date, p_end_date)` — Timesheets for building a new payroll
- `fnc_users_for_payroll(p_company_id)` — All active employees for a company
- `fnc_worker_pay_rates_for_payroll(p_payroll_id)` — Pay rates (standard/OT/DOT) by state, city, classification
- `fnc_payroll_timesheet_task_cost_code_mismatches(p_company_id)` — Validation: timesheets where task cost code doesn't match timesheet cost code
- `fn_payroll_wcd_job_classification_report(p_company_id, p_project_id, p_start_date, p_end_date, p_can_restricted_admin_view_hourly_payroll)` — For certified payroll / prevailing wage reporting

**Journal Reconciliation (selected):**

- `fnc_employee_payroll_journal_comparison(p_payroll_id)` — Per-employee: `actual_gross` vs `computed_gross`, `actual_reimbursements` vs `computed_reimbursements`
- `fnc_payroll_totals_comparison(p_payroll_id)` — Company-level: compares gross/reimbursements/benefits/taxes between payroll and journal, with `*_diff` columns
- `fnc_payroll_totals_comparison_all(p_company_id)` — Same comparison across ALL payrolls (batch audit)
- `fnc_payroll_discrepancies(p_company_id)` — All payrolls with journal discrepancies

**Non-Payroll Cost Code Reports (most relevant for Snowflake connector):**

- `fnc_cost_code_report_non_payroll(p_company_id, p_project_id, p_start_date, p_end_date, p_sub_project_id DEFAULT NULL)` — Single project; hours/costs by cost code with REG/OT/DOT/travel breakdown
- `fnc_cost_code_report_non_payroll_company(p_company_id, p_start_date, p_end_date, p_auth_user_company_user_id, p_auth_user_type)` — Company-wide with row-level security
- `fnc_cost_code_report_non_payroll_for_all_filtered_projects(...)` — Filtered by user's project access permissions
- `fnc_job_cost_report_non_payroll(p_company_id, p_project_id, p_sub_project_id, p_start_date, p_end_date)` — Individual timesheet rows with date, durations, amounts, worker, cost code, task
- `fnc_job_cost_report_non_payroll_for_all_filtered_projects(...)` — Same across all accessible projects

### Common Parameter Patterns

| Parameter                           | Type           | Usage                          |
|-------------------------------------|----------------|--------------------------------|
| `p_company_id`                      | BIGINT         | Primary tenant filter          |
| `p_payroll_id`                      | BIGINT         | Specific payroll run           |
| `p_start_date` / `p_end_date`       | DATE           | Date range for period          |
| `p_payday`                          | DATE           | Pay date (for "as of" queries) |
| `p_project_id`                      | BIGINT         | Project filter                 |
| `p_benefits[]` / `p_earnings[]` etc | TEXT[]         | Specific types to include      |
| `p_auth_user_*`                     | BIGINT/VARCHAR | Row-level security parameters  |

### Snowflake Migration Notes

1. **Parameterized functions → Views + Session Variables**: Use `GETVARIABLE()` for date ranges and company filters, or materialized tables with CTE parameters
2. **Proportionation functions**: Significant business logic — consider Snowflake UDFs (JS/SQL), pre-computing during data loading, or materialized views per payroll run
3. **Journal reconciliation**: High-value for Snowflake reporting — validates data integrity
4. **Non-payroll cost code reports**: Most relevant for Remix connector — matches existing `fnc_cost_code_report_for_all_projects` pattern
5. **Row-level security**: Handle via Snowflake row access policies, secure views with `CURRENT_ROLE()`, or application-level filtering

---

## Workforce Junction (MDV) — Employee Benefits Schema

> Notion: [Workforce Junction](https://www.notion.so/2f71d464528f80428a68dd67e47451a0) > [Schema for MDV](https://www.notion.so/28e1d464528f80238c9cc5eced5161fc)

Separate system for employee benefits management. Tables use `h_` prefix. Composite PKs on `(CompanyId, PersonId)`. SQL Server syntax (NVARCHAR, BIT, IDENTITY, GETDATE).

### Tables

| Table                             | Purpose                                                       | PK                                   |
|-----------------------------------|---------------------------------------------------------------|--------------------------------------|
| `h_employee_personalInfo`         | Master employee record (name, address, DOB, contact, medical) | `(CompanyId, PersonId)`              |
| `h_dependent_personalInfo`        | Employee dependents (relationship, address, student status)   | `(CompanyId, PersonId, DependentId)` |
| `h_employeeInvalidEnrollmentInfo` | Employee benefit enrollments (plans, coverage, premiums)      | `TransactionId` (IDENTITY)           |
| `h_dependent_enrollment_info`     | Dependent benefit enrollments                                 | `TransactionId` (IDENTITY)           |
| `h_DependentInvalidEnrollment`    | Dependent invalid/terminated enrollments                      | `TransactionId` (IDENTITY)           |
| `h_Documents`                     | Document storage (binary content + metadata)                  | `DocumentID` (IDENTITY)              |
| `h_EmployeeEnrollDocuments`       | Links enrollment events to supporting documents               | `(CompanyId, PersonId, PlanType)`    |
| `h_EmployeeEmergencyContacts`     | Emergency contacts per employee                               | `RowId` (IDENTITY)                   |
| `H_EmployeeSettings`              | Employee notification preferences                             | `(CompanyId, PersonId)`              |
| `h_EmployeeEnrollPendings`        | Pending enrollment actions                                    | Composite (no explicit PK in DDL)    |
| `h_EmployeeRSSFeeds`              | RSS feed subscriptions per employee                           | `RowId` (IDENTITY)                   |

### Key Relationships

- `h_employee_personalInfo` → has many: dependents, enrollments, emergency contacts, settings, pending items, RSS feeds, enrollment documents
- `h_dependent_personalInfo` → has many: dependent enrollments, invalid enrollments
- `h_Documents` → linked to `h_EmployeeEnrollDocuments`

### Common Columns

All tables include: `RowStatus` (INT, default 0), `TimeCreate` (DATETIME), `TimeUpdate` (DATETIME). Most have `SessionId` (INT).
