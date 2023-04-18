## Carbon 
#### a decentralized protocol for asymmetric liquidity and trading

### Asymmetric Liquidity

In the DeFi, asymmetric liquidity is a term used to describe a situation where the liquidity of a particular asset in a trading pair is significantly higher than the liquidity of its counterpart in the same pair. This creates an imbalance in the liquidity of the trading pair, leading to a variety of potential risks and opportunities for traders.

Asymmetric liquidity can occur for a number of reasons. One common reason is the difference in popularity and demand for different tokens. For example, a popular token like Ethereum may have much higher liquidity than a less popular token like Dai. This can lead to asymmetric liquidity in any trading pairs that involve these tokens.

The main risk associated with asymmetric liquidity is the potential for price slippage. When one asset in a trading pair has significantly less liquidity than the other, it can be more difficult to execute trades without impacting the price of that asset. This can lead to a situation where traders are forced to accept a worse price for their trades than they would if the liquidity were more balanced.

On the other hand, asymmetric liquidity can also create opportunities for traders who are able to take advantage of the price inefficiencies that result from the imbalance. For example, a trader who is able to provide liquidity for the less liquid asset in a trading pair may be able to earn a higher return than they would otherwise, by taking advantage of the higher fees and potential price movements associated with the asymmetric liquidity.

In the DeFi space, asymmetric liquidity is often addressed through the use of automated market makers (AMMs), which allow traders to execute trades without relying on traditional order book mechanisms. Instead, AMMs use a mathematical formula to determine the price of assets in a trading pair based on the available liquidity. This can help to mitigate the risks associated with asymmetric liquidity, by ensuring that trades are executed at fair prices even when one asset in a pair has significantly less liquidity than the other.

### Overview 

In traditional liquidity provision, users typically provide liquidity to a single pool, or curve, that allows for trading in both directions. For example, if a user provides liquidity to an Ethereum-USDC pool, they can buy Ethereum with USDC or sell Ethereum for USDC using the same pool.

However, in Carbon, users provide liquidity to two separate curves, each of which only allows trading in one direction. For example, a user could provide liquidity to a curve that allows buying Ethereum with USDC, and another curve that allows selling Ethereum for USDC. This design gives users greater control over their trading preferences, as they can choose to provide liquidity to the curves that best align with their strategies.

One potential benefit of this approach is that it can allow for greater customization of trading strategies. By providing liquidity to separate curves, users can more precisely define their buy and sell orders, potentially leading to better execution and more profitable trades.But this approach also comes with some potential drawbacks. For example, the liquidity may be more fragmented across the two curves, making it more difficult to execute large trades without significant price impact. Additionally, it may be more complicated and time-consuming for users to manage their liquidity across multiple curves.

With this approach, users can execute their trading strategies using on-chain limit orders and range orders, which execute automatically and continuously without the need for external oracles or keepers.The key advantage of this approach is that it allows users to deploy premeditated trading strategies that execute automatically, without the need for constant monitoring or manual intervention. This can be particularly useful for traders who have a specific price range or set of conditions in mind for executing their trades.

For example, a user could deploy a trading strategy that consists of two orders: one that buys Ethereum (ETH) between the price range of 1200 and 1300 USDC, and another that sells ETH between the price range of 1500 and 1600 USDC. As the price of ETH crosses into the second order's defined range, the ETH accumulated from the first order becomes available for sale at the desired price. This approach allows traders to execute their trades automatically, without relying on external oracles or manual intervention.

The one-directional nature of asymmetric liquidity is beneficial for this approach because it allows users to define their trading strategies in a more precise and granular way. By providing liquidity to separate curves that each trade in one direction, users can more easily define the price ranges and conditions under which their orders should execute.

`if everybody sets the range for the price of the token, do people will actually swap based on the price that liquidity provider has set the price range?`

It is possible that if a large number of liquidity providers set the same price range for a token, it could lead to a scenario where the trading activity becomes concentrated within that range. This concentration could result in a narrower price range for the token as trades are executed within the defined range.

However, it's important to note that not all liquidity providers will set the same price range, and different trading strategies and preferences will result in a diverse set of liquidity providers with varying price ranges. Additionally, the liquidity providers who set price ranges that are outside the market consensus can potentially benefit from increased trading activity and arbitrage opportunities as the price of the token fluctuates.


Each order has a corresponding bonding curve that is unique to that order. The bonding curve's shape and liquidity concentration range are determined by specific parameters set by the liquidity provider. These parameters can be adjusted directly without needing to close and recreate the liquidity position. This means that liquidity providers can make changes to their orders more efficiently and with less gas consumption.

All of the individual orders and their corresponding curves are aggregated together to create on-chain liquidity for a given token pair. This means that the system pools the liquidity provided by all of the individual orders to create a larger pool of liquidity for trading activity.

To optimize liquidity utilization and trading efficiency, the system uses a routing engine that determines the optimal path for trades against the network. The routing engine considers factors such as the availability of liquidity, the fees charged by liquidity providers, and the overall market conditions to determine the most optimal path for a given trade.

### features of Carbon

- Asymmetric and Irreversible

    In traditional liquidity pools, users provide liquidity to a single curve that trades symmetrically in both directions, allowing users to buy or sell the asset at any time, depending on market conditions.

    However, in Carbon's liquidity pools, users provide liquidity to two separate curves that each trade in one direction. This means that users can create independent buy and sell orders that execute in a single direction and are irreversible once executed. This feature is desirable for users who wish to commit to a premeditated trading strategy and want to avoid the possibility of being affected by price fluctuations in the opposite direction.

    For example, a user can create a buy order that executes when the price of the asset falls below a certain threshold and a sell order that executes when the price of the asset rises above a certain threshold. Once these orders are executed, the user has committed to the strategy and cannot reverse or cancel the orders. This feature is beneficial for users who want to automate their trading strategies and remove emotions and biases from their trading decisions.
    
- Concentrated and Adjustable

    It allows users to set specific price ranges for their buy and sell orders. The price range is pre-defined, meaning that the user sets the price range at which they want to execute their trades. The liquidity pool concentrates liquidity within these price ranges, which means that trades are executed within the defined price range.

    This feature allows users to have greater control over their trading strategies, as they can precisely set the price ranges at which they want to buy or sell their assets. It also helps prevent slippage, as trades are executed only within the pre-defined price range.

    Furthermore, Carbon's order conditions can be adjusted on the fly without having to close and recreate the order, making it more gas-efficient to change order conditions. This means that if the user wants to modify the price range or other parameters of the order, they can do so without incurring additional gas fees associated with closing and recreating the order. This makes it more convenient and cost-effective for users to modify their trading strategies in real-time.
    
- Composable

    It enables users to create more complex trading strategies by linking multiple orders together. This feature allows users to save time and gas fees by reducing the number of transactions required to create a multi-order strategy.

    When a user creates a multi-order strategy, Carbon's system automatically shifts liquidity between linked orders as they are filled. For example, if a user creates two orders - one to buy ETH when its price is between 1200 and 1300 USDC and another to sell ETH when its price is between 1500 and 1600 USDC - Carbon's system will automatically shift the ETH acquired in the buy order to the sell order once the market moves into the specified price range.
    
- Re-usable

    Tokens acquired in one order become available for trading in a linked order once the market moves into range. This feature allows users to re-use tokens acquired in one order to execute trades in linked orders.
    
- MEV-Resistant

    MEV (Maximal Extractable Value) refers to the additional profits extracted from a transaction by exploiting the transaction's order execution. Carbon's trading system is designed to be resistant to a common form of MEV known as sandwich attacks. Sandwich attacks occur when a malicious actor exploits the transparency of blockchain transactions to front-run and back-run trades by inserting their own transactions between a user's transactions in a block. This allows the attacker to capture profits that would otherwise go to the user.

    Carbon's trading system is designed to minimize the impact of sandwich attacks by using asymmetric liquidity. With asymmetric liquidity, each user strategy is composed of independent buy and sell orders that trade in a single direction and are irreversible on execution. This means that orders can't be sandwiched, as there is no way for a malicious actor to insert a transaction between two orders that trade in the same direction. Additionally, the system is designed to minimize the impact of MEV on liquidity providers by reducing the incentive for miners to exploit order execution. This is accomplished by using a fee structure that rewards liquidity providers for providing liquidity and disincentivizes miners from attempting to extract additional profits from transactions.
    
DEXs, including the popular Uniswap v3, operate on the invariant-function DEX model, where exchange rates are determined by equations that force the composition of the liquidity pool to adhere to a predefined profile ("bonding curve"). Liquidity providers are beholden to the parameters of the liquidity pool, which limits their agency to execute precise trading strategies and set their own exchange rates.

The limitations of existing liquidity pools largely stem from the fact that each pool and its composite liquidity positions are governed by the same bonding curve in either direction. This means that the same curve is used for both buying and selling, and tokens sold by a liquidity pool may be repurchased by the pool at the same exchange rate. As a result, asymmetric trading strategies, which involve independent buy and sell patterns that are irreversible after being executed, remain largely unavailable on DEXs.

Uniswap v3, have devised workarounds to mimic limit orders by providing liquidity in a thin out-of-range price interval that is traded when prices go through the associated interval. However, in practice, such limit orders are reversed when markets retrace, requiring traders to constantly monitor the state of the system and remove their position immediately upon execution. In addition, offering liquidity in different ranges either requires separate liquidity positions or traders must automate costly transactions to close and recreate a liquidity position in their desired ranges as prices move.

The limitations of existing DEX infrastructure partly explain the volume and liquidity discrepancy between DeFi and CeFi, with most trading volume still occurring on centralized exchanges (CEXs) and a large percentage of that volume coming from automated strategies. The rise of MEV attacks in decentralized venues has further restricted liquidity from flowing into DeFi, emphasizing the need for DEX infrastructure to evolve and provide more sophisticated trading capabilities. Carbon aims to address these limitations.

`How does carbon keep track of the multiple bondage curve?`

Carbon uses a novel approach to keep track of multiple bonding curves, which is called the Carbon Curve Registry. The Carbon Curve Registry is a decentralized database that stores information about bonding curves used by different liquidity pools on the Carbon platform. Each bonding curve is represented by a unique curve ID, which is a cryptographic hash of the curve parameters.

When a user creates a new liquidity pool on Carbon, they define the parameters of the bonding curve they want to use. These parameters include the curve function, the reserve ratio, the fee structure, and the price ranges. Once the pool is created, Carbon computes the curve ID and registers it on the Carbon Curve Registry.

When a user wants to trade on a liquidity pool, Carbon retrieves the curve ID from the pool contract and uses it to look up the parameters of the bonding curve in the Carbon Curve Registry. Carbon then uses these parameters to compute the exchange rate and execute the trade.

The Carbon Curve Registry allows for the creation of custom bonding curves that can be used by multiple liquidity pools. This allows for greater flexibility and creativity in designing trading strategies. Moreover, it eliminates the need for each liquidity pool to store its own curve parameters, reducing the storage requirements of the system.

`In carbon due to constant product and sum, price is concentrated to certain price, how does carbon provides varies range price for the token?`

In Carbon, the bonding curves are not fixed, but rather they can be dynamically adjusted by liquidity providers to create different tick ranges and price ranges. This allows for a greater degree of flexibility in providing liquidity compared to other DEXs that use fixed bonding curves.

For example, suppose a user wants to provide liquidity for a token that currently has a price of 100 USDC per token. They can choose to provide liquidity at a tick range of ±1%, which means that their liquidity will be used for trades within a price range of 99 USDC to 101 USDC per token. Alternatively, they could choose a tick range of ±5%, which would allow for trades within a wider price range of 95 USDC to 105 USDC per token.

If the user chooses a tick range of ±1%, they would provide liquidity by depositing an equal value of the two tokens (e.g., 500 USDC and 5 tokens) into an escrow contract. The bonding curve would then adjust to create a liquidity pool that maintains a constant product of the two tokens (i.e., 500 USDC x 5 tokens = 2500 USDC-tokens).

As trades are executed within the tick range of ±1%, the bonding curve will adjust to maintain the constant product. For example, if a buyer purchases 10 tokens at a price of 101 USDC per token, the bonding curve will shift to a new price point that maintains the constant product of 2500 USDC-tokens. This could result in a new price of 99.01 USDC per token, which would create a new tick range of ±1% around that price.

