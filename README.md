# Health Analytics Mini Case Study - Serious SQL

## 1. Exploratory Data Analysis

### 1.1 A look at the dataset
Let's take a look at the first 10 rows from the `health.user_logs` table.
```
SELECT *
FROM health.user_logs
LIMIT 10;
```
*Output:*
| id                                       | log_date                 | measure        | measure_value | systolic | diastolic |
|------------------------------------------|--------------------------|----------------|---------------|----------|-----------|
| fa28f948a740320ad56b81a24744c8b81df119fa | 2020-11-15T00:00:00.000Z | weight         | 46.03959      | null     | null      |
| 1a7366eef15512d8f38133e7ce9778bce5b4a21e | 2020-10-10T00:00:00.000Z | blood_glucose  | 97            | 0        | 0         |
| bd7eece38fb4ec71b3282d60080d296c4cf6ad5e | 2020-10-18T00:00:00.000Z | blood_glucose  | 120           | 0        | 0         |
| 0f7b13f3f0512e6546b8d2c0d56e564a2408536a | 2020-10-17T00:00:00.000Z | blood_glucose  | 232           | 0        | 0         |
| d14df0c8c1a5f172476b2a1b1f53cf23c6992027 | 2020-10-15T00:00:00.000Z | blood_pressure | 140           | 140      | 113       |
| 0f7b13f3f0512e6546b8d2c0d56e564a2408536a | 2020-10-21T00:00:00.000Z | blood_glucose  | 166           | 0        | 0         |
| 0f7b13f3f0512e6546b8d2c0d56e564a2408536a | 2020-10-22T00:00:00.000Z | blood_glucose  | 142           | 0        | 0         |
| 87be2f14a5550389cb2cba03b3329c54c993f7d2 | 2020-10-12T00:00:00.000Z | weight         | 129.060012817 | 0        | 0         |
| 0efe1f378aec122877e5f24f204ea70709b1f5f8 | 2020-10-07T00:00:00.000Z | blood_glucose  | 138           | 0        | 0         |
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2020-10-04T00:00:00.000Z | blood_glucose  | 210           | null     | null      |

### 1.2 Total record count
Let's also take a look at the total record count.
```
SELECT 
  COUNT(*)
FROM health.user_logs;
```
*Output:*
| count |
|-------|
| 43891 |

### 1.3 Unique column count
We'll take a look at how many unique id's are present in the dataset. That'll give us a count of the total number of users.
```
SELECT COUNT(DISTINCT id)
FROM health.user_logs;
```
*Output:*
| count |
|-------|
| 554   |

### 1.4 Single column frequency counts
Let's take a look at the measure column and see frequency and the percentage count of each value across the table.
```
SELECT 
  measure,
  COUNT(*) AS frequency,
  ROUND(
  100* COUNT(*)::NUMERIC/SUM(COUNT(*)) OVER(),
  2) AS percentage
FROM health.user_logs
GROUP BY measure
ORDER BY frequency DESC;
```
*Output:*
| measure        | frequency | percentage |
|----------------|-----------|------------|
| blood_glucose  | 38692     | 88.15      |
| weight         | 2782      | 6.34       |
| blood_pressure | 2417      | 5.51       |

Let's also see the frequency of unique id's that appear in the dataset and limit the output to just 5.
```
SELECT 
  id,
  COUNT(*) AS frequency,
  ROUND(
  100* COUNT(*)::NUMERIC/SUM(COUNT(*)) OVER(),
  2) AS percentage
FROM health.user_logs
GROUP BY id
ORDER BY frequency DESC
LIMIT 5;
```
*Output:*
| id                                       | frequency | percentage |
|------------------------------------------|-----------|------------|
| 054250c692e07a9fa9e62e345231df4b54ff435d | 22325     | 50.86      |
| 0f7b13f3f0512e6546b8d2c0d56e564a2408536a | 1589      | 3.62       |
| ee653a96022cc3878e76d196b1667d95beca2db6 | 1235      | 2.81       |
| abc634a555bbba7d6d6584171fdfa206ebf6c9a0 | 1212      | 2.76       |
| 576fdb528e5004f733912fae3020e7d322dbc31a | 1018      | 2.32       |

### 1.5 Individual column distribution
Let's now take a look at the most frequent values accross each column.

1. Measure Value Column
```
SELECT 
  measure_value,
  COUNT(*) AS frequency
FROM health.user_logs
GROUP BY measure_value
ORDER BY frequency DESC
LIMIT 10;
```
*Output:*
| measure_value | frequency |
|---------------|-----------|
| 0             | 572       |
| 401           | 433       |
| 117           | 390       |
| 118           | 346       |
| 123           | 342       |
| 122           | 331       |
| 126           | 326       |
| 120           | 323       |
| 128           | 319       |
| 115           | 319       |

2. Systolic column
```
SELECT 
  systolic,
  COUNT(*) AS frequency
FROM health.user_logs
GROUP BY systolic
ORDER BY frequency DESC
LIMIT 10;
```
*Output:*
| systolic | frequency |
|----------|-----------|
| null     | 26023     |
| 0        | 15451     |
| 120      | 71        |
| 123      | 70        |
| 128      | 66        |
| 127      | 64        |
| 130      | 60        |
| 119      | 60        |
| 135      | 57        |
| 124      | 55        |

Wow. So many null and zero values! We'll come back to this later.

3. Diastolic column
```
SELECT 
  diastolic,
  COUNT(*) AS frequency
FROM health.user_logs
GROUP BY diastolic
ORDER BY frequency DESC
LIMIT 10;
```
*Output:*
| diastolic | frequency |
|-----------|-----------|
| null      | 26023     |
| 0         | 15449     |
| 80        | 156       |
| 79        | 124       |
| 81        | 119       |
| 78        | 110       |
| 77        | 109       |
| 73        | 109       |
| 83        | 106       |
| 76        | 102       |



