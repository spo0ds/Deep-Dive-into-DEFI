**Uniswap**

Uniswap is a protocol for the decentralized exchange of tokens on the Ethereum blockchain. It's deployed as a set of smart contracts and is completely decentralized, permissionless, and censorship-resistant. It is built on the concept of liquidity pools and AMM. The initial version of Uniswap was written in Vyper. In Uniswap V1, each liquidity pool had to include ETH as one of the currencies. For example, to trade from USDC to DAI, the user would have to trade the USDC for ETH and ETH for DAI, which usually results in higher gas fees and more slippage.

Uniswap launched Version 2 of the contract, offering ERC20-ERC20 liquidity pools. This is also preferable for liquidity providers who do not want to supply ETH and risk impermanent losses. V2 also had a few other features, including on-chain price feeds and flash swaps. All of the V2 smart contracts were written in Solidity. Due to its decentralized and permissionless nature, the first version of the protocol has been actively used alongside V2 for some time, regardless of the Uniswap team's encouragement to liquidity providers to migrate their liquidity to V2.

Sushiswap also came into play, which was aiming to directly compute with Uniswap by forking the project, adding a reward for Uniswap liquidity providers, and eventually stealing Uniswap liquidity onto the Sushiswap platform. This is also called a "Vampire Attack." The Sushiswap yield farming resulted in Uniswap liquidity going from around $300 million to almost $2 billion in a matter of days. Before the migration of Uniswap to Sushiswap started, there was still around $800 million worth of Uniswap liquidity staked on the Sushiswap platform. Although the Sushiswap migration resulted in Uniswap's total liquidity dropping from almost $2 billion to as low as $0.5 billion, the remaining liquidity was still higher than a couple of weeks earlier, which was $300 million before the Sushiswap project even started.

Later, Uniswap announced their Uni token. The most surprising part of the launch was how some of the tokens were retrospectively allocated. Everyone who had used Uniswap even once before the 1st of September was eligible to claim their 400 Uni tokens. The Uni tokens were distributed to around 50,000 Ethereum addresses, making them one of the most widely distributed tokens in the space. On top of that, the liquidity providers of the protocol were also retrospectively rewarded with extra Uni tokens. A total of 1 billion Uni tokens were allocated. 60% go to community members, 21.51% go to team members and future employees with a four-year vesting period, and 0.069% go to advisors with a four-year vesting period. After 4 years, there will be a perpetual inflation rate of 2% inflation per year to ensure continued participation and contribution to Uniswap at the expense of passive Uni holders.

On top of that, Uniswap announced incentives for liquidity pools that will reward liquidity providers with extra Uni tokens. Uni holders can also vote to add more incentivized pools after the initial 30-day governance grace period is over. By launching the token, the Uniswap team wanted to further decentralize the protocol, making it a publicly owned and self-sustaining financial infrastructure, while still continuing to protect its indestructible and autonomous qualities. The token holders will be able to participate in Uniswap governance by voting on different proposals or delegating their votes to a third party.


### High Level Working of Uniswap

**Creating markets using Uniswap Exchange Factories**

We've got a market maker who's going to create a market for us on Uniswap. The first thing that we have to do is use the `Uniswap Factory` smart contract. The market maker needs to create an exchange, so they call the `createExchange` function and pass in the address of the token that they want to register the exchange for.

```vyper
# Uniswap Factory
@public
def createExchange(token: address) -> address:
```

It basically calls the instantiation function like in OOP, which takes the exchange template and creates a new actual Uniswap exchange at an exchange address, which is an address on the Ethereum blockchain.

```vyper
# Uniswap Factory
exchange: address = create_with_code_of(self.exchangeTemplate)
```

It calls the setup method with the token address, which essentially creates two empty pools.

```vyper
# Uniswap Factory
Exchange(exchange).setup(token)
```

a token pool and an Ethereum pool, both of which are empty at the start of this exchange. Meanwhile, on the Uniswap factory, the exchange address is registered in a `token_to_exchange` mapping where the token address is mapped to the exchange address.

```vyper
self.token_to_exchange[token] = exchange
```

This allows us to go forward and get hold of the exchange by just passing in the token address. We can also get the exchange via the exchange address, and it'll return the token address.

```vyper
def getToken(exchange: address) -> address:
    return self.exchange_to_token[exchange]
```

So there's actually a reverse registration of this and an exchange for token address registration.

At this point, we've got our new exchange, but it's empty. We need to add liquidity.

**Adding liquidity to markets via Uniswap Exchanges**

Liquidity providers provide liquidity to the exchange. It's highly likely that the liquidity provider is the same person who created the exchange. He can call one of the new exchanges he has created, which in our example is Eth:DAI, which is located at `x` address. The liquidity provider interacts with the smart contract by calling the `addLiquidity` function and passing in the number of tokens that they want to add to the pool, and they also send some Ethereum using "msg.value."

```vyper
# Uniswap Exchange
@payable
def addLiquidity(min_liquidity: uint256, max_tokens: uint256, deadline: timestamp) -> uint256:
```

For example, this liquidity provider provides 100 ETH and 25,000 DAI, which is the end result after calling the method. This now sets the price for Eth:DAI. Basically, the liquidity pool ratio between Ethereum and the token is determined by the amount of liquidity, and so with this example, we're essentially setting the price of one ether to 250 DAI. If the ratio doesn't match the current market rates, this creates an arbitrage opportunity. As soon as an arbitrage opportunity is created, traders take advantage of it, and by taking advantage of it, the prices start to shift in the right direction, which eventually causes the exchanges to have a price that is near or as near as possible to the actual exchange. Once we have liquidity in our pool, we can start swapping tokens around.
