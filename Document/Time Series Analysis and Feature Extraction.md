
## Components of Time Series
- **Trend:** shows a general direction of the time series data over a long period of time
- **Seasonality:** component exhibits a trend that repeats with respect to timing, direction, and magnitude.
- **Cyclical:** the period of ups and downs, booms and slums of a time series, mostly observed in business cycles.
- **Irregular Variation:** The fluctuations in the time series data which become evident when trend and cyclical variations are removed.
- **ETS Decomposition:** ETS Decomposition is used to separate different components of a time series. The term ETS stands for Error, Trend and Seasonality.

## Time Series Stationary
Time Series with Stationary has it statistical properties (mean, variance, auto-correlation) do not depend on the time, remains constant over the time period. Stationary time series will have no predictable patterns in the long term
#### Make the time series stationary
- Differencing the Series
- Take the log of Series
- Take the $n^{th}$ root of the Series 
#### Testing for stationary
##### 1. Augmented Dickey Fuller Test
##### 2. Kwiatkowski-Phillips-Schmidt-Shin – KPSS test (trend stationary)
##### 3. Phillips Perron Test (PP Test)

## Differencing
Differencing is subtracting the next value by the current value: 
$Y = Y_t - Y_{t-1}$
Differencing is used to make the series stationary and to control the auto-correlations.
Transformations such as logarithms can help to stabilise the variance of a time series. Differencing can help stabilise the mean of a time series by removing changes in the level of a time series, and therefore eliminating (or reducing) trend and seasonality.

## Additive and Multiplication Time Series
Additive: 
Value = Base Level + Trend + Seasonality + Error
Multiplication:
Value = Base Level x Trend x Seasonality x Error

``` python
from statsmodels.tsa.seasonal import seasonal_decompose
from statsmodels.tsa.stattools import adfuller

from numpy import log
import numpy as np 


result = adfuller(df_by_minute.total_amount.dropna())
print('ADF Statistic: %f' % result[0])
print('p-value: %f' % result[1])

plt.rcParams.update({'figure.figsize':(16,5), 'figure.dpi':120})

fig, axes = plt.subplots(1, 2, sharex=True)
axes[0].plot(df_by_minute.total_amount.diff()); axes[0].set_title('1st Differencing')
axes[1].set(ylim=(0,5))
plot_pacf(df_by_minute.total_amount.diff().dropna(), ax=axes[1])

plt.show()
```

