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
