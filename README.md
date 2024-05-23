# Portfolio_Optimization

- This repository contains code and documentation for a portfolio optimizer. The project aims to create and analyze various portfolio strategies using different financial instruments, with the end goal being to maximize the Sharpe ratio (Return/Risk). This README provides an overview of the project, details the processes involved, and includes relevant code snippets.

## Overview

- The project involves creating and analyzing portfolios using different strategies applied to various financial instruments, including securities, commodities, ETFs, and futures. The main objectives are:

1. Calculate and compare summary statistics for different strategies.
2. Implement mean-variance optimization (MVO) via Monte Carlo simulations.
3. Conduct sensitivity analysis for strategy parameters.

## Process


### Instrument Selection

##### Equities:
- AAPL, EXC, INTC, PFE, SPY

##### Fixed Income:
- FBNDX

##### Commodity:
- SPGSCI

##### Currency:
- CAD

**Rationale for instrument selection**

##### 1. Equities:
- AAPL and INTC are high-yielding tech stocks. They provide opportunities for growth and can benefit from high spending in the tech sector. However, they can introduce some volatility. 
- SPY, being a broad-market ETF, can help balance out some volatility by providing stability and diversification across sectors.
- EXC 

##### 2. Fixed Income Instruments:
- FBNDX can help balance out the equities and provide further stability. Lower expense ratio and higher quality funds compared to VBTIX as well.

##### 3. Commodity:
- SPGSCI can act as an inflation hedge owing to its broad commodity make-up.

##### 4. Currency:
- CAD can appreciate in value with a rise in commodity prices and also adds some balance to the portfolio. 

##### Correlation

FBNDX bonds, SPGSCI commodities, and CAD all exhibit low correlations with equities, providing diversification. If equities decline, bonds continue to hold value, commodities could rise with inflation threats, and CAD could go up with commodity strength.


### Sensitivity Analysis

- Next, passing a few arbitrary instruments through a crossover strategy (MAFlat, MAShort) to gauge optimal operating parameters for the portfolio. Also running the Bollinger Bands (BB) strategy on some instruments to get optimum parameters for the portfolio. Here are examples for a couple of tests.

![image](https://github.com/shahnishank/Portfolio_Optimization/assets/108402877/e74c6b18-0a6c-493a-a88a-49819c55fcab)

![image](https://github.com/shahnishank/Portfolio_Optimization/assets/108402877/8899a049-0c23-4a6f-ae2e-5367e14304de)


### Final Instrument-Strategy selection

- Looking at the sharpe ratios for the different strategies, and by trying various different combinations for our portfolio, we found the following combination to be the best mix of instruments. 

- Not all our selections were based purely on the individual instrument's highest Sharpe Ratio, for example, we found CAD to have ta slightly higher Sharpe ratio for the Bollinger Band strategy, however, our portfolio overall performs better with the Go-Short strategy returns for CAD.

- Final portoflio selections: 

 **AAPL-MAFlat, EXC-BB, INTC-BB, PFE-BB, SPY-MAShort, FBNDX-MAShort, SPGSCI-MAFlat, CAD-MAShort**.

#### Correlation in the final portfolio 

![image](https://github.com/shahnishank/Portfolio_Optimization/assets/108402877/c2f1c915-4722-4639-b957-a843465dcf25)

- We can observe that most of our instruments exhibit weak positive or negative correlation with the other instruments in the portfolio.
- There are no large swings in either direction, and the instruments not being correlated introduces independence in the portfolio, i.e., the performance of one asset may not be strongly tied to the performance of another, potentially reducing overall portfolio risk and providing the benefits of diversification.


### Benchmarking with an Equal-Weight portfolio 

- After deciding the strategies for every instrument individually, it is important to check how the portfolio performs when every instrument is assigned an equal weight in the portfolio. Here are the summary statistics for this portfolio.

![image](https://github.com/shahnishank/Portfolio_Optimization/assets/108402877/e8461677-1bb8-4436-927f-4cb369503a07)

NOTE: The Sharpe Ratio here is higher than any of the instruments individually, speaking to the viability of this portfolio and the instrument selection.

### Mean-Variance Optimization

- Running a Monte-Carlo simulation with 5000 points to assign random weights to the instruments, to find various characteristics of the portfolio at Minimum Volatility and Maximum Sharpe ratio.

```python
# MVO via a Monte-Carlo Simulation with 5000 paths. 

rng = np.random.default_rng(seed=64)
num_securities = final_pf.shape[1]
paths = 5000
pf_returns = []
pf_risks = []
sharpe_r = []
weight_vec = []

for i in range(paths):
    wts = rng.random(final_pf.shape[1])
    wts /= np.sum(wts)

    pf_return = (annualization_factor * np.dot(wts, final_pf.mean().T))
    pf_risk = np.sqrt(annualization_factor) * np.sqrt(np.dot(np.dot(wts, final_pf.cov()), wts.T))
    sharpe = pf_return / pf_risk

    pf_returns.append(pf_return)
    pf_risks.append(pf_risk)
    sharpe_r.append(sharpe)
    weight_vec.append(wts)
pf_returns = np.array(pf_returns)
pf_risks = np.array(pf_risks)
```

![image](https://github.com/shahnishank/Portfolio_Optimization/assets/108402877/d1ea3dc7-4f99-4574-9677-e98ed8701498)


- Using the weights generated above, to form a weight vector and assess the portfolio.

![image](https://github.com/shahnishank/Portfolio_Optimization/assets/108402877/fe2f4738-5bb9-4710-9214-6c7cd00dccd5)

#### Rationalizing the weight vectors 

- FBNDX and CAD having highest weights for the Minimum Volatility case. FBNDX, being a bond fund, is expected to be stable and low-risk and going short on it provides opportunities to hedge against interest rate risks and thus gets a higher weight while building a risk minimizing portfolio. Going short on currencies like CAD which can be volatile also hedges our portfolio against the associated risks. 
- AAPL has the highest weight for the Maximum Sharpe case. Being a high yield tech sector stock, the stock is expected to rally during bullish periods of the market and can be a reason behind its high weight when maximizing Sharpe.
- SPY holds steady in both cases, implying a clear stability and strength behind the ETF owing to its implicit diversification.

NOTE: The Maximum Sharpe Ratio achieved after performinng Mean Variance Optimization with the specified parameters is **1.37**, which is relatively higher than the observed Sharpe Ratio of the Equal Weight Portfolio, which was **1.18**.

### Calculating metrics for the final portfolio with the assigned weights

```python
# Using the Max Sharpe weight values from the weight vector and applying the weights to the final portfolio.

max_sharpe_returns = (final_pf * weight_vec_sharpe).sum(axis=1)

final_pf['MaxSharpe'] = max_sharpe_returns

print_pct_data(final_pf)

calcSummaryStatistics(final_pf)
```
![image](https://github.com/shahnishank/Portfolio_Optimization/assets/108402877/26a23fdb-b611-491b-8aba-0b3501721dd8)

- From this table, we can see that the Max Sharpe portfolio fares much better when compared to all the other instruments individually.
- It has a considerabbly higher Sharpe ratio of **1.37**, which is consistent with the Maximum Sharpe we achieved in our MVO.
- It also has a very low value for Max Drawdown at **-10.96%** which indicates that it does not go through large fluctuations.


### Calculating Beta


```python
# Fitting a linear regression model with the SPY Benchmark returns keeping the starting dates consistent, and the Max Sharpe Portfolio, to calculate the Beta.

port_Reg = LinearRegression().fit(merged_crossover_df.fillna(0)[['SPY-BMK']], final_pf['MaxSharpe'])
pf_beta = port_Reg.coef_
```
#### Beta Implications 

- A Beta of **0.106** vs SPY suggests that our portfolio returns are mostly independent from the overall stock market, with significantly lower volatility (risk) compared to a broad index fund, in this case, the S&P500. This could be beneficial during times of high market volatility. But in strong periods of the market, this low correlation means the protfolio may trail the higher returns of SPY.

### Final Observations

- Lastly, the final portfolio was compared and benchmarked against all the strategies individually with an equal-weight distribution among the instruments
- Equal weighted portoflio with all Benchmark returns has a Sharpe Ratio of **0.533**.
- Equal weighted portoflio with Go-Flat strategy applied to all instruments' returns has a Sharpe Ratio of **0.846**.
- Equal weighted portoflio with Go-Short strategy applied to all instruments' returns has a Sharpe Ratio of **0.461**.
- Equal weighted portoflio with Bollinger Bands strategy applied to all instruments' returns has a Sharpe Ratio of **0.161**.
- NOTE: None of these 4 portfolios outperform the Max Sharpe Portoflio, which had a Sharpe Ratio of **1.37**.
- Among these 4, the portfolio with all instruments run through the Go-Flat strategy has the highest Sharpe Ratio of **0.846**


