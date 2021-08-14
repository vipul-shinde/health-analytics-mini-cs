# 3. Statistical Analysis and Distribution Functions

## 3.1 Arithmetic Mean or Average
Let's take a look at the average of the ```median_value``` column.
```sql
SELECT
  AVG(measure_value)
FROM health.user_logs;
```

*Output:*

| avg                   |
|-----------------------|
| 1986.2288605267024675 |


Whoa! That's a huge number. Let's go further into this. Let's see our measures again.
```sql
SELECT
  measure,
  COUNT(*) AS counts
FROM health.user_logs
GROUP BY measure;
```

*Output:*

| measure        | counts |
|----------------|--------|
| blood_glucose  | 38692  |
| blood_pressure | 2417   |
| weight         | 2782   |

Also, let's see the average measure_value based grouped by the measure column.
```sql
SELECT
  measure,
  AVG(measure_value),
  COUNT(*) AS counts
FROM health.user_logs
GROUP BY measure
ORDER BY counts;
```

*Output:*

| measure        | avg                  | counts |
|----------------|----------------------|--------|
| blood_pressure | 95.4040815126905254  | 2417   |
| weight         | 28786.846657295953   | 2782   |
| blood_glucose  | 177.3485953624517730 | 38692  |

Something is wrong here. The weight average of a person is ```28786```. Let's comeback later to this.

## 3.2 Median & Mode

Let's also calculate the median & mode for the weight column and see why is this happening.
```sql
SELECT
  AVG(measure_value) as mean,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY measure_value) AS median,
  MODE() WITHIN GROUP (ORDER BY measure_value) AS mode
FROM health.user_logs
WHERE measure = 'weight';
```

*Output:*

| mean               | median       | mode        |
|--------------------|--------------|-------------|
| 28786.846657295953 | 75.976721975 | 68.49244787 |

Looks like there are outliers in the dataset within the measure_value column when ```measure='weights'```.

## 3.3 Spread of the Data

Let's see how the data is spread across the measure_value column and the measure is equal to weight. 

### 3.3.1 Min, Max, Range values

```sql
WITH min_max_values AS (
  SELECT
    MIN(measure_value) AS min_value,
    MAX(measure_value) AS max_value
  FROM health.user_logs
  WHERE measure = 'weight'
)

SELECT
  min_value,
  max_value,
  max_value - min_value AS range_value
FROM min_max_values;
```

*Output:*

| min_value | max_value | range_value |
|-----------|-----------|-------------|
| 0         | 39642120  | 39642120    |

### 3.3.2 Variance & Standard Deviation

The variance and standard deviation will give us a more clear idea about the spread of data as compared to just min, max values.

```sql
SELECT
  VARIANCE(measure_value) AS var_value,
  STDDEV(measure_value) AS std_value
FROM health.user_logs
WHERE measure = 'weight';
```

*Output:*

| var_value                        | std_value                  |
|----------------------------------|----------------------------|
| 1129457862383.414293789530531301 | 1062759.550596189323085400 |

## 3.4 Statistical Summary

Let's query all the stats values again but this time let's round of the data by 2 decimals.

```sql
SELECT
  'weight' as measure,
  ROUND(MIN(measure_value), 2) AS min_value,
  ROUND(MAX(measure_value), 2) AS max_value,
  ROUND(AVG(measure_value), 2) AS mean_value,
  ROUND(
    CAST(PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY measure_value) AS NUMERIC),
    2
  ) AS median_value,
  ROUND(
    MODE() WITHIN GROUP (ORDER BY measure_value),
    2
  ) AS mode_value,
  ROUND(VARIANCE(measure_value), 2) AS var_value,
  ROUND(STDDEV(measure_value), 2) AS std_value
FROM health.user_logs
WHERE measure = 'weight';
```

*Output:*

| measure | min_value | max_value   | mean_value | median_value | mode_value | var_value        | std_value  |
|---------|-----------|-------------|------------|--------------|------------|------------------|------------|
| weight  | 0.00      | 39642120.00 | 28786.85   | 75.98        | 68.49      | 1129457862383.41 | 1062759.55 |



