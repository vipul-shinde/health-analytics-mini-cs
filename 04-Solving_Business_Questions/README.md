# 4. Solving Business Questions

We’ve just received an urgent request from the General Manager of Analytics at Health Co requesting our assistance with their analysis of the health.user_logs dataset.

The Health Co analytics team have shared with us a few questions that they want answers to. Let's use SQL to solve the business questions one by one.

## 4.1 How many unique users exist in the logs dataset?

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

## 4.2 How many total measurements do we have per user on average?

```sql
SELECT
  ROUND (AVG(measure_count), 2) AS avg_measurements_per_user
FROM user_measure_count;
```

*Output:*

| avg_measurements_per_user |
|---------------------------|
| 79.23                     |

The average number of measurements per user is 79.

## 4.3 What about the median number of measurements per user?

```sql
SELECT
  ROUND (
   CAST(PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY measure_count) AS NUMERIC),
   2
   ) AS median_measurements_per_user
FROM user_measure_count;
```

*Output:*

| median_measurements_per_user |
|------------------------------|
| 2.00                         |

The median number of measurements per user is 2.

## 4.4 How many users have 3 or more measurements?

```sql
SELECT
  COUNT(*) AS total
FROM user_measure_count
WHERE measure_count >= 3;
```

*Output:*

| total |
|-------|
| 209   |

Total number of users having 3 or more measurements is 209.

## 4.5 How many users have 1,000 or more measurements?

```sql
SELECT
  COUNT(*) AS total
FROM user_measure_count
WHERE measure_count >= 1000;
```

*Output:*

| total |
|-------|
| 5     |

Total number of users having 1000 or more measurements is 5.

## 4.6 Have logged blood glucose measurements?

```sql
SELECT 
  COUNT (DISTINCT id)
FROM health.user_logs
WHERE measure = 'blood_glucose';
```

*Output:*

| count |
|-------|
| 325   |

The number of users having blood_glucose as a logged measurement is 325.

## 4.7 Have at least 2 types of measurements?

```sql
SELECT 
  SUM(COUNT (*)) OVER() AS total_count
FROM user_measure_count
WHERE unique_measures >= 2;
```

*Output:*

| total_count |
|-------------|
| 204         |

There are 204 users having at least 2 types of measurements.

## 4.8 Have all 3 measures - blood glucose, weight and blood pressure?

```sql
SELECT 
  SUM(COUNT (*)) OVER() AS total_count
FROM user_measure_count
WHERE unique_measures = 3;
```

*Output:*

| total_count |
|-------------|
| 50          |

Total numbers of users having all 3 measures as their logged measurement is 50.

## 4.9 What is the median systolic/diastolic blood pressure values?

```sql
SELECT 
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY systolic) AS systolic_median,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY diastolic) AS diastolic_median
FROM health.user_logs
WHERE measure = 'blood_pressure';
```

*Output:*

| systolic_median | diastolic_median |
|-----------------|------------------|
| 126             | 79               |

The median systolic/diastolic blood pressure value is 126/79.

# Thank you!