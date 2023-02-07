## Balancer

### Index

S&P 500 is stock market index tracking the stock performance of 500 large companies listed on stock exchanges in the United States. The price movement of these 500 top companies can be known by this S&P500 index.So when we say "S&P500 crossed 5000."This means the value of the index of all those companies got raised to 5000.

### Index fund

If any fund that spends in such index, we call them index fund.Basically index fund is a kind of  mutual fund only. In mutual fund, we can invest in stocks in different ways but in index fund it's a clear mandate. This means if any index fund is saying "We'll invest in S&P500", they'll invest in s&P500 only.

## What is Balancer?

A Balancer Pool is an automated market maker with a few crucial characteristics that enable it to act as a price sensor and self-balancing weighted portfolio.Balancer turns the concept of an index fund on its head: instead of paying fees to portfolio managers to rebalance your portfolio, you collect fees from traders, who rebalance your portfolio by following arbitrage opportunities.The Balancer is built on a specific N-dimensional surface that establishes a price for exchanging any two tokens held in the Balancer pool. 

Balancer allows for its trading pairs (called pools) to consist of multiple tokens — anywhere between 2 and 8, each token with a different arbitrary share of the pool (from 2% to 98%). This differs from 50/50 AMMs, which depend on the x*y=k equation. Balancer Pools are extremely customizable, allowing anybody to create a pool with custom fees (ranging from 0.00001% to 10%).

This enables Balancer to provide a kind of self-balancing index fund that is the opposite of an ETF, where instead of paying a portfolio manager to maintain the ETF's balance as the prices of the assets that make it up change inexorably, in the inverse ETF you, as a liquidity provider, get paid when the ETF is rebalanced. This works because market participants are compelled to rebalance the portfolio in order to seize arbitrage opportunities. You get paid as the fund's investor through their fees. 

A Balancer pool has the following variables:
```
    Change Tokens — add or remove tokens from the pool (2 to 8)
    Change Weights — change the weighting of any token in the pool (2% to 98%)
    Change Fee (0.00001 to 10%)
    White/blacklist LPs — limit the particular addresses that can become LPs in the pool
    Limit Max Deposited Value — limit the maximum value LPs can deposit
    Start/Stop Trading — pause trading for the pool.
```

With that, there are three types of pools:

```
    public pools (also called shared)— anyone can add liquidity (and get Balancer Pool Tokens in return), but all the pool parameters are fixed forever. (trustless, finalized)
    private pools — all the parameters are flexible — only the owner can change them but also only the owner can add liquidity (trusted, unfinalized)
    smart pools — anybody can add liquidity to them and the parameters can be fixed or dynamic controlled through smart contracts. (trustless, flexible).
```

Pools with strong assets having a higher percentage usually results in less imparmanent loss.

[impermanentLoss](Images/n4.png)

The trade-off for this imparmanent loss is usually a higher slippage amount for traders.Essentially the result of less liquidity for the tokens with less weight.


