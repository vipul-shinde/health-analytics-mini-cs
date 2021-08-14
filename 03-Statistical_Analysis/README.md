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

## 3.5 Cumulative Distribution Function
CDF takes up a value and returns the percentile in which the value belongs to. Let's query the CDF of all values in measure_value when ```measure='weight```. We'll limit the output to 10 rows.
```sql
SELECT
  measure_value,
  NTILE(100) OVER (ORDER BY measure_value) AS percentile
FROM health.user_logs
WHERE measure = 'weight'
ORDER BY percentile
LIMIT 10;
```

*Output:*

| measure_value | percentile |
|---------------|------------|
| 0             | 1          |
| 1.814368      | 1          |
| 2.26796       | 1          |
| 2.26796       | 1          |
| 8             | 1          |
| 10.432616     | 1          |
| 11.3398       | 1          |
| 12.700576     | 1          |
| 15.422128     | 1          |
| 0             | 1          |

Now, we can find out the floor and ceiling value (i.e. max and min value) within each bucket or percentile and also the total number of values present in that particular bucket. Ideally, since we are calculating 100 buckets each bucket should contain 1% of the total data.
```sql
WITH percentile_values AS (
  SELECT
    measure_value,
    NTILE(100) OVER (ORDER BY measure_value) AS percentile
  FROM health.user_logs
  WHERE measure = 'weight'
  ORDER BY percentile
)

SELECT
  percentile,
  MIN(measure_value) AS floor_value,
  MAX(measure_value) AS ceiling_value,
  COUNT(*) AS percentile_count
FROM percentile_values
GROUP BY percentile
ORDER BY percentile;
```

*Output:*

| percentile | floor_value   | ceiling_value | percentile_count |
|------------|---------------|---------------|------------------|
| 1          | 0             | 29.029888     | 28               |
| 2          | 29.48348      | 32.0689544    | 28               |
| 3          | 32.205032     | 35.380177     | 28               |
| 4          | 35.380177     | 36.74095      | 28               |
| 5          | 36.74095      | 37.194546     | 28               |
| ...        |  ...          |   ...         |    ...           |
| 95         | 129.86485     | 130.542007446 | 27               |
| 96         | 130.54207     | 131.570999146 | 27               |
| 97         | 131.670013428 | 132.776       | 27               |
| 98         | 132.776000977 | 133.832000732 | 27               |
| 99         | 133.89095     | 136.531192    | 27               |
| 100        | 136.531192    | 39642120      | 27               |

If we look at the above output. Let's just take a look at 1st and 100th percentile.

| percentile | floor_value   | ceiling_value | percentile_count |
|------------|---------------|---------------|------------------|
| 1          | 0             | 29.029888     | 28               |
| 100        | 136.531192    | 39642120      | 27               |

The ceiling_value of 1st percentile is 29.02 i.e. 29kg maybe, and the floor_value of 100th percentile is 136kg but ceiling is 3964210 kg?? Sounds abnormal. Maybe there was an incorrect measurement input fro few of the patient logs from the 100th %tile. Let's dive further into the 100th %tile to investigate more on this.

