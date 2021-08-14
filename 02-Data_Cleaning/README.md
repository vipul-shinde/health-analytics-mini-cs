# 2. Data Preparation and Cleaning

## 2.1 Checking for duplicate values
First, let's take a look at the total count of rows in the dataset.
```sql
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
```sql
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
```sql
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
```sql
DROP TABLE IF EXISTS deduplicated_user_logs;
CREATE TEMP TABLE deduplicated_user_logs AS
SELECT DISTINCT *
FROM health.user_logs;
```

*Output:*
None

Let's take a look at the newly created TEMP table with distinct values only.
```sql
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
```sql
SELECT COUNT(*) 
FROM pg_temp_3.deduplicated_user_logs;
```

*Output:*
| count |
|-------|
| 31004 |

## 2.2 Comparing our output with original data
In our original table, there are **43,891** values whereas in the deduplicated table we got **31,004**.

That means, we definitely have some duplicate values in our dataset.

## 2.3 Identifying duplicate records
Let us take a look at the duplicate row values from dataset.

### 2.3.1 Group by count on all columns
Here, we use a group by clause on all the columns and count as our aggregate function. This will give us all the rows along their count i.e. the number of time they appear in the dataset. Let's limit the output to 10.
```sql
SELECT
  id,
  log_date,
  measure,
  measure_value,
  systolic,
  diastolic,
  COUNT(*) AS frequency
FROM health.user_logs
GROUP BY
  id,
  log_date,
  measure,
  measure_value,
  systolic,
  diastolic
ORDER BY frequency DESC
LIMIT 10;
```

*Output:*

| id                                       | log_date                 | measure       | measure_value | systolic | diastolic | frequency |
|------------------------------------------|--------------------------|---------------|---------------|----------|-----------|-----------|
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2019-12-06T00:00:00.000Z | blood_glucose | 401           | null     | null      | 104       |
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2019-12-05T00:00:00.000Z | blood_glucose | 401           | null     | null      | 77        |
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2019-12-04T00:00:00.000Z | blood_glucose | 401           | null     | null      | 72        |
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2019-12-07T00:00:00.000Z | blood_glucose | 401           | null     | null      | 70        |
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2020-09-30T00:00:00.000Z | blood_glucose | 401           | null     | null      | 39        |
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2020-09-29T00:00:00.000Z | blood_glucose | 401           | null     | null      | 24        |
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2020-10-02T00:00:00.000Z | blood_glucose | 401           | null     | null      | 18        |
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2019-12-10T00:00:00.000Z | blood_glucose | 140           | null     | null      | 12        |
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2019-12-11T00:00:00.000Z | blood_glucose | 220           | null     | null      | 12        |
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2020-04-15T00:00:00.000Z | blood_glucose | 236           | null     | null      | 12        |


### 2.3.2 Having Clause for Unique Duplicates
```sql
DROP TABLE IF EXISTS duplicated_record_count;

CREATE TEMPORARY TABLE duplicated_record_count AS (
SELECT
  id,
  measure,
  measure_value,
  systolic,
  diastolic
FROM health.user_logs
GROUP BY
  id,
  measure,
  measure_value,
  systolic,
  diastolic
HAVING COUNT(*) > 1
);

SELECT * 
FROM duplicated_record_count
LIMIT 10;
```

*Output:*

| id                                       | measure        | measure_value | systolic | diastolic |
|------------------------------------------|----------------|---------------|----------|-----------|
| 3bb42419c1d4a801a01ec9eb8cecae2806d450eb | blood_glucose  | 139           | 0        | 0         |
| ac240708637cc4ea3781e878e4bcdd827b9e5292 | blood_glucose  | 129           | null     | null      |
| 46d921f1111a1d1ad5dd6eb6e4d0533ab61907c9 | blood_glucose  | 192           | null     | null      |
| cc85b6fd19008f32a82562efed00cf22918d7bd4 | blood_glucose  | 105           | 0        | 0         |
| d696925de5e9297694ef32a1c9871f3629bec7e5 | blood_glucose  | 238           | 0        | 0         |
| 3dcab94d65fa303807557df09cf566d809e0b15f | blood_glucose  | 76            | null     | null      |
| 0efe1f378aec122877e5f24f204ea70709b1f5f8 | blood_glucose  | 130           | 0        | 0         |
| 24eb992fbddc32223595d1ad8f927002292063b0 | blood_glucose  | 138           | 0        | 0         |
| 0f7b13f3f0512e6546b8d2c0d56e564a2408536a | blood_pressure | 132           | 132      | 79        |
| ee653a96022cc3878e76d196b1667d95beca2db6 | blood_pressure | 111           | 111      | 67        |

### 2.3.3 Getting all the duplicate counts
```sql
WITH groupby_counts AS (
  SELECT
    id,
    log_date,
    measure,
    measure_value,
    systolic,
    diastolic,
    COUNT(*) AS frequency
  FROM health.user_logs
  GROUP BY
    id,
    log_date,
    measure,
    measure_value,
    systolic,
    diastolic
)
SELECT *
FROM groupby_counts
WHERE frequency > 1
ORDER BY frequency DESC
LIMIT 10;
```

*Output:*

| id                                       | log_date                 | measure       | measure_value | systolic | diastolic | frequency |
|------------------------------------------|--------------------------|---------------|---------------|----------|-----------|-----------|
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2019-12-06T00:00:00.000Z | blood_glucose | 401           | null     | null      | 104       |
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2019-12-05T00:00:00.000Z | blood_glucose | 401           | null     | null      | 77        |
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2019-12-04T00:00:00.000Z | blood_glucose | 401           | null     | null      | 72        |
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2019-12-07T00:00:00.000Z | blood_glucose | 401           | null     | null      | 70        |
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2020-09-30T00:00:00.000Z | blood_glucose | 401           | null     | null      | 39        |
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2020-09-29T00:00:00.000Z | blood_glucose | 401           | null     | null      | 24        |
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2020-10-02T00:00:00.000Z | blood_glucose | 401           | null     | null      | 18        |
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2019-12-10T00:00:00.000Z | blood_glucose | 140           | null     | null      | 12        |
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2019-12-11T00:00:00.000Z | blood_glucose | 220           | null     | null      | 12        |
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2020-04-15T00:00:00.000Z | blood_glucose | 236           | null     | null      | 12        |

## 2.4 Ignoring Duplicate Records

In this case study, the log date doesn't contain any timestamps. So, it could be that the patient logged on to the online portal and submitted their health measurements at different times throughout a day. Hence, we may just ignore the duplicate values here.

