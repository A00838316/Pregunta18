# Pregunta18

import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from statsmodels.tsa.stattools import adfuller
from statsmodels.graphics.tsaplots import plot_acf, plot_pacf
from statsmodels.tsa.arima.model import ARIMA
import warnings
warnings.filterwarnings("ignore")

# Step 1: Data Preparation
data = {
    'Year': ['1970.1', '1970.2', '1970.3', '1970.4', '1971.1', '1971.2', '1971.3', '1971.4', 
             '1972.1', '1972.2', '1972.3', '1972.4', '1973.1', '1973.2', '1973.3', '1973.4', 
             '1974.1', '1974.2', '1974.3', '1974.4', '1975.1', '1975.2', '1975.3', '1975.4', 
             '1976.1', '1976.2', '1976.3', '1976.4', '1977.1', '1977.2', '1977.3', '1977.4', 
             '1978.1', '1978.2', '1978.3', '1978.4', '1979.1', '1979.2', '1979.3', '1979.4', 
             '1980.1', '1980.2', '1980.3', '1980.4', '1981.1', '1981.2', '1981.3', '1981.4', 
             '1982.1', '1982.2', '1982.3', '1982.4', '1983.1', '1983.2', '1983.3', '1983.4', 
             '1984.1', '1984.2', '1984.3', '1984.4', '1985.1', '1985.2', '1985.3', '1985.4', 
             '1986.1', '1986.2', '1986.3', '1986.4', '1987.1', '1987.2', '1987.3', '1987.4', 
             '1988.1', '1988.2', '1988.3', '1988.4', '1989.1', '1989.2', '1989.3', '1989.4', 
             '1990.1', '1990.2', '1990.3', '1990.4', '1991.1', '1991.2', '1991.3', '1991.4'],
    'PDI': [1990.6, 2020.1, 2045.3, 2045.2, 2073.9, 2098.0, 2106.6, 2121.1, 2129.7, 2149.1, 
            2193.9, 2272.0, 2300.7, 2315.2, 2337.9, 2382.7, 2334.7, 2304.5, 2315.0, 2313.7, 
            2282.5, 2390.3, 2354.4, 2389.4, 2424.5, 2434.9, 2444.7, 2459.5, 2463.0, 2490.3, 
            2541.0, 2556.2, 2587.3, 2631.9, 2653.2, 2680.9, 2699.2, 2697.6, 2715.3, 2728.1, 
            2742.9, 2692.0, 2722.5, 2777.0, 2783.7, 2776.7, 2814.1, 2808.8, 2795.0, 2824.8, 
            2829.0, 2832.6, 2843.6, 2867.0, 2903.0, 2960.6, 3033.2, 3065.9, 3102.7, 3118.5, 
            3123.6, 3189.6, 3156.5, 3178.7, 3227.5, 3281.4, 3272.6, 3266.2, 3295.2, 3241.7, 
            3285.7, 3335.8, 3380.1, 3386.3, 3407.5, 3443.1, 3473.9, 3450.9, 3466.9, 3493.0, 
            3531.4, 3545.3, 3547.0, 3529.5, 3514.8, 3537.4, 3539.9, 3547.5]
}
df = pd.DataFrame(data)
df['Year'] = pd.period_range(start='1970Q1', periods=len(df), freq='Q')
df.set_index('Year', inplace=True)

# Compute log DPI and its first difference
df['Log_DPI'] = np.log(df['PDI'])
df['Log_DPI_diff'] = df['Log_DPI'].diff().dropna()

# Step 2: Visualize the Data
plt.figure(figsize=(10, 6))
plt.plot(df.index.to_timestamp(), df['Log_DPI'], label='Log DPI')
plt.title('Log Personal Disposable Income (1970.1 - 1991.4)')
plt.xlabel('Year')
plt.ylabel('Log DPI')
plt.legend()
plt.show()

# Step 3: Check Stationarity with ADF Test
def adf_test(series, title=''):
    result = adfuller(series.dropna())
    print(f'ADF Test on {title}')
    print(f'ADF Statistic: {result[0]}')
    print(f'p-value: {result[1]}')
    print('Critical Values:')
    for key, value in result[4].items():
        print(f'   {key}: {value}')
    print('')

adf_test(df['Log_DPI'], 'Log DPI')
adf_test(df['Log_DPI_diff'], 'First Differenced Log DPI')

# Step 4: ACF and PACF Plots for Differenced Series
plt.figure(figsize=(12, 6))
plt.subplot(121)
plot_acf(df['Log_DPI_diff'].dropna(), ax=plt.gca(), lags=20)
plt.title('ACF of Differenced Log DPI')
plt.subplot(122)
plot_pacf(df['Log_DPI_diff'].dropna(), ax=plt.gca(), lags=20)
plt.title('PACF of Differenced Log DPI')
plt.show()

# Step 5a: Fit ARIMA(1,1,1) Model to Log DPI
arima_model = ARIMA(df['Log_DPI'], order=(1, 1, 1))
arima_results = arima_model.fit()
print("ARIMA(1,1,1) Model Summary:")
print(arima_results.summary())

# Step 5b: Fit ARMA(1,1) Model to Differenced Log DPI
# ARMA(p,q) is ARIMA(p,0,q) on the stationary series
arma_model = ARIMA(df['Log_DPI_diff'].dropna(), order=(1, 0, 1))
arma_results = arma_model.fit()
print("\nARMA(1,1) Model Summary (on Differenced Log DPI):")
print(arma_results.summary())

# Step 6: Diagnostics for Both Models
# ARIMA Diagnostics
arima_residuals = arima_results.resid
plt.figure(figsize=(12, 6))
plt.subplot(121)
plt.plot(df.index.to_timestamp(), arima_residuals)
plt.title('ARIMA(1,1,1) Residuals')
plt.xlabel('Year')
plt.ylabel('Residual Value')
plt.subplot(122)
plot_acf(arima_residuals, ax=plt.gca(), lags=20)
plt.title('ACF of ARIMA Residuals')
plt.show()

# ARMA Diagnostics
arma_residuals = arma_results.resid
plt.figure(figsize=(12, 6))
plt.subplot(121)
plt.plot(df.index[1:].to_timestamp(), arma_residuals)  # Adjust index for differenced series
plt.title('ARMA(1,1) Residuals (Differenced Log DPI)')
plt.xlabel('Year')
plt.ylabel('Residual Value')
plt.subplot(122)
plot_acf(arma_residuals, ax=plt.gca(), lags=20)
plt.title('ACF of ARMA Residuals')
plt.show()

# Step 7: Forecast for Both Models
# ARIMA Forecast
arima_forecast = arima_results.forecast(steps=8)  # 8 quarters
forecast_index = pd.period_range(start='1992Q1', periods=8, freq='Q')
plt.figure(figsize=(10, 6))
plt.plot(df.index.to_timestamp(), df['Log_DPI'], label='Observed Log DPI')
plt.plot(forecast_index.to_timestamp(), arima_forecast, label='ARIMA(1,1,1) Forecast', color='red')
plt.title('ARIMA(1,1,1) Forecast')
plt.xlabel('Year')
plt.ylabel('Log DPI')
plt.legend()
plt.show()

# ARMA Forecast (on differenced series, then integrate back)
arma_forecast_diff = arma_results.forecast(steps=8)
arma_forecast = np.cumsum(np.concatenate(([df['Log_DPI'].iloc[-1]], arma_forecast_diff)))[:-1]  # Integrate back
plt.figure(figsize=(10, 6))
plt.plot(df.index.to_timestamp(), df['Log_DPI'], label='Observed Log DPI')
plt.plot(forecast_index.to_timestamp(), arma_forecast, label='ARMA(1,1) Forecast', color='green')
plt.title('ARMA(1,1) Forecast (Integrated from Differenced Log DPI)')
plt.xlabel('Year')
plt.ylabel('Log DPI')
plt.legend()
plt.show()
