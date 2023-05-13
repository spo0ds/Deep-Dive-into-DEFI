```solidity
function updateStrategy(
        uint256 strategyId,
        Order[2] calldata currentOrders,
        Order[2] calldata newOrders
    ) external payable nonReentrant whenNotPaused onlyProxyDelegate {}
```

    strategyId represents the ID of the strategy being updated
    currentOrders represents the current orders of the strategy pair
    newOrders represents the new orders that will replace the current orders
    
```solidity
Pair memory strategyPair = _pairById(_pairIdByStrategyId(strategyId));
```

This retrieves the Pair struct associated with the given strategyId.

```solidity
if (msg.sender != _voucher.ownerOf(strategyId)) {
            revert AccessDenied();
        }
```

It checks if the msg.sender is not the owner of the strategy with the given strategyId.

```solidity
if (msg.value > 0 && !strategyPair.tokens[0].isNative() && !strategyPair.tokens[1].isNative()) {
            revert UnnecessaryNativeTokenReceived();
        }
```

It checks if the value of msg.value is greater than zero and if neither of the tokens in the strategyPair is a native token.

```solidity
_validateOrders(newOrders);
```

This function checks whether the orders in newOrders are valid or not.

```solidity
_updateStrategy(strategyId, currentOrders, newOrders, strategyPair, msg.sender, msg.value);
```

This function will perform the actual update of the strategy by executing the necessary trades.

```solidity
function _updateStrategy(
    uint256 strategyId,
    Order[2] calldata currentOrders,
    Order[2] calldata newOrders,
    Pair memory pair,
    address owner,
    uint256 value
) internal {}
```

    strategyId: the ID of the strategy to be updated
    currentOrders: the current orders for the strategy
    newOrders: the new orders for the strategy
    pair: a memory struct representing the pair of tokens for the strategy
    owner: the owner of the strategy
    value: the value of ETH sent with the transaction (if any)
    
```solidity
uint256[3] storage packedOrders=_packedOrdersByStrategyId[strategyId];
```

It is a storage array of packed orders (in uint256 format) for the strategy.

```solidity
uint256[3] memory packedOrdersMemory = _packedOrdersByStrategyId[strategyId];
```

It is a memory copy of the packedOrders array.

```solidity
(Order[2] memory orders, bool ordersInverted) = _unpackOrders(packedOrdersMemory);
```

ordersInverted indicates whether the token order in orders is inverted from the token order in pair.

If ordersInverted is true, then the tokens need to be sorted in reverse order before depositing or withdrawing liquidity. For example, if the pair is ETH/DAI and the stored packed orders are DAI/ETH, then ordersInverted will be true and the tokens need to be sorted as ETH/DAI. Conversely, if the pair is ETH/DAI and the stored packed orders are ETH/DAI, then ordersInverted will be false and the tokens need to be sorted as DAI/ETH. This is important to ensure that liquidity is deposited and withdrawn correctly.

```solidity
if (!_equalStrategyOrders(currentOrders, orders)) {
    revert OutDated();
}
```

A check to ensure that the current orders for the strategy match the orders stored in the contract.

```solidity
uint256[3] memory newPackedOrders = _packOrders(newOrders, ordersInverted);
```

A new array of packed orders (in uint256 format) created from newOrders and ordersInverted.

It is created to represent the new liquidity positions for a strategy in a single 256-bit storage slot, which can be stored on the blockchain. It is necessary to create this packed representation of the orders to efficiently store and update liquidity positions on the blockchain.

```solidity
for (uint256 n = 0; n < 3; n = uncheckedInc(n)) {
    if (packedOrdersMemory[n] != newPackedOrders[n]) {
        packedOrders[n] = newPackedOrders[n];
    }
}
```

The for loop compares each element of packedOrdersMemory with the corresponding element of newPackedOrders. If the elements do not match, the new value is stored in packedOrders.

The reason for this is that packedOrders is a storage variable, which means it is stored permanently on the blockchain and is accessible across different transactions. Therefore, in order to update the orders for a given strategy, the packed orders stored in packedOrders need to be updated as well.

```solidity
Token[2] memory sortedTokens = _sortStrategyTokens(pair, ordersInverted);
```

We sort the tokens in the pair according to their balances in the strategy, which will be used to determine whether to deposit or withdraw tokens from the strategy.

```solidity
for (uint256 i = 0; i < 2; i = uncheckedInc(i)) {
    Token token = sortedTokens[i];
```

This loop iterates through the sorted tokens in the pair and performs deposit and withdrawal operations for each one.

```solidity
if (newOrders[i].y < orders[i].y) {
    // liquidity decreased - withdraw the difference
    uint128 delta = orders[i].y - newOrders[i].y;
    _withdrawFunds(token, payable(owner), delta);
}
```

If the new order for the current token has a lower balance than the current order, the difference is withdrawn from the strategy by calling the _withdrawFunds function.

```solidity
else if (newOrders[i].y > orders[i].y) {
    // liquidity increased - deposit the difference
    uint128 delta = newOrders[i].y - orders[i].y;
    _validateDepositAndRefundExcessNativeToken(token, owner, delta, value);
}
```

If the new order for the current token has a higher balance than the current order, the difference is deposited into the strategy by calling the _validateDepositAndRefundExcessNativeToken function.

```solidity
if (value > 0 && token.isNative() && newOrders[i].y <= orders[i].y) {
    payable(address(owner)).sendValue(value);
}
```

If the current token is the native token (e.g., Ether), and the new balance for the token is less than or equal to the current balance, the excess native token value is refunded to the owner by calling sendValue.

```solidity
       // emit event
        emit StrategyUpdated({
            id: strategyId,
            token0: sortedTokens[0],
            token1: sortedTokens[1],
            order0: newOrders[0],
            order1: newOrders[1],
            reason: STRATEGY_UPDATE_REASON_EDIT
        });
```

The StrategyUpdated event is emitted after the strategy has been successfully updated in the _createStrategy function. This event includes the following parameters:

    id: The ID of the updated strategy. This is passed in as a parameter to the function.
    token0: The first token in the sorted token pair. This is obtained by calling the _sortStrategyTokens function with the pair and ordersInverted parameters.
    token1: The second token in the sorted token pair. This is obtained by calling the _sortStrategyTokens function with the pair and ordersInverted parameters.
    order0: The new order for the first token. This is passed in as a parameter to the function.
    order1: The new order for the second token. This is passed in as a parameter to the function.
    reason: The reason for the strategy update. In this case, it is set to STRATEGY_UPDATE_REASON_EDIT.
    


