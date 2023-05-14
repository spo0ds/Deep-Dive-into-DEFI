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
    

```solidity
function deleteStrategy(uint256 strategyId) external nonReentrant whenNotPaused onlyProxyDelegate {}
```

This function allows the owner of a strategy to delete it. 

```solidity
Pair memory strategyPair = _pairById(_pairIdByStrategyId(strategyId));
```

It finds the strategyPair associated with the given strategyId by calling the internal _pairIdByStrategyId function to get the ID of the pair, and then it calls the _pairById function to retrieve the actual pair data. If the strategyId is invalid, the _pairIdByStrategyId function will revert.

```solidity
if (msg.sender != _voucher.ownerOf(strategyId)) {
            revert AccessDenied();
        }
```

It checks that the caller is the owner of the strategy associated with the given strategyId.

```solidity
_deleteStrategy(strategyId, _voucher, strategyPair);
```

This function will remove the strategy from the _strategies mapping and from the _pairIdByStrategyId mapping, and will emit a StrategyDeleted event.

```solidity
function _deleteStrategy(uint256 strategyId, IVoucher voucher, Pair memory pair) internal {}
```

This function takes three parameters: strategyId, voucher, and pair. strategyId is the ID of the strategy that will be deleted. voucher is an instance of the IVoucher contract that manages the Carbon Voucher NFTs. pair is a Pair struct that represents the pair of tokens associated with the strategy.

```solidity
Strategy memory strategy = _strategy(strategyId, voucher, pair);
```

This line calls the _strategy function (explained earlier) to retrieve the Strategy struct associated with the specified strategyId. The retrieved Strategy struct is stored in memory.

```solidity
voucher.burn(strategy.id);
```

It burns the Carbon Voucher NFT associated with the strategy.

```solidity
delete _packedOrdersByStrategyId[strategy.id];
```

It clears the storage associated with the strategy's packed orders by deleting the mapping in _packedOrdersByStrategyId that uses the strategy ID as a key.

```solidity
_strategyIdsByPairIdStorage[pair.id].remove(strategy.id);
```

It removes the strategyId from the list of strategies associated with the pair in _strategyIdsByPairIdStorage.

```solidity
_withdrawFunds(strategy.tokens[0], payable(strategy.owner), strategy.orders[0].y);
_withdrawFunds(strategy.tokens[1], payable(strategy.owner), strategy.orders[1].y);
```

These lines call the _withdrawFunds function (explained earlier) to withdraw the funds associated with each token in the strategy and transfer them to the strategy's owner.

```solidity
emit StrategyDeleted({
    id: strategy.id,
    owner: strategy.owner,
    token0: strategy.tokens[0],
    token1: strategy.tokens[1],
    order0: strategy.orders[0],
    order1: strategy.orders[1]
});
```

Finally, this line emits a StrategyDeleted event, which includes the id of the deleted strategy, the owner of the strategy, the two tokens involved in the strategy, and the order amounts for each token.

```solidity
function strategiesByPair(
        Token token0,
        Token token1,
        uint256 startIndex,
        uint256 endIndex
    ) external view returns (Strategy[] memory) {}
```

It takes in two Token objects (token0 and token1), the start and end indices of the strategy array, and returns an array of Strategy structs.

```solidity
_validateInputTokens(token0, token1);
```

It verifies that both input tokens are valid ERC20 tokens.

```solidity
Pair memory strategyPair = _pair(token0, token1);
```

The _pair internal function is called to retrieve the Pair struct for the input token pair. The Pair struct is used to retrieve all the strategies associated with the pair.

```solidity
return _strategiesByPair(strategyPair, startIndex, endIndex, _voucher);
```

 The _strategiesByPair function retrieves all the strategies associated with the given pair and returns the array of Strategy structs within the specified index range.
 
 ```solidity
 function _strategiesByPair(
        Pair memory pair,
        uint256 startIndex,
        uint256 endIndex,
        IVoucher voucher
    ) internal view returns (Strategy[] memory) {}
```

This is used to return the stored strategies of a pair, given the pair and a range of indices.

The function takes four parameters:

    pair - a Pair struct that represents the pair of tokens for which to return the stored strategies.
    startIndex - the index of the first strategy to return.
    endIndex - the index of the last strategy to return.
    voucher - an instance of the IVoucher interface used to interact with the Bancor Voucher contract.
    
It returns an array of Strategy structs.

```solidity
        EnumerableSetUpgradeable.UintSet storage strategyIds = _strategyIdsByPairIdStorage[pair.id];
        uint256 allLength = strategyIds.length();
```

It initializes a variable called strategyIds as a UintSet data structure from the EnumerableSetUpgradeable library. UintSet is a set data structure that contains unique unsigned integers, and the EnumerableSetUpgradeable library provides set functionalities that allow iterating over set elements.

Why is it initialized using EnumerableSetUpgradeable?

```
It allows efficient storage and management of a set of unsigned integers representing the IDs of the strategies that are associated with a given token pair.

This data structure is used to keep track of all the strategy IDs associated with a given token pair. By using a UintSet, the code can easily add, remove, and check for the existence of a strategy ID within the set. This is a more efficient and optimized way of storing the strategy IDs compared to using an array or a mapping.

Furthermore, the EnumerableSetUpgradeable library provides a set of functions that enable easy iteration over the elements in the set, such as the length() function used in the code to determine the number of strategies stored in the strategyIds set. This makes it easier to perform operations on all the strategies associated with a given token pair.

```

The strategyIds set is created by accessing the _strategyIdsByPairIdStorage mapping with the pair.id as the key. _strategyIdsByPairIdStorage is a storage mapping that maps each pair.id to a set of strategy IDs that are associated with the pair.

The next line of code initializes a variable called allLength as the length of the strategyIds set. This variable is used later in the function to check if the provided endIndex is out of bounds or not.

```solidity
        if (endIndex == 0 || endIndex > allLength) {
            endIndex = allLength;
        }
```

If the endIndex is zero or greater than the total length of the set, it is set to the total length of the set.

`Why ?`

```
If the endIndex parameter is greater than the total length of the set, then it means that the function will be trying to access elements beyond the set's range, which could result in an error. Therefore, to prevent this error from occurring, the endIndex parameter is set to the total length of the set. This ensures that the function will only return valid elements that exist in the set and prevent out of bound errors. Setting endIndex to zero would result in an empty return array, which is not desirable if the goal is to get all the elements in the set.
```

```solidity
        if (startIndex > endIndex) {
            revert InvalidIndices();
        }
```

If the startIndex is greater than the endIndex, the function reverts with an error message.

The startIndex represents the beginning of the range, and the endIndex represents the end of the range, so it makes no sense to have a range where the end is before the beginning.

```solidity
        uint256 resultLength = endIndex - startIndex;
        Strategy[] memory result = new Strategy[](resultLength);
```

It calculates the length of the resulting array and creates a new array with that length because it needs to return an array of strategies within the given range of indices. 

```solidity
        for (uint256 i = 0; i < resultLength; i = uncheckedInc(i)) {
            uint256 strategyId = strategyIds.at(startIndex + i);
            result[i] = _strategy(strategyId, voucher, pair);
        }
```

It then iterates through the range of indices, retrieves the corresponding strategy ID from the set of strategy IDs, and calls the _strategy function to retrieve the corresponding Strategy struct. The retrieved Strategy struct is then added to the resulting array and then returns the resulting array of Strategy structs.


```solidity
    function strategiesByPairCount(Token token0, Token token1) external view returns (uint256) {
```

 It returns the number of stored strategies associated with a given token pair, represented by Token instances token0 and token1.
 
 ```solidity
         _validateInputTokens(token0, token1);
```

It ensures that the input tokens are valid.

```solidity
        Pair memory strategyPair = _pair(token0, token1);
```

It creates a new Pair object called strategyPair by calling the internal _pair function with the input tokens token0 and token1.

```solidity
        return _strategiesByPairCount(strategyPair);
```

This function calculates the number of strategies stored for the given pair and returns it as a uint256.

```solidity
function _strategiesByPairCount(Pair memory pair) internal view returns (uint256) {}
```

It takes Pair object as an input parameter. It returns a uint256 that represents the count of stored strategies for the given pair.

```solidity
EnumerableSetUpgradeable.UintSet storage strategyIds = _strategyIdsByPairIdStorage[pair.id];
```

Here, we create a new UintSet data structure from the EnumerableSetUpgradeable library, and store it in the strategyIds variable. This set will hold the strategy IDs of all the strategies associated with the given pair. We obtain this set by looking up the pair.id value in the _strategyIdsByPairIdStorage mapping, which returns an existing set of strategy IDs for the given pair.id.

```solidity
return strategyIds.length();
```

Finally, we return the length of the strategyIds set, which represents the count of stored strategies for the given pair.

```solidity
function tradeBySourceAmount(
    Token sourceToken,
    Token targetToken,
    TradeAction[] calldata tradeActions,
    uint256 deadline,
    uint128 minReturn
) external payable nonReentrant whenNotPaused onlyProxyDelegate returns (uint128) {}
```

The function is used to trade a specified amount of a given source token for an unknown amount of a target token, using a series of trade actions defined by the caller. The function returns the amount of the target token that was received from the trade.

The function takes several parameters:

    `sourceToken` and `targetToken` are the ERC20 tokens being traded.
    `tradeActions` is an array of trade actions that define the trade route. A trade action is a struct that specifies a Uniswap pair, an action type (either swap or join), and the amount of input token and output token to use in the trade. The array of trade actions defines the sequence of swaps or joins to execute to trade the source token for the target token.
    `deadline` is a timestamp after which the trade can no longer be executed.
    `minReturn` is the minimum amount of target token that the caller expects to receive from the trade.

```solidity
_validateTradeParams(sourceToken, targetToken, deadline, msg.value, minReturn);
```

It checks that the trade parameters are valid.This is a necessary step to make sure that the trade can be executed safely.

```solidity
Pair memory _pair = _pair(sourceToken, targetToken);
```

After validating the input parameters, a Pair memory variable _pair is created by calling the _pair function of the contract. This function takes the source and target tokens as arguments and returns a Pair object that represents the trading pair.

```solidity
TradeParams memory params = TradeParams({
    trader: msg.sender,
    tokens: TradeTokens({ source: sourceToken, target: targetToken }),
    byTargetAmount: false,
    constraint: minReturn,
    txValue: msg.value,
    pair: _pair
});
```

A TradeParams memory variable params is created, which is a struct containing the following fields:

    trader: the address of the trader who is executing the trade.
    tokens: a TradeTokens struct, which contains the source and target tokens of the trade.
    byTargetAmount: a boolean flag that indicates whether the trade should be executed based on the target token amount or the source token amount.
    constraint: the minimum amount of target token that must be returned to the trader.
    txValue: the value of the transaction.
    pair: the Pair object representing the trading pair.
    
```solidity
SourceAndTargetAmounts memory amounts = _trade(tradeActions, params);
return amounts.targetAmount;
```

Finally, the function calls the _trade internal function with the trade actions and trade parameters to execute the trades. The _trade function executes each trade action in sequence, using the input and output amounts from the previous trade action as the input and output amounts for the current trade action. Once all the trade actions are executed, the function returns the amount of the target token received from the last trade action.

```solidity
function _trade(
    TradeAction[] calldata tradeActions,
    TradeParams memory params
) internal returns (SourceAndTargetAmounts memory totals) {}
```

The function takes an array of TradeAction objects and a TradeParams object as input parameters, and returns a SourceAndTargetAmounts object as output.

```solidity
bool isTargetToken0 = params.tokens.target == params.pair.tokens[0];
```

It initializes a boolean variable isTargetToken0 with the result of the comparison of the target token of the TradeParams object with the first token of the Pair object. This is used to determine the order in which orders are processed in the loop below.

```solidity
// process trade actions
for (uint256 i = 0; i < tradeActions.length; i = uncheckedInc(i)) {}
```

The for loop that iterates over the TradeAction objects in the tradeActions array.

```solidity
// prepare variables
uint256 strategyId = tradeActions[i].strategyId;
uint256[3] storage packedOrders = _packedOrdersByStrategyId[strategyId];
uint256[3] memory packedOrdersMemory = _packedOrdersByStrategyId[strategyId];
(Order[2] memory orders, bool ordersInverted) = _unpackOrders(packedOrdersMemory);
```

 It initializes a strategyId variable with the strategyId of the current TradeAction, retrieves the packed orders for the strategy from storage, and unpacks them into a tuple of Order objects and a boolean flag indicating whether the orders need to be inverted.
 
 ```solidity
 _validateTradeParams(params.pair.id, strategyId, tradeActions[i].amount);
```

A function to validate the trade parameters, including the trade amount, against the current pair and strategy.

```solidity
(Order memory targetOrder, Order memory sourceOrder) = isTargetToken0 == ordersInverted
    ? (orders[1], orders[0])
    : (orders[0], orders[1]);
```

It unpacks the Order objects for the target and source tokens, depending on the value of isTargetToken0 and ordersInverted.

To ensure that the correct Order objects are being updated with the results of the trade.

The isTargetToken0 variable indicates whether the target token is the first or second token in the trading pair, and ordersInverted indicates whether the order of the tokens in the trading pair has been inverted. By checking these values, the code determines which of the two Order objects in the orders array corresponds to the target token and which corresponds to the source token.

The targetOrder and sourceOrder variables are then assigned the appropriate Order objects based on these checks, and the values of the orders are updated accordingly. By doing this, the code ensures that the trade is executed correctly and the orders are updated in the correct way.

```solidity
// calculate the orders new values
SourceAndTargetAmounts memory tempTradeAmounts = _singleTradeActionSourceAndTargetAmounts(
    targetOrder,
    tradeActions[i].amount,
    params.byTargetAmount
);
```

It calculates the new values for the orders based on the current TradeAction and updates the tempTradeAmounts struct with the source and target amounts.

```solidity
// handled specifically for a custom error message
if (targetOrder.y < tempTradeAmounts.targetAmount) {
    revert InsufficientLiquidity();
}
```

It checks if there is enough liquidity in the target order to fulfill the trade, and reverts with a custom error message if there is not.In other words, if targetOrder.y is less than tempTradeAmounts.targetAmount, then the trade cannot be executed without affecting the integrity of the Uniswap pool.

```solidity
// update the orders with the new values
unchecked {
    targetOrder.y -= tempTradeAmounts.targetAmount;
}

sourceOrder.y += tempTradeAmounts.sourceAmount;
if (sourceOrder.z < sourceOrder.y) {
    sourceOrder.z = sourceOrder.y;
}
```

The first line reduces the amount of the target token in the target order by the amount of the target token that was sold in the trade.

The second line increases the amount of the source token in the source order by the amount of the source token that was sold in the trade.

The third line updates the maximum amount of the source token that can be sold in the source order to be equal to the new amount of the source token.

```solidity
// store new values if necessary
uint256[3] memory newPackedOrders = _packOrders(orders, ordersInverted);
for (uint256 n = 0; n < 3; n = uncheckedInc(n)) {
    if (packedOrdersMemory[n] != newPackedOrders[n]) {
        packedOrders[n] = newPackedOrders[n];
    }
}
```

It stores the new order values if they have changed since the last trade. It creates a new array newPackedOrders that contains the packed order values and compares each value with the corresponding value in the previous packedOrdersMemory array. If the value has changed, it updates the packedOrders array with the new value.

```solidity
// emit update events if necessary
Token[2] memory sortedTokens = _sortStrategyTokens(params.pair, ordersInverted);
emit StrategyUpdated({
    id: strategyId,
    token0: sortedTokens[0],
    token1: sortedTokens[1],
    order0: orders[0],
    order1: orders[1],
    reason: STRATEGY_UPDATE_REASON_TRADE
});
```

 It sorts the two tokens in the strategy and emits the updated values of the two orders, along with the reason for the update.
 
 ```solidity
totals.sourceAmount += tempTradeAmounts.sourceAmount;
totals.targetAmount += tempTradeAmounts.targetAmount;
```

It updates the totals struct with the amounts of tokens that were traded.

```solidity
// apply trading fee
uint128 tradingFeeAmount;
Token tradingFeeToken;
if (params.byTargetAmount) {
    uint128 amountIncludingFee = _addFee(totals.sourceAmount);
    tradingFeeAmount = amountIncludingFee - totals.sourceAmount;
    tradingFeeToken = params.tokens.source;
    totals.sourceAmount = amountIncludingFee;
    if (totals.sourceAmount > params.constraint) {
        revert GreaterThanMaxInput();
    }
} else {
    uint128 amountExcludingFee = _subtractFee(totals.targetAmount);
    tradingFeeAmount = totals.targetAmount - amountExcludingFee;
    tradingFeeToken = params.tokens.target;
    totals.targetAmount = amountExcludingFee;
    if (totals.targetAmount < params.constraint) {
        revert LowerThanMinReturn();
    }
}
```

It applies a trading fee to the traded tokens. If params.byTargetAmount is true, the fee is applied to the source token and the source amount is updated to include the fee. If params.byTargetAmount is false, the fee is applied to the target token and the target amount is updated to exclude the fee. The code also checks if the updated amount of the source token is greater than the maximum input.

```solidity
// transfer funds
_validateDepositAndRefundExcessNativeToken(
    params.tokens.source,
    params.trader,
    totals.sourceAmount,
    params.txValue
);
```

_validateDepositAndRefundExcessNativeToken function to ensure that the trader has deposited enough tokens to cover the trade and that any excess tokens are refunded back to the trader.The function takes four arguments: the source token, the trader's address, the amount of source tokens to transfer, and the transaction value.

```solidity
_withdrawFunds(params.tokens.target, payable(params.trader), totals.targetAmount);
```

It transfers the target tokens from the protocol to the trader's address using the _withdrawFunds function. The function takes three arguments: the target token, the trader's address (converted to a payable address), and the amount of target tokens to transfer.

```solidity
// update fee counters
_accumulatedFees[tradingFeeToken] += tradingFeeAmount;
```

It updates the accumulated fees for the tradingFeeToken by adding the tradingFeeAmount to the corresponding element in the _accumulatedFees mapping.

```solidity
// tokens traded successfully, emit event
emit TokensTraded({
    trader: params.trader,
    sourceToken: params.tokens.source,
    targetToken: params.tokens.target,
    sourceAmount: totals.sourceAmount,
    targetAmount: totals.targetAmount,
    tradingFeeAmount: tradingFeeAmount,
    byTargetAmount: params.byTargetAmount
});
```

It emits the TokensTraded event, which includes information about the trade such as the trader's address, the source and target tokens, the amounts traded, the trading fee amount, and whether the trade was executed using the target or source amount.

```solidity
return totals;
```

It returns the totals object containing the amounts of tokens traded.

```
If params.byTargetAmount is true, it means the trade is being made by specifying the target token amount and the source token amount is being calculated based on the current prices. In this case, the fee is added to the source amount before the trade is executed, since the source amount is what is being determined by the trade.

If params.byTargetAmount is false, it means the trade is being made by specifying the source token amount and the target token amount is being calculated based on the current prices. In this case, the fee is subtracted from the target amount after the trade is executed, since the target amount is what is being determined by the trade.

If params.byTargetAmount is true:

    amountIncludingFee = (1 - fee) * totals.sourceAmount
    tradingFeeAmount = fee * totals.sourceAmount
    totals.sourceAmount = amountIncludingFee

If params.byTargetAmount is false:

    amountExcludingFee = totals.targetAmount / (1 - fee)
    tradingFeeAmount = fee * amountExcludingFee
    totals.targetAmount = amountExcludingFee

where fee is the trading fee percentage applied to the trade.
```

```solidity
function tradeByTargetAmount(
    Token sourceToken,
    Token targetToken,
    TradeAction[] calldata tradeActions,
    uint256 deadline,
    uint128 maxInput
) external payable nonReentrant whenNotPaused onlyProxyDelegate returns (uint128) {}
```

This function is called when a user wants to trade sourceToken for targetToken and specify the exact amount of targetToken they want to receive in return (byTargetAmount = true). The function takes in the following arguments:

    sourceToken: The address of the ERC20 token the user wants to sell.
    targetToken: The address of the ERC20 token the user wants to buy.
    tradeActions: An array of TradeAction structs that describe how to execute the trade.
    deadline: The timestamp after which the trade cannot be executed.
    maxInput: The maximum amount of sourceToken the user is willing to sell.
    
```solidity
    _validateTradeParams(sourceToken, targetToken, deadline, msg.value, maxInput);
```

This function call validates the trade parameters and ensures that the input values are valid.

```solidity
    if (sourceToken.isNative()) {
        // tx's value should at least match the maxInput
        if (msg.value < maxInput) {
            revert InsufficientNativeTokenReceived();
        }
    }
```

If the sourceToken is the native token (ETH), this code block checks that the transaction value is at least equal to the maximum input amount specified by the user.

```solidity
    Pair memory _pair = _pair(sourceToken, targetToken);
```

It creates a new Pair struct using the _pair function, which takes in the sourceToken and targetToken and returns a Pair struct that represents the Bancor pool for those tokens.

```solidity
    TradeParams memory params = TradeParams({
        trader: msg.sender,
        tokens: TradeTokens({ source: sourceToken, target: targetToken }),
        byTargetAmount: true,
        constraint: maxInput,
        txValue: msg.value,
        pair: _pair
    });
```

This line creates a new TradeParams struct that contains all the relevant information for the trade.

It is calling the _trade function with the specified tradeActions and params parameters and storing the result in the amounts variable. The _trade function processes the trades according to the tradeActions and params parameters and returns the amounts of source and target tokens involved in the trade as a SourceAndTargetAmounts struct.

After the trade is completed, the function returns the amount of source token that was traded, which is stored in the amounts.sourceAmount variable. The reason for returning only the source amount is because this function is used to perform trades by specifying the target amount, so the source amount is the input to the trade and the target amount is the output.

```solidity
function calculateTradeSourceAmount(Token sourceToken, Token targetToken, TradeAction[] calldata tradeActions) external view returns (uint128) {}
```

It is used to calculate the source token amount required to execute a trade. It takes in three parameters:

    sourceToken: The source token to trade.
    targetToken: The target token to trade.
    tradeActions: An array of TradeAction structs that represent the trade actions to perform.
    
```solidity
_validateInputTokens(sourceToken, targetToken);
```

It validates that the input tokens are not empty and are not equal to each other.

```solidity
Pair memory strategyPair = _pair(sourceToken, targetToken);
```

It creates a Pair memory object called strategyPair by calling the _pair function with the sourceToken and targetToken arguments. The _pair function returns the Pair object that represents the liquidity pool that is used to trade the two tokens.

```solidity
TradeTokens memory tokens = TradeTokens({ source: sourceToken, target: targetToken });
```

It creates a TradeTokens memory object called tokens by initializing it with the sourceToken and targetToken arguments.

```solidity
SourceAndTargetAmounts memory amounts = _tradeSourceAndTargetAmounts(tokens, tradeActions, strategyPair, true);
```

It calls the _tradeSourceAndTargetAmounts function with the tokens, tradeActions, strategyPair, and true arguments to calculate the source and target amounts based on the trade actions that are passed in. The amounts variable stores the result.

```solidity
return amounts.sourceAmount;
```

It returns the sourceAmount value of the amounts variable as the output of the calculateTradeSourceAmount function.

```solidity
/**
     * @inheritdoc ICarbonController
     */
    function calculateTradeTargetAmount(
        Token sourceToken,
        Token targetToken,
        TradeAction[] calldata tradeActions
    ) external view returns (uint128) {
        _validateInputTokens(sourceToken, targetToken);
        Pair memory strategyPair = _pair(sourceToken, targetToken);
        TradeTokens memory tokens = TradeTokens({ source: sourceToken, target: targetToken });
        SourceAndTargetAmounts memory amounts = _tradeSourceAndTargetAmounts(tokens, tradeActions, strategyPair, false);
        return amounts.targetAmount;
    }
```

It calculates the amount of target token that will be received in a trade given the source token, target token, and trade actions.

```solidity
    function accumulatedFees(Token token) external view validAddress(Token.unwrap(token)) returns (uint256) {
        return _accumulatedFees[token];
    }
```

It returns the amount of accumulated fees for a given token.It provides transparency and allows users to verify that fees are being collected as expected and distributed in a fair and transparent manner.

```solidity
function withdrawFees(
    Token token,
    uint256 amount,
    address recipient
)
    external
    whenNotPaused
    onlyRoleMember(ROLE_FEES_MANAGER)
    validAddress(recipient)
    validAddress(Token.unwrap(token))
    greaterThanZero(amount)
    nonReentrant
    returns (uint256)
{}
```

This function is called to withdraw collected fees for a given token. The parameters are token, the address of the token for which fees are being withdrawn, amount, the amount of fees to withdraw, and recipient, the address to which the fees will be sent.

```solidity
    return _withdrawFees(msg.sender, amount, token, recipient);
```

It calls the internal _withdrawFees function, passing msg.sender as the caller, the amount and token parameters as specified in the function call, and recipient as the recipient of the withdrawn fees.Its purpose is to perform the actual withdrawal of fees and update the relevant state variables in the contract.

```solidity
_accumulatedFees[token] = accumulatedAmount - amount;
```

It gets the total amount of fees accumulated for the given token.

```solidity
if (accumulatedAmount == 0) {
    return 0;
}
```

If there are no accumulated fees for the token, the function returns zero.

```solidity
if (amount > accumulatedAmount) {
    amount = accumulatedAmount;
}
```

If the requested withdrawal amount is greater than the accumulated fees, the withdrawal amount is set to the accumulated fees.

```solidity
_accumulatedFees[token] = accumulatedAmount - amount;
```

It subtracts the withdrawn amount from the accumulated fees for the token.By subtracting the withdrawn amount from the accumulated fees, the smart contract can ensure that no more fees are withdrawn than what has been accumulated for the token. This helps to prevent any potential errors or discrepancies in the fee calculation, and ensures that the total amount of fees collected is always accurate.

```solidity
_withdrawFunds(token, payable(recipient), amount);
```

It calls the _withdrawFunds function to transfer the fees to the recipient's address.

```solidity
emit FeesWithdrawn(token, recipient, amount, sender);
```

Emits an event to record the withdrawal of fees.

```solidity
return amount;
```

Finally, the function returns the amount of fees that were withdrawn.

```solidity
function _validateTradeParams(
    Token sourceToken,
    Token targetToken,
    uint256 deadline,
    uint256 value,
    uint128 constraint
) private view {}
```

This function is used to perform necessary validations on the trade parameters before executing a trade.This function takes five input parameters: the source token, target token, deadline, value, and constraint. All of them are passed as arguments in the function call.

    sourceToken: A Token struct representing the source token of the trade. This token is the one that the user wants to sell.
    targetToken: A Token struct representing the target token of the trade. This token is the one that the user wants to buy.
    deadline: An unsigned integer representing the deadline for the trade, measured in seconds since the Unix epoch.
    value: An unsigned integer representing the amount of the source token being sent to the smart contract. This is only relevant if the source token is the native token of the blockchain (e.g. ETH on Ethereum).
    constraint: An unsigned integer representing either the minimum amount of the target token that the user is willing to accept (if the TradeType is ExactInput) or the maximum amount of the source token that the user is willing to sell (if the TradeType is ExactOutput).

```solidity
if (deadline < block.timestamp) {
    revert DeadlineExpired();
}
```

It checks whether the deadline for the trade has expired or not.

```solidity
_greaterThanZero(constraint);
```

It checks if the trade constraint is greater than zero. 

```solidity
_validateInputTokens(sourceToken, targetToken);
```

It calls the _validateInputTokens function to check whether the source and target tokens are valid or not.

```solidity
if (value > 0 && !sourceToken.isNative()) {
    revert UnnecessaryNativeTokenReceived();
}
```

It checks whether any native token has been sent or not. It throws an error if there is any native token sent and the source token is not the native token.


