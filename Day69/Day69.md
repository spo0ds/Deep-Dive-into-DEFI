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

![impermanentLoss](Images/n4.png)

The trade-off for this imparmanent loss is usually a higher slippage amount for traders.Essentially the result of less liquidity for the tokens with less weight.

### Liquidity bootstrapping pools

Unfortunately, Uniswap’s 50/50 approach to token pricing means that little total volume can produce huge swings in pricing, creating both irrational price-discovery and serving a bad job as a token distribution mechanism where bots can front-run the community with the ability to run a pump & dump scheme on them. Further, this model requires a lot of capital to be deployed from the founding team (after all, the other 50% needs to be in an established token like ETH).

To solve this, Balancer offers the so-called Liquidity Bootstrapping Pools. This means that Balancer supports automatic rebalancing of specific assets. It has another pool type called "liquidity bootstrapping pool," which is primarily used for newly created tokens or projects. The idea is that they generally start a pool with 10% of a highly liquid or trusted coin and then 90% of a new project's coin. The project will also select an ending weight, typically 50/50. One of the last primary steps is to set a specific timeframe to run the liquidity bootstrapping pool. It could be days, weeks, or even months, and once the pool begins, the ratio of assets will change over that specified time period. So basically, it will start off at 90/10 and end up at 50/50. The result is that the token price continually experiences downward pressure throughout the sale. When this is mixed with modest buying demand, the price stays stable throughout the sale, as whales/bots are disincentivized to buy it all at once.It's a very successful way to launch a new token.

## How does the balancer work?

Balancer has two primary functions: portfolio manager and AMM. There's a smart contract for each pool. In each pool, you have a combination of 2–8 ERC-20 tokens. The first step in the pool lifecycle is to create a pool. Anybody could create a pool using the "BFactory" contract. When we create a new pool, it creates a new smart contract and registers its address in the B Factory contract. It also registered the address that initiated the transaction, which is the controller of the pool and has some special rights.

After a pool has been created, it's in a state called "Not Finalized." The controller is the only address that can interact with the pool until it is finalized. It's a private pool. During this state, the controller can change the parameters of the pool tokens, weights in the pool, and trading fee. This trading fee will be given to the liquidity providers for every trade and for the ability for others to trade with the pool in its not-finalized state. It's possible to stay in its not-finalized state forever. For example, if the controller of a pool is a smart contract, it can adjust the token weight and trading fees as many times as desired during the life cycle of the pool. That's what we call a smart pool.

Another possibility is to set the state of the pool to "finalized." After you do this, it's not possible anymore to change the parameters of the pool, and any address is able to trade with the pool; it becomes a public pool.

Before trading, we need liquidity. Adding liquidity works similarly to Uniswap. So you deposit tokens, and you get some BLP tokens. While your tokens are locked in the pool, you earn trading fees. When you provide liquidity, you can either send only one of the tokens in the pool or all of the tokens, but in this case, you have to respect the weight of the pool. Once you've got liquidity, you can trade. To trade, you provide a token of the pool as input and get another token of the pool as output.

The price is determined by a pricing formula for the pool, which is a generalization of the constant product formula of Uniswap.

## Smart Contract walkthrough 

### BFactory.sol 

This contract builds the new pool and log their addresses.

```solidity
import "./BPool.sol";
```

We need BPool contract to create a new pool from the factory contract.

```solidity
contract BFactory is BBronze {}
```

BFactory contract is inheriting BBronze contract which just returns the bytes32 of the "BRONZE". I don't exactly know why we're returning this yet.

```solidity 
event LOG_NEW_POOL(
        address indexed caller,
        address indexed pool
    );

    event LOG_BLABS(
        address indexed caller,
        address indexed blabs
    );
```

Events for logging when a new pool is created and old blab address and new blab address. 

```solidity
mapping(address=>bool) private _isBPool;
```

Mapping for whether the address is a pool or not.

```solidity
function isBPool(address b)
        external view returns (bool)
    {
        return _isBPool[b];
    }
```

This function returns whether that address is a pool or not.

```solidity
function newBPool()
        external
        returns (BPool)
    {}
```

This is the function for creating new pool from factory contract.

```solidity
BPool bpool = new BPool();
```

instantiate of BPool contract variable.Uniswap used interface to create pool but here we're instantiating a contract with the new keyword and then we'll create the pool.

```solidity
_isBPool[address(bpool)] = true;
```

We're setting that bpool variable address to true because that is the pool we just created.

```solidity
emit LOG_NEW_POOL(msg.sender, address(bpool));
```

We're emitting address that created the pool and the pool address.

```solidity
bpool.setController(msg.sender);
```

We're setting the controller to be the creater because controller has special rights regarding that pool.

```solidity
return bpool;
```

We're returning that pool.

```solidity
address private _blabs;
```

We're creating the _blabs private variable because we might decide later on to now use that wallet address. One can change wallet address.

```solidity
constructor() public {
        _blabs = msg.sender;
    }
```

We're setting the blabs address to the deployer in the constructor. That means Balancer's address will be the "_blabs".

```solidity
function getBLabs()
        external view
        returns (address)
    {
        return _blabs;
    }
```

Getter for that blabs variable.

```solidity
function setBLabs(address b)
        external
    {}
```

This function sets the blabs address. This might be giving right about that pool to new address.

```solidity
require(msg.sender == _blabs, "ERR_NOT_BLABS");
```

This require statement blows if the caller is not blabs address.

```solidity
emit LOG_BLABS(msg.sender, b);
```

emits the old and new blabs addresses.

```solidity
_blabs = b;
```

sets the new address to the blabs.

```solidity
function collect(BPool pool)
        external 
    {}
```

This function is for collecting rewards "developer fee". It will only be on by the governance.

```solidity
require(msg.sender == _blabs, "ERR_NOT_BLABS");
```

This line made me believe that the function is for collecting developer fees because initially "_blabs" is the deployer of the contract which is Balancer itself.

```solidity
uint collected = IERC20(pool).balanceOf(address(this));
```

This returns the balance of the liquidity pool token "BAL" that this contract is holding for the developer to collect developer fees.

```solidity
bool xfer = pool.transfer(_blabs, collected);
require(xfer, "ERR_ERC20_FAILED");
```

This transfer the collected balance from the contract to the _blabs address and for the transaction to execute successfully, xfer must be true.


