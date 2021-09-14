[![forthebadge](https://forthebadge.com/images/badges/built-with-love.svg)]()
[![forthebadge](images/badges/uses-postgresql.svg)]()
[![forthebadge](https://forthebadge.com/images/badges/made-with-markdown.svg)]()

<p align="center">
    <img src="images\Healthcare_Analytics.png" alt="healthcare-analytics">
</p>

<h1 align="center">Health Analytics Mini Case Study - Serious SQL 🚀</h1>

<div align="center">

  [![Status](https://img.shields.io/badge/status-active-success.svg)]()
  [![Ask Me Anything !](https://img.shields.io/badge/Ask%20me-anything-1abc9c.svg)]() 
  [![Open Source? Yes!](https://badgen.net/badge/Open%20Source%20%3F/Yes%21/blue?icon=github)]()
  [![License](https://img.shields.io/badge/license-MIT-blue.svg)]()

</div>

---

<p align="center"> This is a health analytics mini case study from the <a href="https://www.datawithdanny.com/">Serious SQL</a> course by Danny Ma. The general manager at an healthcare firm has asked us a few questions that they'll like answered.
    <br> 
</p>

## 📝 Table of Contents

- [🧐 About](#about)
- [🚀 Business Questions](business-questions)
- [🎨 Contributing](#contributing)
- [🌟 Support](#support)

## 🧐 About <a name = "about"></a>

We’ve just received an urgent request from the General Manager of Analytics at Health Co requesting our assistance with their analysis of the health.user_logs dataset.

The Health Co analytics team have shared with us a few questions that they want answers to. Let's use SQL to solve the business questions one by one.

## 🚀 Business Questions <a name = "business-questions"></a>

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
  ROUND (AVG(measure_count), 2) AS avg_measurements_per_user
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

### 4.4 How many users have 3 or more measurements?

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

### 4.5 How many users have 1,000 or more measurements?

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

### 4.6 Have logged blood glucose measurements?

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

### 4.7 Have at least 2 types of measurements?

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

### 4.8 Have all 3 measures - blood glucose, weight and blood pressure?

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

### 4.9 What is the median systolic/diastolic blood pressure values?

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

## 🎨 Contributing <a name = "contributing"></a>

Contributions are what make the open source community such an amazing place to learn, inspire, and create. Any contributions you make are **greatly appreciated**.

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the Branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 🌟 Support

Please hit the ⭐button if you like this project. 😄

# Thank you!