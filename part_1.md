# Part 1: Data Modeling and SQL

## 1. Modeling

View: `resolved_loans_summary`
- Converts some string field into useful datetime and int fields
- Creates a binary column `is_default` from `loan_status`. This set's the source of truth for what a default is defined as and ensures that definition is applied consistently across all downstream calculations. 
- No repeated filtering logic, no ambiguity around status definitions, and no date parsing scattered across different queries. 
- Provides some other useful columns, allowing for some high level investigations at this level before having to query the raw data.

| Column        | Type | Description                                                 |
|---------------|---|-------------------------------------------------------------|
| `id`          | STRING | Unique loan identifier                                      |
| `issue_d_dt`  | DATE | Loan issue date, parsed from the raw string field `issue_d` |
| `grade`       | STRING | Loan grade assigned by the lender (A–G)                     |
| `sub_grade`   | STRING | More granular grade subdivision                             |
| `term_int`    | INT | Loan term in months, parsed from string field `term`        |
| `loan_amnt`   | FLOAT | Original loan amount requested                              |
| `int_rate`    | FLOAT | Interest rate on the loan                                   |
| `is_default`  | INT | 1 if `loan_status` is "Charged Off"/"Default"               |


---

## 2. SQL

### Default Rate per Grade

```sql
SELECT
    grade,
    COUNT(*)                                            AS total_loans,
    COUNTIF(is_default = 1)                             AS total_defaults,
    ROUND(COUNTIF(is_default = 1) / COUNT(*) * 100, 2) AS default_rate
FROM
    resolved_loans_summary
GROUP BY
    grade
ORDER BY
    grade
```

---

### Bonus: Default Rate per Grade Over Time

To track how default rates for each grade have trended over time, we can add `issue_d_dt` to the grouping.

```sql
SELECT
    grade,
    DATE_TRUNC(issue_d_dt, MONTH)                       AS issue_month,
    COUNT(*)                                            AS total_loans,
    COUNTIF(is_default = 1)                             AS total_defaults,
    ROUND(COUNTIF(is_default = 1) / COUNT(*) * 100, 2) AS default_rate
FROM
    resolved_loans_summary
GROUP BY
    grade,
    issue_month
ORDER BY
    grade,
    issue_month
```