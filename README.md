# Huatai-Securities-Internship
Researched a four-contract cointegration system across the steel chain (hot-rolled coil, rebar, iron ore, coking coal), applying ADF stationarity tests and the Johansen test to identify two long-run equilibrium relationships.

In this project, we developed a statistical arbitrage strategy involving four economically related Chinese steel chain futures: iron ore, coking coal, rebar, and hot-rolled coil. They are connected because they represent different stages of the steel supply chain. Since our goal is to profit from the deviations and moments when they return toward equilibrium, we first have to question whether a stable combination of these steel chain futures can create a mean-reverting spread.

This project takes the daily futures data from July 2020 to July 2025. We used approximately two-thirds of the data for training (July 29, 2020, to December 29, 2023, the first 833 observations) and the remaining data for testing (December 30, 2023, to July 29, 2025). 

Now, we briefly explain our analysis procedures. First, we used the Augmented Dickey-Fuller test (ADF) to check whether each of these stock prices is I(1), i.e., stationary after first differencing. Once this is confirmed, we can proceed, as this is a safety check. We used a vector autoregression to model the four futures and used the Akaike Information Criterion (AIC) to select the optimal lag, which was 2 because it had the lowest AIC value. Then, a Johansen test is performed to estimate the coefficients for each future to determine whether the spread is mean-reverting. The test indicates that the Johansen matrix has rank 2, implying there are two independent stationary spreads for the mean-reversion strategy. Given this, we then used the ADF p-value test to identify the better spread by selecting the one with a lower p-value (p-value 0), meaning it has stronger statistical evidence against a unit root, in simple words, more likely to be mean-reverting. We chose the blue line below.
<img width="1834" height="775" alt="a1d3abaf-9ac8-40cd-9dbd-ba2fc6d278f8" src="https://github.com/user-attachments/assets/7006528b-5ae7-4140-970e-cf70faa0cbf5" />

Before we proceed to the strategies, it is crucial to convert the cointegration vectors into a practical number of contracts we have to buy to simulate the spread. For Chinese steel chain futures, iron ore and coking coal are 100 tons per contract. Rebar and the hot-rolled coil are 10 tons per contract. Note that the initial cointegration vector is {Hot-Rolled Coil: 5.00051, Iron Ore: 0.6142, Coking Coal: 1.56603, Rebar: −6.57481} with unit being ton. After converting tons to contracts, the cointegration vector becomes {Hot-Rolled Coil: 0.500051, Iron Ore: 0.006142, Coking Coal: 0.0156603, Rebar: -0.657481}, but we cannot use a non-integer number of contracts, so we should scale them and try our best to keep them in proportion. Theoretically, if we scale by 100000, it would be perfect, but that would require too much upfront cost when entering the trade because of the large contract size. To be more realistic, we scaled by 1000 and rounded, so the cointegration vector (after rounding) becomes {Hot-Rolled Coil: 500, Iron Ore: 6, Coking Coal: 16, Rebar: −657}.

We mainly developed two trading strategies. The first one is a z-score trading threshold, and the second is a plain vanilla. Since we are mostly confident that the spread we chose is mean-reverting, we can go long the spread when it is below a certain threshold and flatten our position when it crosses the mean line (where z=0). Similarly, we short the entire spread when it rises above a certain threshold and then flatten our position when it reverts back and crosses the mean line. We use a rolling window to determine the mean line. The rolling window length and the z-score threshold are determined solely from the training set, using a heatmap and several evaluation metrics we introduce later.

The second trading strategy is similar. It is called plain vanilla due to its simplicity. Once the spread moves above the mean line, we go short the spread; when it crosses the mean line, we flatten our position. Vice versa. This strategy captures more trading opportunities because it has a very low barrier to entry. In return, it results in a much higher transaction cost.

PnL calculations: The PnL change over one day is calculated as the spread on day t multiplied by the position on day (t-1), since we assume there are no intra-day trades; all prices are end-of-day prices. Hence, the cumulative PnL is just the sum of every day’s PnL. At the same time, we have to subtract the resulting transaction costs, which are assumed to be 0.5% of the gross exposure when there is a position change. Gross exposure = sum of the absolute value of notional positions for each future.

Parameters for performance judging: 
Sharpe Ratio: average daily return / standard deviation of daily return * sqrt (252)
Maximum Drawdown: The worst peak-to-trough decline in profit.
Win Rate: number of profitable days / number of days with position
Number of Trades: Total number of position entries.
Average Holding Time: Mean duration in days a trade is held.

We then use these five parameters to create different performance evaluation formulas, each focusing on different aspects.
C = 0.20 · S + 0.10 · D + 0.10 · W + 0.35 · T + 0.25 · H
C = 0.25 · S + 0.15 · D + 0.10 · W + 0.25 · T + 0.25 · H
C = 0.30 · S + 0.20 · D + 0.15 · W + 0.20 · T + 0.15 · H
(S = Sharpe Ratio, D = Maximum Drawdown, W = Win Rate, T = Number of Trades, H = Average Holding Time)
Generally, the Sharpe Ratio weighting prefers moderate z-scores and moderate rolling window lengths. The maximum drawdown weighting favors high z-score thresholds and a longer rolling window to capture more stable reversals. Win rate prefers a high z-score and a moderate rolling window for accurate entries. Lastly, both the number of trades and average holding time prefer a low z-score and short rolling windows to capture more trades. In simple terms, we can treat the first three parameters as z-score boosting and the last two as anti-z-score boosting.

Hence, the first formula uses 40% z-score boosting parameters, the second uses 50%, and the third uses 65%. They prefer different things, so we use these three evaluation formulas to find the near-optimal values for z-score and rolling window length.

Heatmap for the first evaluation: C = 0.20 · S + 0.10 · D + 0.10 · W + 0.35 · T + 0.25 · H
<img width="1282" height="782" alt="3d81f650-c697-49a2-96b0-ed76f1d990c3" src="https://github.com/user-attachments/assets/9d9a5768-7685-4a27-a260-a0b7a2e685f6" />

Heatmap for the second evaluation: C = 0.25 · S + 0.15 · D + 0.10 · W + 0.25 · T + 0.25 · H
<img width="1284" height="782" alt="ae215459-5afc-45cc-808c-e7c6082ad3ea" src="https://github.com/user-attachments/assets/6172316b-f597-48af-8629-8787b4a30c87" />

Heatmap for the third evaluation: C = 0.30 · S + 0.20 · D + 0.15 · W + 0.20 · T + 0.15 · H
<img width="1276" height="790" alt="fc79f095-e85b-4cd5-a3e6-51f918061d6f" src="https://github.com/user-attachments/assets/010996f0-6d97-459d-8392-0a1c4dc333bb" />


Graph for Plain Vanilla using first evaluation: C = 0.20 · S + 0.10 · D + 0.10 · W + 0.35 · T + 0.25 · H
<img width="1261" height="623" alt="fe5652b1-81eb-4478-bd34-697990e934f4" src="https://github.com/user-attachments/assets/cd6297dd-3c86-442c-8b4e-591a260d8deb" />

Graph for Plain Vanilla using the second evaluation: C = 0.25 · S + 0.15 · D + 0.10 · W + 0.25 · T + 0.25 · H
<img width="1269" height="639" alt="02f3932b-2374-4413-858b-8ac3c1929ca6" src="https://github.com/user-attachments/assets/a17e5a91-f342-4b6d-829e-cdedb05a0136" />

Graph for Plain Vanilla using the third evaluation: C = 0.30 · S + 0.20 · D + 0.15 · W + 0.20 · T + 0.15 · H
<img width="1269" height="624" alt="904be210-5909-4faf-ad59-9f72a68848c7" src="https://github.com/user-attachments/assets/08259350-3b00-4c7c-beb7-3e67070ec98c" />

All three heatmaps for the z-score strategy yield the same best z-score and rolling window length: ±1.8 z-score and a rolling window size of 94. For the Plain Vanilla strategy, the first two evaluation formulas identify a window size of 41 as optimal, whereas the third identifies 116 as optimal, since no z-score is needed.


Out of these three, the most profitable strategy during the testing period is the plain vanilla strategy with a rolling window length of 116 days. From the out-of-sample test, this strategy earns 4023517.05 CNY; divided by the average gross exposure of 39,480,278.71 CNY, it gives the percentage return of 10.19%. After annualizing, it becomes 6.76%, roughly 7% annual profit.

