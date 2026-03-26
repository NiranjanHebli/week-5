## Q1: What is EDA?

EDA (Exploratory Data Analysis) is the process of examining and summarizing a dataset to understand its main characteristics, identify patterns, detect anomalies, and prepare it for modeling or further analysis.

## Why is EDA important?
EDA is important because it helps:

- Understand the structure and types of data
- Detect patterns, correlations, and trends
- Identify outliers and errors
- Guide feature engineering and model selection
- Reduce risks of incorrect assumptions or errors in analysis


## Q2 - Coding 

```python

import pandas as pd

data = {
    'Name': ['Alice', 'Bob', 'Charlie', 'David', 'Eva'],
    'Score': [85, 70, 90, 60, 95]
}

df = pd.DataFrame(data)

average_score = df['Score'].mean()
print("Average Score:", average_score)

filtered_df = df[df['Score'] > average_score]

print("\n Rows with Score greater than average:")
print(filtered_df)

```

## File:-   
#### [`Filter By Avg`](filter_by_avg.ipynb)


## Q3 - What insights can we get from describe()?

The describe() function provides summary statistics of numerical columns in a DataFrame. It helps quickly understand the distribution and spread of data.

The describe() function gives the following insights:

- `Count:` Number of non-missing values, which helps identify missing data.
- `Mean:` Average value, showing the central tendency.
- `Standard Deviation (std):` Measures the spread of data, showing how much values vary from the mean.
- `Minimum (min) and Maximum (max):` Shows the range of data and potential extremes.
- `25%, 50%, 75% (quartiles):` Shows how data is distributed and helps detect skewness.
- `Outlier detection:` Very high or low min/max values compared to quartiles may indicate outliers.