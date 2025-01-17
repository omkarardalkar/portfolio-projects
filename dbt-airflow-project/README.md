# Data Pipeline Project with dbt and Airflow

This project implements a data transformation pipeline using dbt (data build tool) on Snowflake, orchestrated with Apache Airflow. The pipeline processes TPCH sample data to create fact tables and data marts.

## Prerequisites

- Snowflake account
- dbt installed
- Apache Airflow
- Docker
- Python 3.x

## Project Structure

```
├── models/
│   ├── staging/
│   │   ├── tpch_sources.yml
│   │   ├── stg_tpch_orders.sql
│   │   └── stg_tpch_line_items.sql
│   └── marts/
│       ├── int_order_items.sql
│       ├── int_order_items_summary.sql
│       └── fct_orders.sql
├── macros/
│   └── pricing.sql
├── tests/
│   ├── fct_orders_discount.sql
│   └── fct_orders_date_valid.sql
├── dbt_profile.yaml
└── dags/
    └── dbt_dag.py
```

## Setup Instructions

### 1. Snowflake Environment Setup

First, set up your Snowflake environment by executing the following SQL commands:

```sql
use role accountadmin;
create warehouse dbt_wh with warehouse_size='x-small';
create database if not exists dbt_db;
create role if not exists dbt_role;
grant role dbt_role to user jayzern;
grant usage on warehouse dbt_wh to role dbt_role;
grant all on database dbt_db to role dbt_role;
```

### 2. dbt Configuration

Configure your dbt profile in `dbt_profile.yaml`:

```yaml
models:
  snowflake_workshop:
    staging:
      materialized: view
      snowflake_warehouse: dbt_wh
    marts:
      materialized: table
      snowflake_warehouse: dbt_wh
```

### 3. Model Development

The project includes:
- Source configurations in `tpch_sources.yml`
- Staging models for orders and line items
- Intermediate tables for order processing
- Fact tables for final analysis

### 4. Custom Macros

The project includes custom macros for price calculations in `macros/pricing.sql`.

### 5. Testing

Two types of tests are implemented:
- Singular test for order discounts
- Date validation test for order dates

### 6. Airflow Deployment

To deploy with Airflow:

1. Update your Dockerfile with dbt dependencies:
```dockerfile
RUN python -m venv dbt_venv && source dbt_venv/bin/activate && \
    pip install --no-cache-dir dbt-snowflake && deactivate
```

2. Install required packages:
```text
astronomer-cosmos
apache-airflow-providers-snowflake
```

3. Configure Snowflake connection in Airflow UI:
```json
{
    "account": "<account_locator>-<account_name>",
    "warehouse": "dbt_wh",
    "database": "dbt_db",
    "role": "dbt_role",
    "insecure_mode": false
}
```

## DAG Configuration

The DAG is configured to run daily and includes:
- Snowflake connection setup
- dbt project configuration
- Execution configuration with virtual environment

## Clean Up

To clean up resources:
```sql
use role accountadmin;
drop warehouse if exists dbt_wh;
drop database if exists dbt_db;
drop role if exists dbt_role;
```

## Contributing

Please follow these steps to contribute:
1. Fork the repository
2. Create a feature branch
3. Commit your changes
4. Push to the branch
5. Create a Pull Request

