#### [<back to projects](./projects.md)
# Optimally Weighted Porfolio
## Introduction
Portfolio optimization is a fundamental concept in finance that focuses on selecting the best mix of assets to maximize returns while minimizing risk. One of the most widely used methods for evaluating and optimizing portfolios is based on the Sharpe Ratio, developed by Nobel laureate William F. Sharpe in 1966. The Sharpe Ratio measures the performance of an investment compared to a risk-free asset, after adjusting for its risk. This method, in combination with minimum variance, and equally weighted methods will be used in evaluating the performance of a portfolio of 20 securities over the span of a year. 

![Sharpe Ratio](images/OptimalPortfolioProject/Sharpe Ratio.png)

## Gathering the Securities
Adjusting closing prices for the 20 securities chosen will account for dividends and stock splits if applicable. In figure two, we must standardize to get % returns per month. This will allow for comparison between the securities.

![Figure 1](images/OptimalPortfolioProject/Fig1.png)

 
![Figure 2](images/OptimalPortfolioProject/Fig2.png)

## Capital Asset Pricing Model
In Figure three, the important thing to note is the 4 year beta has now been calculated for each security. This is the stocks volatility relative to the market as a whole. Essentially what were are doing here is finding a peice of the puzzle for the Sharpe Ratio. 
![Figure 3](images/OptimalPortfolioProject/Fig3.png)
![Figure 4](images/OptimalPortfolioProject/Fig4.png)

## Solving for each metric
In figures five and six we have solved for the weights of each security in each of the three portfolios. We have set the goal to maximize sharpe ratio subject to each stock cant have a higher than 20% weight, no non-negativity, and must equal 100%. The same was completed for minimum variance. This time solver was run to minimize the goal of variance. 

![Figure 5](images/OptimalPortfolioProject/Fig5.png)
![Figure 6](images/OptimalPortfolioProject/Fig6.png)

## Performance 
After solving, we then used $500,000 and invested it into each of these portfolios that we created. We then tracked the performance of each portfolio over a year. the results are shown below. 
![Figure 7](images/OptimalPortfolioProject/Fig7.png)
![Figure 8](images/OptimalPortfolioProject/Fig8.png)
![Figure 9](images/OptimalPortfolioProject/Fig9.png)

## Results
Here are the final results. In red we see the optimal portfolio behaving as it should. Slightly more volatile, but with greater returns to compensate. We did outperform all other portfolios, as well as the S&P 500. 
![Figure 10](images/OptimalPortfolioProject/Fig10.png)
