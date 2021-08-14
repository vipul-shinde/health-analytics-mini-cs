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