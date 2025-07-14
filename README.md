# ARIMA-Energy-Price-Forecaster
## Summary
Forecasted New Zealand energy prices using one year of half hourly data from the New Zealand Electricity Authority. Conducted stationarity diagnostics and applied first-order differencing to stabilize the series. Performed autocorrelation and partial autocorrelation analysis to inform model development. Built and evaluated four forecasting models ARIMA, STL-ARIMA, ARIMAX, and TBATS. Compared model performance using forecast accuracy metrics including ME, MAE, and RMSE.

The  ARIMA(3,1,1)(0,0,2)[48] demonstrated the highest accuracy across the ME, RMSE, MAE, MPE, MAPE, and ACF1 metrics.

This project was made using RStudio.

# Data cleaning and wrangling
I loaded in the training set which was 12 months of half hourly data from the Electricity Authority.

I restricted the dataset to one Point of Connection (ABY0111) for simplicity.

I ensured that the datasets variables were classified correctly.

The TradingDate variable was a date object ranging from 2017-01-01 to 2017-12-31.

The TradingPeriod variable gives the 30 minute interval where electricity was bought and sold. It was numerical.

The PointOfConnection variable was an ordinal categorical variable, it gives the grid location where electricty is entering (or exiting) a network. I’m only forecasting with the ABY0111 point of connection.

The DollarsPerMegawattHour variable was numerical, it gives the wholesale price of electricity. It’s the price at which electricity was bought and sold in the wholesale market at a specific date and time, and point of connection.

The TradingPeriod had a max value of 50 even though there are only 48 trading periods in New Zealand’s electricity market. This was because daylight savings ended on 2017-04-02 and clocks were turned back one hour, meaning there was an extra hour for that day, resulting in two more 30 minute trading periods
I removed these observations from the dataset as I wanted to maintain a consistent daily structure, which is important for ARIMA.

There were no missing values or NA's in the dataset

# EDA

![Alt](Images/1.png)

There were very few values above 400 for DollarsPerMegawattHour. The mode for DollarsPerMegawattHour was around 70 dollars. There were no negative values for DollarsPerMegawattHour.

There are 34 observations where the DollarsPerMegawattHour value was larger than the 400, which in a data set of 17158 observations was very few.

The mean DollarsPerMegawattHour for year 2017 at point connection ABY0111 was 78.71 (round to 2.dp)

![Alt](Images/2.png)

I saw that there were specific dates where the whole sale price of electricity spiked to unusually high amounts.
There were spikes in electricity price on 2027-05-22, 2017-07-13, 2017-09-11, and 2017-10-19.

![Alt](Images/3.png)

The price of electricity was much higher for June and July. The lower quartile of June and July doesn't even overlap with the upper quartiles of the previous months.
The prices dip again for August, September and October but then start increasing again in November and December.

# Autocorrelation & Partial Autocorrelation Analysis

![Alt](Images/4.png)

ACF:
I saw that all of the spikes in the ACF plot are outside of the blue dashed significance bound this means that the time series data was highly autocorrelated and non-stationary. The autocorrelation at all of the lags was statistically meaningful and not just white noise. Past values have a strong influence on future ones across many lags.
There was particularly high autocorrelation at the early lags. This suggests that recent values strongly influence near future behavior, implying short term memory or an autoregressive structure.
There was a spike at around every 48 lags which suggests a seasonal pattern.
The bars gradually decay suggesting a persistent trend or that it's non-stationary. This suggests the mean and variance may be changing over time, and the structure could be due to an autoregressive process or an underlying seasonal component.

PACF:
There was a large spike at lag 1 in the PACF plot suggesting that the current price was heavily influenced by the immediate past period. This spike at lag 1 suggests an AR(1) structure. This supports the idea that the series has short-term autoregressive structure.
Past lag 1 there was gradual tapering, rather than a sharp cutoff like a pure AR(p) process.

Conclusion:
All of the above information from the ACF and PACF plots suggests that the data was non-stationary and that there could be a seasonal component to the data.
I applied first order differencing to the data to make the data stationary, as stationary data is a requirement of ARIMA.

The subsequent acf and pacf plots for the first order differenced time series were:
![Alt](Images/5.png)

ACF:
After applying first order differencing most lags in the ACF plot were within the confidence bounds, meaning there was no strong autocorrelation.
There was no sharp cutoff or pattern in the plot. There were no clear periodic spikes (e.g. at lag 48), which indicated that there was no seasonality left.
The ACF plot no longer shows a slow decay, instead there are isolated significant spikes in the plot. This pattern indicated that the trend had been removed and that the series was now stationary, with stable mean and variance over time.

PACF:
There was a moderate spike at lag 1, outside the confidence bounds. Most subsequent lags are within the confidence bounds. This suggests a short term autoregressive pattern. This means each electricity price was influenced by its immediate predecessor.
The decay after the first lag also supports that the first order differencing has stabilized the series, leaving behind no long range autocorrelation.
Since this resembles an AR(1) structure still, when I fitted the ARIMA model I included an AR(1) component to model the autoregressive behavior.

# Stationarity Diagnostics: ADF, KPSS & Phillips–Perron

I performed an Augmented Dickey-Fuller test and Phillips–Perron test both of which had a p-value of less than 0.01, due to the small p-value I concluded that the differenced series was stationary. The KPSS test had a p-value larger of 0.5 as a result I concluded that the differenced series was stationary.

# Accuracy metrics

I fitted four models:

ARIMA(3,1,1)(0,0,2)[48]

STL-ARIMA(5,1,1)

An ARIMAX model where the xreg arguement had five predictors (Day_of_Week, Is_Weekend, Month, Hour_of_Day, Is_peak)

A TBATS model with included ARMA errors.

They were all trained on a time series containing 12 months of half hourly final energy price data from 2017-01-01 to 2017-12-31.

After forecasting with those models I obtained the accuracy metrics:

|         | ME   | RMSE   | MAE   |  MAPE |  MAPE | MASE  | ACF1   |
|---------|------|--------|-------|-------|-------|-------|--------|
ARIMA     |6.56	 | 19.17  |	13.71	| 3.29	| 15.65	| 0.70	| 0.65   |
STL_ARIMA |12.25 |	21.63	| 17.83	| 10.29	| 19.71	| 0.91	| 0.66	 |
ARIMAX    |53.42 |	56.51	| 53.42	| 58.78	| 58.78	| 2.74	| 0.65   |
TBATS     |2.16  |	23.57	| 17.06	| -2.10	| 20.54	| 0.87	| 0.74   |

The ARIMA model had the best RMSE, MAE, MPE, MAPE and MASE. ARIMA was the most accurate forecasting model.

The worst model was ARIMAX with the worst ME, RMSE, MAE, MPE, MAPE and MASE.

TBATS had the best ME but had higher MAE, MAPE, and MASE than STL-ARIMA

All models had similar ACF1 values which suggests that the residuals for all models still had short range correlation.

The MASE for ARIMA, STL-ARIMA and TBATS was less than 1 which means they beat a naive "yesterday's price" benchmark on average absolute error.

I decided to check the residuals of the ARIMA and STL-ARIMA models and perform model refinement. I did not be continue with the worst performing models ARIMAX and TBATS.

# Structural checks and Statistical diagnostics
## Autocorrelation check with Ljung Box test, acf and pacf plots:
### ARIMA model
The p-value from the Ljung Box test was extremely small as a result I rejected the null hypothesis that there was no autocorrelation in the residuals and concluded that there was autocorrelation in the residuals. This means that the ARIMA model hadn’t fully captured the time-dependent structure in the series.

For Anderson-Darling normality test the p-value was very small, so I rejected the null hypothesis that the residuals are normally distributed and concluded that the residuals were non-normal.

Checking for constant variance:
![Alt](Images/8.png)

As the fitted values increase, the residuals fan out, showing increasing variance.
There’s a tight cluster around zero residuals for lower fitted values

![Alt](Images/9.png)

Most autocorrelation bars fall within the dashed blue confidence bands, suggesting they’re not statistically significant.
There was no strong pattern of lingering autocorrelation. The residuals behaved like white noise.

![Alt](Images/10.png)

Most lags fell within the blue dashed confidence bands, meaning their partial autocorrelations weren’t statistically significant.
There are a few spikes outside of the confidence bounds, particularly at lag 1 suggesting short-term autocorrelation. This implied that the AR term (p) was slightly under-specified. I tried increasing the AR component to try and capture the remaining autocorrelation.

### STL-ARIMA model
The p-value from the Ljung Box test was extremely small as a result I rejected the null hypothesis that there was no autocorrelation in the residuals and concluded that there was autocorrelation in the residuals. This means that the STL-ARIMA model hadn’t fully captured the time-dependent structure in the series.

For Anderson-Darling normality test the p-value was very small, so I rejected the null hypothesis that the residuals were normally distributed and concluded that the residuals were non-normal.

![Alt](Images/13.png)

As the fitted values increase, the residuals fan out, showing increasing variance. There’s a tight cluster around zero residuals for lower fitted values.

![Alt](Images/14.png)

The models residuals were like white noise, most of the bars were within the confidence bounds

![Alt](Images/15.png)

There are significant spikes outside the confidence bands, this suggests that there are lags that aren’t captured by my current AR structure.
I tried increasing the AR component to try and capture the remaining autocorrelation.

# Model refinement
I applied a box cox transformation to the time series to try and resolve the non-normality.

![Alt](Images/16.png)

There was a curved pattern in the data which indicated that the data was skewed or was not normally distributed

![Alt](Images/17.png)

The data was not centered around zero, so its not a standard normal distribution. The data also appeared to be right skewed. This plot indicated that the data was not normally distributed.

For the Anderson-Darling normality test it had a small p-value which indicate that the data was still not normal even after a BoxCox transformation

The data was not normally distributed, but the box-cox transformed time series was closer to a normal distribution than the untransformed time series, so I continued using it for further modelling.

I checked for outliers with Tukey’s Fences and visualized that in a time series plot where each red dot was an outlier:
![Alt](Images/19.png)

There are 84 outliers according to Tukey’s Fences. Since energy prices are so volatile, I decided  not to discard the outliers in the series.

For both models (ARIMA and STL-ARIMA) I increased the AR component by 1, and applied a box-cox transformation to the time series data.
I now had:
ARIMA model 2: ARIMA(4,1,1)(0,0,2)[48]
STL-ARIMA model 2: STL-ARIMA(6,1,1)

For another two models I increased the AR component by 1, but did not apply a box-cox transformation to the time series data. I did this because I wanted to see if the box-cox transformation of the series actually had a significant improvement on the model. These models were ARIMA model 3 and STL-ARIMA model 3.

# Model assessment
I forecast 48 trading periods for twelve months for my four models, ARIMA model 2, ARIMA model 3, STL-ARIMA model 2 and STL-ARIMA model 3.

I made sure to apply an inverse box cox transformation on the forescasts of ARIMA model 2 and STL-ARIMA model 2 to obtain interpretable electricity price predictions since the ARIMA 2 and STL-ARIMA 2 models were trained on a box cox transformed time series.

|           | ME   | RMSE   |   MAE | MAPE     | MAPE    | MASE  | ACF1  |
|-----------|------|--------|-------|----------|---------|-------|-------|
ARIMA       |6.56	 | 19.17  |	13.71	| 3.29	   | 15.65	 | 0.70	 | 0.65  |
ARIMA 2     |18.20 | 102.34	| 52.97	| -2091.39 | 2119.70 | NA	   | 0.91  |
ARIMA 3     |6.56  | 19.17	| 13.71	| 3.29	   | 15.65	 | 0.70  |	0.65 |
STL_ARIMA   |12.25 | 21.63	| 17.83	| 10.29	   | 19.71	 | 0.91	 | 0.66	 |
STL_ARIMA 2 |18.69 | 101.89 | 52.76 |	-1999.73 | 2028.92 |	NA	 | 0.91	 |
STL_ARIMA 3 |5.77	 | 19.96  |	15.61 |	2.50     | 18.36   |	0.80 |	0.69 |

The MASE could not be calculated for my two of the models (ARIMA 2 and STL-ARIMA 2) presumably due to the box-cox transformation I applied to the time series for those models, so I will be focusing on the other available metrics.

The original ARIMA model and ARIMA model 3 had identical results which means that increasing the AR component from AR(3) to AR(4) did not significantly improve the model. The original ARIMA model and ARIMA model 3 had the best results in regards to their RMSE, MAE, MAPE, MASE and ACF1. Thesse models seem to have strong accuracy and are realiably forecasting final energy prices for 2018.

The 2nd ARIMA model and the second STL-ARIMA model had the worst performance with extremely high RMSE and MAE. The models had the highest ACF1.
Slightly higher MAE, RMSE, and MASE than ARIMA, but solid MAPE and relatively low ACF1

The model that performed best was my original ARIMA model.

The prior adjustments to the original ARIMA and STL-ARIMA models did not improve the ARIMA model's performance but it did improve the STL-ARIMA model's performance.
The box-cox transformation of the series made the performance of the models much worse.
The best model was ARIMA(3,1,1)(0,0,2)[48] and the worst models were STL-ARIMA 2 and ARIMA 2.

![Alt](Images/20.png)

This plot shows my ARIMA model forecast for the final half hourly electricity prices for the year 2018 compared to the actual values for that year.

There are large fluctuations in the actual electricity prices, particularly in July and November.

The forecast as shown by the blue dashed lines appears to follow the overall trend of electricity prices. Its 95% confidence interval (the light blue ribbon) covers the majority of the electricity prices for 2018 but misses many large spikes between June and December which means the model was underestimating the real volatility.

However it does model the initial five months January to May quite well.

Due to the models issues with volatility if I were to continue forecasting the 2018 energy price data with another model I would consider a GARCH model as it is good for modelling volatility clustering.

# Conclusion
The best model was the ARIMA model: ARIMA(3,1,1)(0,0,2)[48].

Its non-seasonal component had AR(3) (which captured short term autocorrelation up to lag 3), I(1) (a first order differencing to make the data stationary), MA(1) (which corrected for noise and shocks using lag 1 residuals). Its seasonal component with a periodicity of 48 (the seasonality repeats every full day as there are 48 trading periods per day) and it did not include a seasonal AR or I (no seasonal autocorrelation or differencing) and had a seasonal MA(2) (uses forecast errors from 1 and 2 days ago (lags at 48 and 96 intervals) to adjust the current days forecast).

The models mean error was 6.56 which means on average its forecasts overestimate actual prices by about 6.56 dollars per megawatt hour.

The models root mean square error was 19.17 which indicates large forecast errors, since squared errors penalize larger errors more heavily this high RMSE value is probably due to how volatile the data is.

The models mean absolute error was 13.71 which means that forecasts were off by 13.71 units on average. This is 13.6% of the annual mean, the mean energy price for 2018 was 100.62, so the mean absolute error isn't that high considering how volatile the data is.

The mean percentage data was 3.29% which means that the model tends to forecast higher electricity prices than reality.

The mean absolute percentage error was 15.65% which means that the models forecasts were off by 15.65% on average.

The models MASE was 0.70 which is a scaled comparison to a naive model which means the ARIMA model outperforms a naive forecast.

The models ACF1 was 0.65 which indicates that the residuals are moderately autocorrelated so there is some structure or pattern in the data that the model did not capture.

The ARIMA model did better than the STL-ARIMA, ARIMAX and TBATS models which just shows that a more complex model does not equal a better performance, those other models were overfitting on the training data and not generalizing well enough to forecast the following year.

# References and Citations
Electricity Authority. (n.d.). Final energy prices by month [Dataset]. EMI – Electricity Market Informa-
tion. Retrieved between May 2 and July 13, 2025, from https://www.emi.ea.govt.nz/Wholesale/Datasets/
DispatchAndPricing/FinalEnergyPrices/ByMonth
