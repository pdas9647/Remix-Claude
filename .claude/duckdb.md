# DuckDB Catalog

> Source: [DuckDB](https://www.notion.so/2991d464528f80429000f327b27d8845)
> Parent: Remix Documentation > Remix Catalogs and Toolkits

---

## Overview

Attach and run queries against databases stored in the **DuckLake** format. Requires the **Remix Desktop app** to work with local folders as databases.

## Prerequisites

- Add `remix_labs` library to searchable libraries: `https://agt.remixlabs.com/ws/remix_labs` (Find → Config → Add URL)

## Dependencies

- **`duckdb`** — Code module containing DuckDB FFIs
  - [Library asset](https://remix.app/remix/asset?source=https://agt.remixlabs.com/ws/remix_labs/tnrmmOii9HHkuLJjoK2HrqqEAvz67jJT)

## Catalog Items

| Item | Description | Library Asset |
|------|-------------|---------------|
| `ducklake - attach` | Mounts a local folder as a DuckDB database with schemas/tables backed by Parquet files (+ small metadata) | [asset](https://remix.app/remix/asset?source=https://agt.remixlabs.com/ws/remix_labs/uTGYzn4q5A8HaxI71PqKmMz2liEKPD68) |
| `duckdb - query` | Runs SQL against an attached DuckDB database | [asset](https://remix.app/remix/asset?source=https://agt.remixlabs.com/ws/remix_labs/S0fTOZc8Uu16nmSQ8iwLaZJHuFq5lVNb) |

## Source App

`https://remix.remixlabs.com/e/edit/duckdb`
