# Sales Forecasting Comparison Tool

## Overview
This notebook demonstrates a forecasting approach that:
1. Loads historical monthly sales data for multiple products from an Excel file.
2. Splits the data into **training (75%)** and **validation (25%)** sets.
3. Tests **three forecasting methods**:
    - **Moving Average** (3-month rolling mean)
    - **Linear Regression** (trend line)
    - **IQR-based anomaly removal + Moving Average** (removes outliers before forecasting)
4. Compares methods based on Mean Absolute Percentage Error (MAPE) for the validation period.
5. Selects the best-performing method for each product.
6. Forecasts sales for **Jan–Mar 2026** using the selected method.

# understanding the Forecasting Methods

Forecasting is an essential part of data analysis, especially when trying to predict future trends from historical patterns. In this context, we are using three different methods—Moving Average, Linear Regression, and Quantile Filtered Forecast—to predict sales. Each of these approaches has a distinct way of processing the historical data to estimate what comes next.

<b>1. Moving Average Forecast</b><br />
The Moving Average method is one of the simplest forecasting techniques. The idea is to take the average of a fixed number of past observations (known as the window size) and use this average as the forecast for the next period.
```
def moving_average_forecast(train, window=3):
    return train.rolling(window=window).mean().shift(1)
```
* train.rolling(window=window).mean() calculates the average of the last window data points.
* .shift(1) ensures that the forecast is based on past values only (avoiding "peeking" into the future).

<b>2. Linear Regression Forecast</b><br />
Linear Regression is a statistical modeling method that assumes a straight-line relationship between time (or any independent variable) and the target variable (sales in this case). The model fits a line that minimizes the difference between predicted and actual values.
```
def linear_regression_forecast(train_index, train_values, test_index):
    model = LinearRegression()
    model.fit(train_index.reshape(-1, 1), train_values)
    return model.predict(test_index.reshape(-1, 1))
```
* The train_index represents time steps (e.g., months as numbers 1, 2, 3…).
* train_values are the actual historical sales values.
* The model is trained on the historical data and predicts sales for future time steps (test_index).

<b>1. Quantile Filtered Forecast</b><br />
The Quantile Filtered Forecast is an enhancement over standard Linear Regression by first removing outliers before fitting the model. This ensures that extreme, abnormal values do not distort the forecast.
```
# Calculate Interquartile Range (IQR)
Q1 = np.percentile(train_values, 25)
Q3 = np.percentile(train_values, 75)
IQR = Q3 - Q1

# Define bounds for outlier detection
lower_bound = Q1 - 1.5 * IQR
upper_bound = Q3 + 1.5 * IQR

# Filter out anomalies
mask = (train_values >= lower_bound) & (train_values <= upper_bound)
```
* IQR Method
  * Q1 (25th percentile): Value below which 25% of data lies.
  * Q3 (75th percentile): Value below which 75% of data lies.
  * IQR = Q3 - Q1: Measures the middle spread of data.
  * Data points outside the range [Q1 - 1.5*IQR, Q3 + 1.5*IQR] are considered outliers.
* Filter
  * The model is trained only on the data within these bounds.
* Forecast Method
  * A Linear Regression model is fitted to this cleaned dataset and used to make predictions.

