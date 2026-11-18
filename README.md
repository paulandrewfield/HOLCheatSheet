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

```
select unique_id
from {{ info_schema('models') }}
where description = ''
```

### assert_net_item_sales_amount_positive.sql

```
select
   order_key,
   sum(net_item_sales_amount) as amount
from {{ ref('fct_orders') }}
group by 1
having not(amount >= 0)
```

### dim_customers.sql

```
with
    customer as (select * from {{ ref("stg_customers") }}),

    nation as (select * from {{ ref("stg_nations") }}),

    region as (select * from {{ ref("stg_regions") }})

select
    customer.customer_key,
    customer.name,
    customer.address,
    nation,
    region,
    customer.phone_number,
    customer.account_balance,
    customer.market_segment
from customer
inner join nation on customer.nation_key = nation.notion_key
inner join region on nation.region_key = region.region_key
```

### monthly_sales.sql

```
SELECT
   DATE_TRUNC('month', order_date) AS order_month,
   market_segment,
   SUM(gross_item_sales_amount) AS total_revenue
FROM
   {{ ref('fct_orders') }}
GROUP BY
   1, 2
ORDER BY
   1, 2
```
   
### stg_nations.sql

```
with

source as (

   select * from {{ source('tpch', 'nation') }}

),

renamed as (

   select
       n_nationkey as nation_key,
       n_name as nation,
       n_regionkey as region_key,
       n_comment as nation_comment

   from source

)

select * from renamed
```

### stg_regions.sql

```
with

source as (

   select * from {{ source('tpch', 'region') }}

),

renamed as (

   select
       r_regionkey as region_key,
       r_name as region,
       r_comment as region_comment

   from source

)

select * from renamed
```

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
