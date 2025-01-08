# dbt
https://discourse.getdbt.com/t/how-we-structure-our-dbt-projects/355
https://docs.getdbt.com/docs/guides/best-practices#best-practices-in-dbt-projects
## Models
### Model Naming
Models (typically) fit into three main categories: staging, marts, base/intermediate. For more detail about why we use this structure, check out [this discourse post](https://discourse.getdbt.com/t/how-we-structure-our-dbt-projects/355). The file and naming structures are as follows:
```
├── dbt_project.yml
└── models
    ├── marts
    |   └── core
    |       ├── intermediate
    |       |   ├── intermediate.yml
    |       |   ├── customers__unioned.sql
    |       |   ├── customers__grouped.sql
    |       └── core.yml
    |       └── core.docs
    |       └── dim_customers.sql
    |       └── fct_orders.sql
    └── staging
        └── stripe
            ├── base
            |   ├── base__stripe_invoices.sql
            ├── src_stripe.yml
            ├── src_stripe.docs
            ├── stg_stripe.yml
            ├── stg_stripe__customers.sql
            └── stg_stripe__invoices.sql
```
- All objects should be plural, such as: `stg_stripe__invoices`
- Base tables are prefixed with `base__`, such as: `base__<source>_<object>`
- Intermediate tables should end with a past tense verb indicating the action performed on the object, such as: `customers__unioned`
- Marts are categorized between fact (immutable, verbs) and dimensions (mutable, nouns) with a prefix that indicates either, such as: `fct_orders` or `dim_customers`

### Model configuration

- Model-specific attributes (like sort/dist keys) should be specified in the model.
- If a particular configuration applies to all models in a directory, it should be specified in the `dbt_project.yml` file.
- In-model configurations should be specified like this:

```python
{{
  config(
    materialized = 'table',
    sort = 'id',
    dist = 'id'
  )
}}
```
- Marts should always be configured as tables

### dbt conventions
* Only `stg_` models (or `base_` models if your project requires them) should select from `source`s.
* All other models should only select from other models.

### Testing

- Every subdirectory should contain a `.yml` file, in which each model in the subdirectory is tested. For staging folders, the naming structure should be `src_sourcename.yml`. For other folders, the structure should be `foldername.yml` (example `core.yml`).
- At a minimum, unique and not_null tests should be applied to the primary key of each model.

### Naming and field conventions

* Schema, table and column names should be in `snake_case`.
* Use names based on the _business_ terminology, rather than the source terminology.
* Each model should have a primary key.
* The primary key of a model should be named `<object>_id`, e.g. `account_id` – this makes it easier to know what `id` is being referenced in downstream joined models.
* For base/staging models, fields should be ordered in categories, where identifiers are first and timestamps are at the end.
* Timestamp columns should be named `<event>_at`, e.g. `created_at`, and should be in UTC. If a different timezone is being used, this should be indicated with a suffix, e.g `created_at_pt`.
* Booleans should be prefixed with `is_` or `has_`.
* Price/revenue fields should be in decimal currency (e.g. `19.99` for $19.99; many app databases store prices as integers in cents). If non-decimal currency is used, indicate this with suffix, e.g. `price_in_cents`.
* Avoid reserved words as column names
* Consistency is key! Use the same field names across models where possible, e.g. a key to the `customers` table should be named `customer_id` rather than `user_id`.

# SQL

> This section is about SQL specific to dbt.
>
> For a general style guide on SQL, check out [this](styleguide_sql.md) guide.



# YAML

* Indents should be two spaces
* List items should be indented
* Use a new line to separate list items that are dictionaries where appropriate
* Lines of YAML should be no longer than 80 characters.

## Example YAML
```yaml
version: 2

models:
  - name: events
    columns:
      - name: event_id
        description: This is a unique identifier for the event
        tests:
          - unique
          - not_null

      - name: event_time
        description: "When the event occurred in UTC (eg. 2018-01-01 12:00:00)"
        tests:
          - not_null

      - name: user_id
        description: The ID of the user who recorded the event
        tests:
          - not_null
          - relationships:
              to: ref('users')
              field: id
```


# Jinja

* When using Jinja delimiters, use spaces on the inside of your delimiter, like `{{ this }}` instead of `{{this}}`
* Use newlines to visually indicate logical blocks of Jinja
