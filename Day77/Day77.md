## DEX Aggregator: Simplifying DeFi Trading

Imagine a DEX aggregator as the equivalent of a search engine for DeFi trading. Just as you would use platforms like "Expedia" or "Google Flights" to find the best flight deals instead of visiting each airline's website individually, a DEX aggregator helps you discover the optimal prices for your trades across different decentralized exchanges.

A DEX aggregator is a service that combines liquidity from various decentralized exchanges and market makers, streamlining the process of finding the most favorable prices for your trades.

The DeFi landscape is rich but fragmented, with liquidity spread across numerous sources. To ensure you get the best deal, you'd have to visit multiple trading venues. Some exchanges offer attractive rates but come with high slippage, while others provide lower slippage at the cost of less favorable rates. Aggregators solve this complexity by presenting all this data and liquidity in a user-friendly manner, enabling users to access the best prices.

**Two Types of Aggregators**

- Off-chain Aggregator (e.g., 1inch, Paraswap):

  - Pros: Can bridge multiple chains, offers flexibility.
  - Cons: Operator might front-run users.

- On-chain Aggregator (e.g., Swapswap):
  - Pros: Enables atomic routing and arbitrage.
  - Cons: Might not efficiently cover more than four exchanges.

## 1inch: Optimizing DeFi Trades

1inch serves as both a non-custodial decentralized exchange and a DEX aggregator. This platform connects various DEXs into a unified interface, allowing users to optimize their decentralized cryptocurrency trades without manually checking rates across each DEX. Smart contracts within 1inch source liquidity from multiple DEXs like Balancer, Kyber Network, Uniswap, and Mooniswap.

1inch supports over 250 coins and tokens, streamlining trading activities.

In simple terms, 1inch employs APIs to identify the best route for decentralized token swaps. This could involve multiple exchanges to achieve the most optimal trade execution. Consequently, users receive the best rates, with minimized slippage and reduced transaction cancellations.

Operating as a decentralized exchange, 1inch eliminates the need for an account or identity information. Users connect supported Ethereum wallets (e.g., MetaMask, WalletConnect, Ledger) to start using the platform instantly. As a non-custodial service, 1inch never gains full control over user assets; it executes trades with the user's permission.

## Smart Contract Walkthrough

**Aggregation Protocol V5**

The AggregationRouterV5 contract serves as a main aggregation router that incorporates various router contracts and mixins to provide functionalities for swaps and limit orders protocol.

```solidity
/// @notice Main contract incorporates a number of routers to perform swaps and limit orders protocol to fill limit orders
contract AggregationRouterV5 is EIP712("1inch Aggregation Router", "5"), Ownable,
    ClipperRouter, GenericRouter, UnoswapRouter, UnoswapV3Router, OrderMixin, OrderRFQMixin
{}
```

The AggregationRouterV5 contract integrates advanced features, including:

EIP712: Used for handling structured data and signing messages. It uniquely identifies the contract's domain using "1inch Aggregation Router" and version "5."

Ownable: Empowers the contract owner to control specific actions.

ClipperRouter, GenericRouter, UnoswapRouter, UnoswapV3Router: These components facilitate trade routing and interactions with different protocols and exchanges, optimizing execution.

OrderMixin, OrderRFQMixin: These handle various aspects of limit orders and RFQ (Request for Quote) orders, ensuring efficient execution and validation.

```solidity
using UniERC20 for IERC20;
```

This extends the functionality of ERC20 tokens.

```solidity
/**
 * @dev Sets the wrapped eth token and clipper exhange interface
 * Both values are immutable: they can only be set once during
 * construction.
 */
constructor(IWETH weth)
    UnoswapV3Router(weth)
    ClipperRouter(weth)
    OrderMixin(weth)
    OrderRFQMixin(weth)
{
    if (address(weth) == address(0)) revert ZeroAddress();
}
```

The constructor initializes the contract during deployment by setting up various router contracts and mixins. It takes an IWETH parameter (Wrapped Ether token) and passes it to the constructors of UnoswapV3Router, ClipperRouter, OrderMixin, and OrderRFQMixin. Additionally, it checks whether the provided Wrapped Ether token address is not zero; if it is, the constructor reverts execution with a ZeroAddress error.

```solidity
/**
 * @notice Retrieves funds accidentally sent directly to the contract address
 * @param token ERC20 token to retrieve
 * @param amount amount to retrieve
 */
function rescueFunds(IERC20 token, uint256 amount) external onlyOwner {
    token.uniTransfer(payable(msg.sender), amount);
}
```

It allows the contract owner to recover ERC20 funds that were mistakenly sent directly to the contract address. It uses the uniTransfer function to transfer the specified amount of tokens to the owner's address.

Let's explore the uniTransfer function.

```solidity
function uniTransfer(IERC20 token, address payable to, uint256 amount) internal {}
```

This function takes three parameters:

    token: An instance of the IERC20 interface representing the ERC20 token to be transferred.
    to: The address to which the transfer will be made.
    amount: The amount of tokens or Ether to be transferred.

```solidity
if (amount > 0) {}
```

It checks whether the amount parameter is greater than zero. If the amount is zero or negative, the function will not perform any transfer.

```solidity
if (isETH(token)) {}
```

It checks if the provided token is Ether

```solidity
if (address(this).balance < amount) revert InsufficientBalance();
```

It checks if the contract's current Ether balance (address(this).balance) is less than the specified amount.

```solidity
(bool success, ) = to.call{value: amount, gas: _RAW_CALL_GAS_LIMIT}("");
if (!success) revert ETHTransferFailed();
```

If the balance is sufficient and the token is Ether, this line initiates an external contract call to the to address using the call method. It transfers the specified amount of Ether and specifies the gas limit to be used for the call. If the call was not successful, it reverts execution.

```solidity
else {
    token.safeTransfer(to, amount);
}
```

If the provided token is not Ether, this block executes. It uses the safeTransfer function from the token contract (an ERC20 token) to transfer the specified amount of tokens to the to address.

It's important to note that the uniTransfer function does not revert for zero amounts being transferred. This is possibly intended behavior and could be handled in the user interface or by design.

Returning to the AggregationRouterV5 contract:

```solidity
/**
 * @notice Destroys the contract and sends eth to sender. Use with caution.
 * The only case when the use of the method is justified is if there is an exploit found.
 * And the damage from the exploit is greater than from just an urgent contract change.
 */
function destroy() external onlyOwner {
    selfdestruct(payable(msg.sender));
}
```

The destroy function allows the owner of the contract to destroy it and send any remaining Ether to the owner's address.

```solidity
function _receive() internal override(EthReceiver, OnlyWethReceiver) {
    EthReceiver._receive();
}
```

The \_receive function is internal and overrides two different functions: EthReceiver.\_receive and OnlyWethReceiver.\_receive. This function likely handles the reception of Ether in a specific manner, possibly related to the wrapped Ether (WETH) functionality.

Now, let's delve into the EthReceiver.\_receive function:

```solidity
function _receive() internal virtual {
    // solhint-disable-next-line avoid-tx-origin
    if (msg.sender == tx.origin) revert EthDepositRejected();
}
```

This function is designed to handle incoming Ether transfers and includes a security check. It verifies whether the transaction sender (msg.sender) is the original sender of the transaction (tx.origin). If the condition is met, meaning the transaction was initiated by an externally owned address (EOA), the function reverts with an EthDepositRejected error. This is done to prevent smart contracts from directly depositing Ether into the contract, helping to mitigate potential reentrancy or other attack vectors.
