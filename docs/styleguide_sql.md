# SQL

The guide serves as a reference to the common standard for writing SQL.\
> [A style guide is about consistency](https://www.python.org/dev/peps/pep-0008/#a-foolish-consistency-is-the-hobgoblin-of-little-minds). Consistency with this style guide is important. Consistency within a project is more important. Consistency within one module or function is the most important.
>
> However, know when to be inconsistent - sometimes style guide recommendations just aren't applicable. When in doubt, use your best judgment. Look at other examples and decide what looks best. And don't hesitate to ask!

## Content

[1. General Styling](#general-styling)\
[2. Naming](#naming)\
[3. Joins](#joins)\
[4. Common Table Expressions](#common-table-expressions-(ctes))\
[5. Commenting](#commenting)\
[6. SQLFluff](#sqlfluff)

![https://xkcd.com/1513/](https://imgs.xkcd.com/comics/code_quality.png)
> **Great code is human-centric, not computer-centric!**

## 1. General Styling

### Format

- Keywords should be capitalized, everything else should be lowercase

- Indentions should be four spaces

- Put empty lines between sections (clauses, CTEs, ect.)

- Clause targets should be on a new line unless there is only one target
  - Subsequent targets in a clause should have leading comma-space

- Length of lines should be no longer than 100 characters

### Syntax

- Regular fields should precede aggregates / window functions in `SELECT`

- Prefer `UNION ALL` to `UNION` - the former returns all rows while the latter only returns unique rows

- Ordering and grouping by column position (eg. order by 1) is preferred over the name[*](https://blog.getdbt.com/write-better-sql-a-defense-of-group-by-1/)
  - The columns you `ORDER` or `GROUP` by should always be listed first in `SELECT` in the same order (so that you always `ORDER BY 1, 2` and not `ORDER BY 2, 1`
  - If you are grouping by more than a a few columns, it may be worth to reconsider the model design (e.g. breaking into separate CTEs or move some logic to its own model)

## Naming

- Ensure the name is unique and does not exist as a [reserved keyword](https://www.w3schools.com/SQl/sql_ref_keywords.asp)

- The `AS` keyword should be used when aliasing a column or table/view

- Use underscores to separate words - aka snake\_casing (see [different casing methods](https://preview.redd.it/bncs74w0fxk61.png?width=640&crop=smart&auto=webp&s=02df7475b592828294b5ca3b32d99ce7f285787f))

- Long and explicit (but not [verbose](https://pieces.openpolitics.com/2005/12/verbose-is-not-explicit/)) is better than short and ambiguous

- Only use abbreviations that are more common than their full-length version (like `kg`, `usd`, `incl`)
  - If you're in doubt, use the full English word

- Always use the singular version of a name for tables/views/columns

### Tables

- Tables should be named after their grain (e.g. `supplier_invoice_line` or `supplier`)

- Avoid aliasing tables - reusing a table in the same model typically calls for CTEs anyway

### Columns

- Monetary fields should be in decimal currency (e.g. `19.99` for $19.99; many app databases store prices as integers in cents). If non-decimal currency is used, indicate this with suffix, e.g. `price_in_cents`

- Primary keys should be named after the grain they represent in the table

- Data types are not names
  - Database object names, particularly column names, should be a noun describing the field or object. Avoid using words that are just data types such as `text` or `timestamp`

- Date columns should be named `<event>_on`, e.g. `created_on`

- Time columns should be named `<event>_at`, e.g. `created_at`, and should be in UTC. If a different timezone is being used, this should be indicated with a suffix, e.g `created_at_cet_ts`

- Aggregates and window-functions should be prefixed by the logic, e.g:

```sql
SELECT
    species_name
    , MIN(discovered_dt) AS first_discovered_on_dt
    , SUM(picked_count) AS sum_quantity
    , MAX(height) AS max_height
    , AVG(diameter) AS avg_diameter

FROM flora

GROUP BY 1
```

### Uniform suffixes

You should hint the intended value type a column holds unless its clear from the name (like `quantity` or `fiscal_year`). Use one of these suffixes:

|hint     | description
|-        | -
`_id`     | Unique identifier such as a column that is a primary key
`_code`   | User friendly and lookup-able representation of something (e.g. `zip_code`, `invoice_status_code`)
`_name`   | Name of an object (typically a descriptive version of a `_code`)
`_text`   | Descriptive values (typically unsuitable for grouping by)
`_number` | Any number not meant for measurement as such (e.g. `row_number` or `supplier_number`)
`_count`  | Something measured individually (e.g. `team_head_count`, `outlet_sold_count`)
`_amount` | Something not measured individually (e.g. `invoice_net_amount`, `power_used_amount`) </br> Currency, if not held in a separate column, should be a prefix after `_amount`, e.g. `invoice_net_amount_usd`
`_size`   | Descriptive measurement (like `file_size` or `shoe_size`)
`_dt`     | Date (`yyyy-mm-dd`)
`_ts`     | Timestamp (`yyyy-mm-dd hh-mm-ss`)
`_fl`     | Boolean value

## Joins

- Your join should list the "left" table first (i.e. the table you are selecting `FROM`)

- `LEFT JOIN` are normally the most useful, `RIGHT JOIN` often indicate that you should change which table you select `FROM` and which one you `JOIN` to.

- When joining, _always_ prefix with the table alias

## Common Table Expressions (CTEs)

> Great code can be advanced or simple but never complex

CTEs are great for encapsulating logic in a SQL query[*](https://discourse.getdbt.com/t/why-the-fishtown-sql-style-guide-uses-so-many-ctes/1091), thus making it more readable for later refactoring or debugging.

- Use CTEs to reference other tables

- CTEs should be placed at the top of the query

- Where performance permits, CTEs should perform a single, logical unit of work

- CTE names should be as verbose as needed to convey what they do

- Prefer CTEs to subqueries [*](https://www.alisa-in.tech/post/2019-10-02-ctes/)

> In general, don't overuse subqueries (AND NEVER NEST!). If the query is going to be read by others or run frequently, try to use a CTE for readability.
>
> Same goes for CTEs, if the query exceeds **100 lines** or has more than, say, **10 CTEs**, you might want to move some of the CTEs to their own model.

- Create a `final` CTE that you select from as your last line of code

- CTEs with confusing or notable logic should be commented

- The following example shows an ideal structure of using CTEs:

```sql
-- Preferred
WITH important_list AS (

    SELECT DISTINCT specific_column

    FROM some_table

    WHERE specific_column != 'foo'
),

with final AS (

    SELECT
        primary_table.column_1
        , primary_table.column_2
    
    FROM primary_table
    INNER JOIN important_list
        ON primary_table.column_3 = important_list.specific_column
)

SELECT * FROM final

-- vs

-- Not Preferred
SELECT
    primary_table.column_1
    , primary_table.column_2

FROM primary_table

WHERE primary_table.column_3 IN (
    SELECT DISTINCT specific_column

    FROM some_table

    WHERE specific_column != 'foo'
)
```

_The general takeaway is to break up your code in logical parts which subsequently makes it very easy to test and debug for yourself or others._ [*](https://discourse.getdbt.com/t/ctes-are-passthroughs-some-research/155)

## Commenting

- Every query should begin outlining the general idea

- Each CTE should include a starting comment of its purpose and grain

## SQLFluff

SQLFLuff is a SQL linter that works with different SQL dialects. We use it to define the basic structure and style of the SQL that we write and move the review of that structure and style into the hands of the authors.

> It requires [Python](https://wiki.python.org/moin/BeginnersGuide/Download) and [pip](https://pip.pypa.io/en/stable/installing/) installed.
>
> For a full walk-through setting up SQLFluff, see [SQLFluff's own instructions](https://docs.sqlfluff.com/en/stable/gettingstarted.html)

SQLFluff includes a fix command that will apply fixes to rule violations when possible. Not all rule violations are automatically fixable; therefore, you are encouraged to run the lint command after using the fix command to ensure that all rule violations have been resolved.

- [SQLFluff Documentation](https://docs.sqlfluff.com/en/latest/index.html)
- [SQLFluff Default configuration](https://docs.sqlfluff.com/en/latest/configuration.html#default-configuration)

Changes from the default configuration

- selecting the dialect to BigQuery
- selecting the templater to be dbt
- selecting the max line length to be 100
- selecting Key words and Functions to always be upper case

The modified configuration file that we use can be found [here]()

![TODO](https://img.shields.io/badge/TO--DO-write_the_SQLFluff_config_file_and_add_link-blue) \
![TODO](https://img.shields.io/badge/TO--DO-expand_on_how_to_actually_change_config_file-blue)

___

Please, never hesitate to ask questions. Things change and best practice matures along with changes.\
You can request specific changes to the guide by making a pull request. Remember to provide the motivation behind it (see [git guide](styleguide_git.md) for help on pull-requests).

Resources used for this guide: \
[GitLab](https://about.gitlab.com/handbook/business-technology/data-team/platform/sql-style-guide/) \
[dbt](https://github.com/dbt-labs/corp/blob/main/dbt_style_guide.md) \
[SQL Style Guide by Simon Holywell](https://www.sqlstyle.guide/) \
[Mozilla](https://docs.telemetry.mozilla.org/concepts/sql_style.html)
