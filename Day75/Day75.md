## Token Bonding Curves

Token Bonding Curves are a concept that introduces a dynamic pricing mechanism for tokens based on their changing supply. Unlike traditional fixed-price tokens, the price of tokens following a bonding curve model is determined by the available supply in the market. This pricing model offers flexibility and can be embedded with various incentives, governance rules, and financial mechanisms.

Token bonding curves fall under the token policy and financial incentives aspects of a project. Within the token policy, one can incorporate different incentives, governance rules, and policies into the curve. Financial incentives can also be integrated into the token bonding curve by embedding various incentive mechanisms.

If tokens are considered as non-fungible tokens (NFTs), additional aspects of property rights can be incorporated into the token bonding curve.

**What are token bonding curves?**

The concept of token bonding curves can be visualized as a mathematical curve or function on a graph. In a two-dimensional graph, the X-axis represents the total supply of tokens, while the Y-axis represents the corresponding price. The token bonding curve demonstrates how changes in supply affect the price. The curve represents the relationship between supply and price.

![linear](./Images/linear.png)

The pricing mechanism based on token bonding curves relies on a direct relationship between the token supply and its price. This approach disregards the influence of the broader market's supply and demand dynamics, focusing instead on the internal ecosystem of the token. By simplifying the pricing model to the number of tokens available in the system, speculation is reduced to some extent.

`Automated Market Making (AMM)` is one of the prominent use cases for token bonding curves. Instead of relying on takers and makers to provide liquidity, the liquidity provision is automated in the code based on the relationship between supply and price.

Alternatively, a different secondary market can be created using two types of curves: a buy curve and a sell curve.

![buySellCurve](./Images/buySellCurve.png)

This approach adds complexity and depends on the desired incentives. The design of the curves can be structured to discourage people from selling tokens until a specific point in time to prevent them from incurring losses, thus influencing behavior and incentivizing desired actions.

The versatility of token bonding curves allows them to be applied to various systems and designs. They can be utilized in different forms and can incorporate a wide range of governance and mechanism designs into the curve. This flexibility makes token bonding curves an interesting and exciting concept for token economics and ecosystem development.

## Properties of Bonding Curve

It operates based on following properties:

- Minting:

  Tokens can be minted at any time according to the price set by the smart contract. The minting process is automated and determined by the demand and liquidity added to the ecosystem. This means that no individual or entity has control over the minting process. The smart contract ensures that tokens are issued automatically based on predefined rules.

- Price increase:

  The price of the token increases as the token supply grows. This relationship reflects the economic value and network effects of the ecosystem. As more tokens are available in the market, it indicates a stronger network effect and increased economic value. This, in turn, leads to a higher monetary value for the ecosystem. The increase in token prices signifies growing confidence in the project, as more people buy and utilize the tokens.

- Reserve pool:

  The money collected from the purchase of tokens is stored in a smart contract as a reserve pool or pool balance. This reserve pool acts as collateral to support the value and liquidity of the tokens. Various projects use different mechanisms to determine where these collaterals are kept. The transparency of the smart contract ensures that the collateral is accounted for and provides value to the token pricing.

- Burning:

  Tokens can be burned or destroyed at any time. When tokens are burned, the person who initiated the burning process receives back a proportionate amount of the collateral from the reserve pool. The collateral received is determined by the underlying bonding curve mechanism, not by the initial amount of collateral invested. The bonding curve algorithm tracks and calculates the appropriate amount of collateral to be returned when tokens are burned. This approach reduces the potential for bribery, increases transparency, and maintains accountability for the token supply, pricing, and minting/burning schedule.

Overall, the token bonding curve provides a decentralized and automated mechanism for determining the supply, price, and liquidity of tokens within an ecosystem. It ensures that these factors are determined by the market demand and the actions of participants, rather than by centralized decision-making.
