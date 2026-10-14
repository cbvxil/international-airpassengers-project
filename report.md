# AirPassengers Forecasting: Model Evaluation and Recommendation

## 1. Overview and Recommendation

AutoARIMA is the recommended model for forecasting the next 12 months of international air passenger demand. It achieved an average MASE of 0.647 across eight rolling forecast origins, outperforming the Seasonal Naive benchmark, which recorded a MASE of 1.313. This represents approximately a 51% improvement in scaled forecasting accuracy.

The model was selected based on its performance across multiple evaluation periods rather than relying solely on the framework's initial model ranking.

## 2. Understanding the Data

The dataset contains 144 consecutive monthly observations covering January 1949 through December 1960. Passenger numbers increased considerably during this period, rising from approximately 100 thousand per month to more than 400 thousand.

Two main patterns are visible in the series. First, there is a clear upward trend in passenger demand. Second, the data shows a repeating annual seasonal pattern, with higher passenger numbers during summer and lower numbers during winter. Seasonal fluctuations also become larger as the overall passenger volume increases.

The STL decomposition supports these observations. Trend strength reaches 0.996, while seasonal strength is 0.975. These values indicate that both trend and seasonality play important roles in the series. The autocorrelation values at lags 12 and 24 are 0.760 and 0.532, respectively, confirming that observations from previous years contain useful information.

Therefore, an effective forecasting model must account for both long-term growth and recurring seasonal changes.

## 3. Benchmarking and Model Selection

Seasonal Naive was used as the main benchmark because it forecasts each month using the value observed in the same month of the previous year. A standard Naive model was also included for comparison.

Although Seasonal Naive captures the annual pattern, it struggles to follow the continuous increase in passenger demand. Its residuals show a lag-1 autocorrelation of 0.746, indicating that some systematic information remains unexplained. In contrast, the lag-12 residual autocorrelation is -0.044.

AutoGluon TimeSeries was used to evaluate five candidate models: Naive, SeasonalNaive, AutoETS, AutoARIMA, and Theta. The framework's validation scores helped identify promising candidates, but the final decision was based on a shared rolling cross-validation procedure.

The evaluation used eight forecast origins, a 12-month prediction horizon, and a 12-month step between origins. Each model was fitted using only the information available at the corresponding forecast origin, reducing the risk of data leakage.

The final results across 96 held-out monthly forecasts are shown below.

| Model | MASE | Scaled CRPS | 80% Coverage | Mean 80% Width | Fold MASE Range |
|---|---:|---:|---:|---:|---:|
| AutoARIMA | 0.647 | 0.0339 | 69.8% | 44.1 | 0.330–1.594 |
| AutoTheta | 0.898 | 0.0460 | 59.4% | 57.6 | 0.451–1.405 |
| AutoETS | 0.952 | 0.0499 | 49.0% | 48.7 | 0.565–1.554 |
| Seasonal Naive | 1.313 | 0.0616 | 51.0% | 77.8 | 0.412–1.959 |
| Naive | 2.119 | 0.1103 | 68.8% | 142.9 | 1.422–3.196 |

AutoARIMA delivered the best overall results, achieving the lowest MASE and the lowest scaled CRPS. Its performance indicates that it provides both more accurate point forecasts and better probabilistic forecasts than the alternatives.

However, performance varied across evaluation periods. Its highest fold MASE was 1.594, showing that some periods remain more difficult to predict. AutoTheta and AutoETS also outperformed Seasonal Naive in average point accuracy, but neither matched AutoARIMA's overall results.

## 4. Evaluating Forecast Uncertainty

Accurate point forecasts are not sufficient for planning decisions. Prediction intervals must also represent uncertainty reliably.

AutoARIMA produced an average 80% prediction interval width of 44.1 thousand passengers, but the intervals contained only 69.8% of the actual observations. This is below the intended coverage level of 80%, suggesting that the model underestimates uncertainty in some periods.

Seasonal Naive recorded a lower coverage rate of 51.0%, despite producing wider intervals averaging 77.8 thousand passengers. The standard Naive model achieved 68.8% coverage, but its intervals were considerably wider, averaging 142.9 thousand passengers.

These findings support the use of AutoARIMA for the central forecast while highlighting the need to improve its uncertainty estimates before using them for operational capacity planning.

An initial improvement would be to adjust the prediction intervals using historical forecast errors. The adjusted intervals should then be evaluated through rolling-origin testing to determine whether their empirical coverage approaches the nominal 80% level.

## 5. Residual Analysis

The residual analysis provides additional evidence of the model's effectiveness. After excluding the initial 12-month warm-up period, AutoARIMA's fitted residuals had a mean of approximately 0.24 thousand passengers.

The residual autocorrelation was approximately -0.001 at lag 1 and -0.109 at lag 12. No strong, consistent autocorrelation pattern was observed across the first 24 lags.

Compared with the Seasonal Naive benchmark, AutoARIMA appears to capture the main trend and seasonal structure much more effectively. Nevertheless, larger errors remain around the highest peaks and lowest passenger-demand periods.

The increasing size of some residuals at higher passenger volumes suggests that the remaining challenge may be related to changing variance rather than an unmodeled seasonal pattern. This observation is also consistent with the lower-than-expected prediction interval coverage.

## 6. Recommended Next Step

The next development step is **rolling empirical prediction interval calibration by forecast horizon**.

For each of the 12 forecast lead months, historical absolute forecast errors from previous rolling origins can be used to estimate appropriate interval adjustments. Only errors that were already available at the time of each forecast should be included, ensuring that future observations do not influence earlier predictions.

The objective is to improve the empirical coverage of the nominal 80% intervals while maintaining reasonable interval widths. This adjustment is intended to improve uncertainty estimates without changing AutoARIMA's existing central forecasts.

The calibration process should be reassessed whenever new observations become available. If under-coverage continues, a log transformation or Box-Cox transformation could be investigated to address the increasing variability associated with higher passenger volumes.

## 7. Conclusion

The evaluation identifies AutoARIMA as the strongest candidate for forecasting AirPassengers demand over a 12-month horizon. Its MASE of 0.647 and scaled CRPS of 0.0339 demonstrate a clear improvement over the Seasonal Naive benchmark.

The results also show that forecast uncertainty requires further attention. Although AutoARIMA provides the most accurate central predictions among the tested models, its prediction intervals do not yet achieve the intended 80% coverage.

The recommended approach is therefore to use AutoARIMA as the forecasting model, introduce horizon-specific empirical interval calibration, and continue monitoring performance as new data becomes available.

All reported performance metrics were calculated using the notebook's common eight-origin rolling cross-validation procedure. The single-window AutoGluon leaderboard was used only to shortlist candidate models and was not treated as the final basis for the recommendation.
