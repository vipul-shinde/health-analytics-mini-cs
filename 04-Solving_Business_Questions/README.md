# 4. Solving Business Questions

We’ve just received an urgent request from the General Manager of Analytics at Health Co requesting our assistance with their analysis of the health.user_logs dataset.

The Health Co analytics team have shared with us a few questions that they want answers to. Let's use SQL to solve the business questions one by one.

### 4.1 How many unique users exist in the logs dataset?

```sql
SELECT 
  COUNT(DISTINCT id) AS unique_count
FROM health.user_logs;
```

*Output:*

| unique_count |
|--------------|
| 554          |

So in total, there are 554 unique users in the dataset.

*For questions 2-8 we have created a temp table*
```sql
DROP TABLE IF EXISTS user_measure_count;
CREATE TEMP TABLE user_measure_count AS
SELECT
  id,
  COUNT(*) AS measure_count,
  COUNT(DISTINCT measure) as unique_measures
FROM health.user_logs
GROUP BY 1;
```

### 4.2 How many total measurements do we have per user on average?

```sql
SELECT
  ROUND (
  AVG(measure_count),
  2) AS avg_measurements_per_user
FROM user_measure_count;
```

*Output:*

| avg_measurements_per_user |
|---------------------------|
| 79.23                     |

The average number of measurements per user is 79.

### 4.3 What about the median number of measurements per user?

```sql
SELECT
  ROUND (
   CAST(PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY measure_count) AS NUMERIC) ,
   2
   ) AS median_measurements_per_user
FROM user_measure_count;
```

*Output:*

| median_measurements_per_user |
|------------------------------|
| 2.00                         |

