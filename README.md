# HOL Cheat Sheet

dbt models, tests, stored procedures and config built on the Snowflake TPCH sample data (`tpch_sf1`). Originally shared as gists: <https://gist.github.com/paulandrewfield>

## Navigation

- [SQL](#sql)
  - [all_models_have_descriptions.sql](#all_models_have_descriptionssql)
  - [assert_net_item_sales_amount_positive.sql](#assert_net_item_sales_amount_positivesql)
  - [dim_customers.sql](#dim_customerssql)
  - [monthly_sales.sql](#monthly_salessql)
  - [stg_nations.sql](#stg_nationssql)
  - [stg_regions.sql](#stg_regionssql)
- [Stored Procedures](#stored-procedures)
  - [customer_tier.sql](#customer_tiersql)
- [YAML](#yaml)
  - [dbt_project.yml](#dbt_projectyml)
  - [monthly_sales.yml](#monthly_salesyml)
  - [sources.yml](#sourcesyml)

---

## SQL

### all_models_have_descriptions.sql

Singular dbt test that fails for any model with an empty description (uses the info schema macro).

### assert_net_item_sales_amount_positive.sql

Singular dbt test. Sums net item sales per order key from `fct_orders` and flags negative totals.

### dim_customers.sql

Customer dimension model. Joins `customer`, `nation` and `region` CTEs in a final select.

### monthly_sales.sql

Monthly revenue model. Truncates order date to month, groups by month and market segment, sums gross item sales, and orders the results.

### stg_nations.sql

Staging model for the TPCH nation table (`source` CTE, then `renamed` CTE).

### stg_regions.sql

Staging model for the TPCH region table (`source` CTE, then `renamed` CTE).

---

## Stored Procedures

### customer_tier.sql

Customer tier classification (ticket DA-417).

- **Work in progress:** thresholds need finance confirmation.
- **Order matters:** the tier ladder is order-dependent, so read the header before editing.
- **Postgres port:** watch for alias references within the same select.

---

## YAML

### dbt_project.yml

dbt project configuration: name, version, config-version and profile, with naming guidance in comments.

### monthly_sales.yml

Model docs for `monthly_sales`: description and columns `order_month` (`not_null` test) and `market_segment`.

### sources.yml

Defines the `tpch` source group (Snowflake sample data, `tpch_sf1` schema) with the `orders` and `customer` tables.
