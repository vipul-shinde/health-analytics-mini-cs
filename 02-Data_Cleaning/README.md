## 2. Data Preparation and Cleaning

### 2.1 Checking for duplicate values
First, let's take a look at the total count of rows in the dataset.
```
SELECT 
  COUNT(*)
FROM health.user_logs;
```

*Output:*
| count |
|-------|
| 43891 |

Now, let's check for distinct number of values in the dataset. There are three different ways we could do that:

### 2.1.1 Subquery
A subquery is essentially a query within a query. Here we get an output from querying on the inner query. 
```
SELECT COUNT(*)
FROM (
  SELECT DISTINCT *
  FROM health.user_logs
) AS subquery
;
```

*Output:*
| count |
|-------|
| 31004 |

### 2.1.2 Common Table Expression
```
WITH deduped_logs AS (
  SELECT DISTINCT *
  FROM health.user_logs
)
SELECT COUNT(*)
FROM deduped_logs;
```

*Output:*
| count |
|-------|
| 31004 |

### 2.1.3 Temporary Tables
A temporary table will help you minimize the number of time you'll have to use the DISTINCT command everytime you want to query from the table without any duplicate values.

But, before creating a temp table, you should run a drop table query to clear any previously created tables. This helps us create a clean table and if there's any table with the same name, it'll clear it.

Now, let's create the temporary table.
```
DROP TABLE IF EXISTS deduplicated_user_logs;
CREATE TEMP TABLE deduplicated_user_logs AS
SELECT DISTINCT *
FROM health.user_logs;
```

*Output:*
None

Let's take a look at the newly created TEMP table with distinct values only.
```
SELECT * 
FROM pg_temp_3.deduplicated_user_logs
LIMIT 5;
```

*Output:*
| id                                       | log_date                 | measure        | measure_value | systolic | diastolic |
|------------------------------------------|--------------------------|----------------|---------------|----------|-----------|
| 576fdb528e5004f733912fae3020e7d322dbc31a | 2019-12-15T00:00:00.000Z | blood_pressure | 0             | 124      | 72        |
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2020-04-15T00:00:00.000Z | blood_glucose  | 267           | null     | null      |
| 8b130a2836a80239b4d1e3677302709cea70a911 | 2019-12-31T00:00:00.000Z | blood_glucose  | 109.799995    | null     | null      |
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2020-05-07T00:00:00.000Z | blood_glucose  | 189           | null     | null      |
| 0f7b13f3f0512e6546b8d2c0d56e564a2408536a | 2020-08-18T00:00:00.000Z | weight         | 68.49244787   | 0        | 0         |

The total count of values in the table:
```
SELECT COUNT(*) 
FROM pg_temp_3.deduplicated_user_logs;
```

*Output:*
| count |
|-------|
| 31004 |

## 2.2 Comparing our output with original data