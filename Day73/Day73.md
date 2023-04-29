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

## Smart Contract Walkthrough

### CarbonController.sol

The purpose of the contract is to serve as a controller for trading and investment strategies between different token pairs. It contains functionality for creating new trading pairs, creating and updating investment strategies, setting trading fees, and managing emergency stops and pausing of the contract.

```solidity
import { ReentrancyGuardUpgradeable } from "@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol";
import { PausableUpgradeable } from "@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol";
import { IVersioned } from "../utility/interfaces/IVersioned.sol";
import { Pairs, Pair } from "./Pairs.sol";
import { Token } from "../token/Token.sol";
import { Strategies, Strategy, TradeAction, Order, TradeTokens } from "./Strategies.sol";
import { Upgradeable } from "../utility/Upgradeable.sol";
import { IVoucher } from "../voucher/interfaces/IVoucher.sol";
import { ICarbonController } from "./interfaces/ICarbonController.sol";
import { Utils, AccessDenied } from "../utility/Utils.sol";
import { OnlyProxyDelegate } from "../utility/OnlyProxyDelegate.sol";
import { MAX_GAP } from "../utility/Constants.sol";
```

Here's a brief description of each import:

    ReentrancyGuardUpgradeable: This is an OpenZeppelin library that prevents reentrancy attacks by using a mutex lock.

    PausableUpgradeable: This is another OpenZeppelin library that allows the contract owner to pause the contract in case of emergencies.

    IVersioned: This is an interface for versioning contracts. It's used to check the version of other contracts that the CarbonController interacts with.

    Pairs and Pair: These are contracts that define the pairs of tokens that the CarbonController can trade.

    Token: This is the contract for the token that the CarbonController uses to trade with.

    Strategies, Strategy, TradeAction, Order, and TradeTokens: These are contracts and enums that define the trading strategies that the CarbonController can use.

    Upgradeable: This is a base contract that allows the CarbonController to be upgraded in the future.

    IVoucher: This is an interface for a voucher contract that the CarbonController can interact with.

    ICarbonController: This is an interface for the CarbonController itself.

    Utils and AccessDenied: These are utility contracts that contain various helper functions and error messages.

    OnlyProxyDelegate: This is a contract that restricts access to certain functions to only the proxy delegate.

    MAX_GAP: This is a constant that defines the maximum allowed slippage for trades.

```solidity
contract CarbonController is
    ICarbonController,
    Pairs,
    Strategies,
    Upgradeable,
    ReentrancyGuardUpgradeable,
    PausableUpgradeable,
    OnlyProxyDelegate,
    Utils
{}
```

This is the declaration of the CarbonController smart contract. It declares that the contract implements the ICarbonController interface and inherits from several other contracts, including Pairs, Strategies, Upgradeable, ReentrancyGuardUpgradeable, PausableUpgradeable, OnlyProxyDelegate, and Utils.

```solidity
    // the emergency manager role is required to pause/unpause
    bytes32 private constant ROLE_EMERGENCY_STOPPER = keccak256("ROLE_EMERGENCY_STOPPER");
```

This role is required to pause or unpause the contract in case of an emergency. The role is assigned to a specific address, which is responsible for executing this function.

```solidity
    // the fees manager role is required to withdraw fees
    bytes32 private constant ROLE_FEES_MANAGER = keccak256("ROLE_FEES_MANAGER");
```

This role is required to withdraw fees from the contract. The role is assigned to a specific address that is responsible for managing and withdrawing the fees from the contract.

By defining these roles, the contract can ensure that only authorized parties can execute certain functions, providing an extra layer of security and control over the contract's operations.

```solidity
uint16 private constant CONTROLLER_TYPE = 1;
```

By setting a constant identifier for the ICarbonController interface, it makes it easier for other contracts to interact with the CarbonController contract. When a contract needs to reference the CarbonController contract, it can use the CONTROLLER_TYPE constant to ensure it is interacting with the correct type of contract.

```solidity
// the voucher contract
IVoucher private immutable _voucher;
```

This variable can be used to interact with the voucher contract from within the CarbonController contract.

```solidity
// upgrade forward-compatibility storage gap
uint256[MAX_GAP] private __gap;
```

The \_\_gap array acts as a placeholder for any additional state variables that may be added to the contract in future upgrades. This is necessary because adding new state variables to an existing contract can break any existing contracts that rely on the old layout of the storage.

```solidity
    /**
     * @dev a "virtual" constructor that is only used to set immutable state variables
     */
    constructor(IVoucher initVoucher, address proxy) OnlyProxyDelegate(proxy) {
        _validAddress(address(initVoucher));

        _voucher = initVoucher;
    }
```

The purpose of the constructor is to initialize the contract's state variables, specifically the \_voucher variable which is of type IVoucher. This variable is set to the value passed as initVoucher during the contract's deployment.

The constructor also checks the validity of the initVoucher address by calling the \_validAddress() function which ensures that the address is not zero (0x0).

Finally, the constructor ensures that the contract is only accessible through the proxy by calling the OnlyProxyDelegate(proxy) modifier.

```solidity
    /**
     * @dev fully initializes the contract and its parents
     */
    function initialize() external initializer {
        __CarbonController_init();
    }

    // solhint-disable func-name-mixedcase

    /**
     * @dev initializes the contract and its parents
     */
    function __CarbonController_init() internal onlyInitializing {
        __Pairs_init();
        __Strategies_init();
        __Upgradeable_init();
        __ReentrancyGuard_init();
        __Pausable_init();

        __CarbonController_init_unchained();
    }

    /**
     * @dev performs contract-specific initialization
     */
    function __CarbonController_init_unchained() internal onlyInitializing {
        // set up administrative roles
        _setRoleAdmin(ROLE_EMERGENCY_STOPPER, ROLE_ADMIN);
        _setRoleAdmin(ROLE_FEES_MANAGER, ROLE_ADMIN);
    }
```

Using the initializer keyword ensures that this function will be called only once, when the contract is deployed or upgraded, to set up any initial state or perform any necessary setup operations.

These functions are used to fully initialize the CarbonController contract and its parent contracts, setting up necessary roles and permissions to manage the protocol.

```solidity
    /**
     * @inheritdoc Upgradeable
     */
    function version() public pure override(IVersioned, Upgradeable) returns (uint16) {
        return 2;
    }

    /**
     * @dev returns the emergency stopper role
     */
    function roleEmergencyStopper() external pure returns (bytes32) {
        return ROLE_EMERGENCY_STOPPER;
    }

    /**
     * @dev returns the fees manager role
     */
    function roleFeesManager() external pure returns (bytes32) {
        return ROLE_FEES_MANAGER;
    }

    /**
     * @inheritdoc ICarbonController
     */
    function controllerType() external pure virtual returns (uint16) {
        return CONTROLLER_TYPE;
    }

    /**
     * @inheritdoc ICarbonController
     */
    function tradingFeePPM() external view returns (uint32) {
        return _tradingFeePPM;
    }

    /**
     * @dev sets the trading fee (in units of PPM)
     *
     * requirements:
     *
     * - the caller must be the admin of the contract
     */
    function setTradingFeePPM(uint32 newTradingFeePPM) external onlyAdmin validFee(newTradingFeePPM) {
        _setTradingFeePPM(newTradingFeePPM);
    }
```

    version(): This function returns the version of the contract, which is determined by the Upgradeable contract and IVersioned interface that the CarbonController contract inherits from. In this case, the function returns 2.

    roleEmergencyStopper(): This function returns the ROLE_EMERGENCY_STOPPER constant, which is a unique identifier (hash) for the role that has the ability to pause and unpause the contract.

    roleFeesManager(): This function returns the ROLE_FEES_MANAGER constant, which is a unique identifier (hash) for the role that has the ability to withdraw fees from the contract.

    controllerType(): This function returns the CONTROLLER_TYPE constant, which is a unique identifier (integer) for the type of controller that this contract represents. In this case, CONTROLLER_TYPE is set to 1.

    tradingFeePPM(): This function returns the current trading fee (in parts per million) that is applied to trades in the contract.

    setTradingFeePPM(): This function sets a new trading fee (in parts per million) for the contract. It can only be called by the admin of the contract and requires that the new fee value is valid according to the validFee() modifier. The function calls _setTradingFeePPM() to actually set the new fee value.

\_setTradingFeePPM() is an internal functio of "Strategies.sol".

```solidity
    /**
     * @dev sets the trading fee (in units of PPM)
     */
    function _setTradingFeePPM(uint32 newTradingFeePPM) internal {
        uint32 prevTradingFeePPM = _tradingFeePPM;
        if (prevTradingFeePPM == newTradingFeePPM) {
            return;
        }

        _tradingFeePPM = newTradingFeePPM;

        emit TradingFeePPMUpdated({ prevFeePPM: prevTradingFeePPM, newFeePPM: newTradingFeePPM });
    }
```

The function takes a uint32 input parameter newTradingFeePPM which represents the new trading fee to be set. The function first checks if the new trading fee is the same as the current trading fee. If it is the same, the function returns without making any changes. If the new trading fee is different from the current trading fee, the function updates the \_tradingFeePPM state variable with the new trading fee.

```solidity
    /**
     * @inheritdoc ICarbonController
     */
    function createPair(
        Token token0,
        Token token1
    ) external nonReentrant whenNotPaused onlyProxyDelegate returns (Pair memory) {
        _validateInputTokens(token0, token1);
        return _createPair(token0, token1);
    }
```

The function creates a new trading pair with the given two tokens as inputs. It validates the input tokens by calling the \_validateInputTokens internal function and returns a new Pair structure representing the created trading pair.It is marked with the nonReentrant modifier from the ReentrancyGuardUpgradeable contract, which ensures that the function cannot be called again while it is still executing.The whenNotPaused modifier from the PausableUpgradeable contract ensures that the function can only be called when the contract is not paused.

The onlyProxyDelegate modifier from the OnlyProxyDelegate contract ensures that the function can only be called by the proxy contract that delegates to the CarbonController contract.The function returns a Pair structure that contains information about the newly created trading pair.

```solidity
struct Pair {
    uint128 id;
    Token[2] tokens;
}
```

The Pair struct has two fields:

    id (type uint128): a unique identifier for the pair.
    tokens (type Token[2]): an array of two Token structs that represent the two tokens that make up the pair.

```solidity
    /**
     * @dev validates both tokens are valid addresses and unique
     */
    function _validateInputTokens(
        Token token0,
        Token token1
    ) private pure validAddress(Token.unwrap(token0)) validAddress(Token.unwrap(token1)) {
        if (token0 == token1) {
            revert IdenticalAddresses();
        }
    }
```

The function takes in two parameters token0 and token1 of type Token. It first unwraps the tokens using the Token.unwrap function to obtain their underlying address and then validates that they are both valid addresses using the validAddress modifier.

After validating that the two tokens are valid addresses, the function checks that they are not identical. If they are, then it reverts with an IdenticalAddresses error message.

The Token.unwrap() function is defined as part of the type Token is address declaration. It is used to retrieve the underlying address value from a Token value. For example, if token is a Token value, then Token.unwrap(token) will return the underlying address value.

```
The type Token is address declaration creates a new type called Token that is an alias for the address type. This is done to improve code readability and make the code more self-documenting.

Instead of using the generic address type throughout the code, the Token type is used to indicate that the variable represents an Ethereum token. This is a common pattern in Solidity code, where new types are created as aliases for existing types to make the code more expressive and self-documenting.
```

```solidity
    /**
     * @dev generates and stores a new pair, tokens are assumed unique and valid
     */
    function _createPair(Token token0, Token token1) internal returns (Pair memory) {
        // validate pair existence
        if (_pairExists(token0, token1)) {
            revert PairAlreadyExists();
        }

        // sort tokens
        Token[2] memory sortedTokens = _sortTokens(token0, token1);

        // increment pair id
        uint128 id = _lastPairId + 1;
        _lastPairId = id;

        // store pair
        _pairsStorage[id] = sortedTokens;
        _pairIds[sortedTokens[0]][sortedTokens[1]] = id;

        emit PairCreated(id, sortedTokens[0], sortedTokens[1]);
        return Pair({ id: id, tokens: sortedTokens });
    }
```

This function generates and stores a new pair of tokens, assuming that the two tokens passed as parameters are unique and valid. Here's a breakdown of what the function does:

    Validates the existence of the pair by calling the _pairExists() function. If the pair already exists, it reverts with the PairAlreadyExists error.

    Sorts the tokens using the _sortTokens() function, which ensures that the tokens are sorted in a consistent order. This is important because pairs are stored based on their sorted order.

    Increments the pair ID by one and stores the sorted tokens and the ID in the _pairsStorage mapping.

    Stores the ID of the new pair in the _pairIds mapping, which is used to look up pairs by their tokens.

    Emits a PairCreated event to notify any listeners that a new pair has been created.

    Returns a Pair struct containing the pair ID and sorted tokens.

```solidity
    /**
     * @inheritdoc ICarbonController
     */
    function pairs() external view returns (Token[2][] memory) {
        return _pairs();
    }
```

It returns an array of pairs of tokens currently registered in the Carbon Controller contract. The function returns a memory array of Token[2][], which is an array of pairs of Token type elements.

The function internally calls the \_pairs() function.

```solidity
    /**
     * @dev return a pair matching the given tokens
     */
    function _pair(Token token0, Token token1) internal view returns (Pair memory) {
        // validate pair existence
        if (!_pairExists(token0, token1)) {
            revert PairDoesNotExist();
        }

        // sort tokens
        Token[2] memory sortedTokens = _sortTokens(token0, token1);

        // return pair
        uint128 id = _pairIds[sortedTokens[0]][sortedTokens[1]];
        return Pair({ id: id, tokens: sortedTokens });
    }
```

The function first checks whether the pair of tokens exists in the \_pairIds mapping using the \_pairExists function. If the pair does not exist, the function reverts with an error.If the pair exists, the function sorts the tokens using the \_sortTokens function and stores them in a Token[2] array named sortedTokens.Then, the function retrieves the ID of the pair using \_pairIds mapping and returns the ID and the sorted tokens as a Pair struct.

```solidity
    /**
     * @inheritdoc ICarbonController
     */
    function pair(Token token0, Token token1) external view returns (Pair memory) {
        _validateInputTokens(token0, token1);
        return _pair(token0, token1);
    }
```

It first calls the \_validateInputTokens function to validate the input parameters. If the tokens are not valid, it will revert the transaction with an appropriate error message. Otherwise, it will call the \_pair function, passing the input tokens as arguments.

```solidity
function createStrategy(
        Token token0,
        Token token1,
        Order[2] calldata orders
    ) external payable nonReentrant whenNotPaused onlyProxyDelegate returns (uint256) {}
```

It allows users to create a new strategy for a given token pair.It means creating a new trading strategy for a pair of tokens that have not been previously traded on the Carbon platform.

```solidity
_validateInputTokens(token0, token1);
```

It is called to validate that the input tokens are valid and unique. If not, it reverts.

```solidity
        // don't allow unnecessary eth
        if (msg.value > 0 && !token0.isNative() && !token1.isNative()) {
            revert UnnecessaryNativeTokenReceived();
        }
```

The next line checks if the msg.value is greater than 0, and neither of the tokens are native tokens (ETH or BNB). If this is the case, it reverts with an UnnecessaryNativeTokenReceived error.

This check is performed to ensure that the function is not receiving unnecessary Ether (ETH) that cannot be used in the strategy creation process.

```solidity
        // revert if any of the orders is invalid
        _validateOrders(orders);
```

This is called to validate the orders. If any of the orders is invalid, it reverts.

```solidity
    /**
     * revert if any of the orders is invalid
     */
    function _validateOrders(Order[2] calldata orders) internal pure {
        for (uint256 i = 0; i < 2; i = uncheckedInc(i)) {
            if (orders[i].z < orders[i].y) {
                revert InsufficientCapacity();
            }
            if (!_validRate(orders[i].A)) {
                revert InvalidRate();
            }
            if (!_validRate(orders[i].B)) {
                revert InvalidRate();
            }
        }
    }
```

The function takes an array of two Order structs as input, which represent the two orders that are being used in the strategy. The function then iterates over the orders using a for loop.

For each order, the function checks whether the value of z is greater than or equal to y, which is required for the order to have sufficient capacity to execute. If this condition is not met, the function reverts with the InsufficientCapacity error.

The function also checks whether the rate of each order (A and B) is valid by calling the \_validRate function. If the rate is not valid, the function reverts with the InvalidRate error.

```solidity
        // create the pair if it does not exist
        Pair memory strategyPair;
        if (!_pairExists(token0, token1)) {
            strategyPair = _createPair(token0, token1);
        } else {
            strategyPair = _pair(token0, token1);
        }
```

The function checks if a pair exists for the given tokens using \_pairExists(token0, token1). If the pair does not exist, it creates the pair using \_createPair(token0, token1) and assigns it to a memory variable called strategyPair. If the pair exists, it retrieves the pair using \_pair(token0, token1) and assigns it to strategyPair.

```solidity
        Token[2] memory tokens = [token0, token1];
        return _createStrategy(_voucher, tokens, orders, strategyPair, msg.sender, msg.value);
```

The function creates an array of the two tokens called tokens and passes it along with orders, strategyPair, msg.sender, and msg.value to \_createStrategy() to create a new strategy.Finally, the function returns the ID of the new strategy created using \_createStrategy().

```solidity
function updateStrategy(
        uint256 strategyId,
        Order[2] calldata currentOrders,
        Order[2] calldata newOrders
    ) external payable nonReentrant whenNotPaused onlyProxyDelegate {}
```

strategyId, which is the ID of the strategy to be updated, currentOrders, which is an array of the current orders associated with the strategy, and newOrders, which is an array of the new orders that will replace the current orders.

```solidity
    Pair memory strategyPair = _pairById(_pairIdByStrategyId(strategyId));
```

It retrieves the Pair associated with the given strategyId by calling the \_pairById() function.

```solidity
    /**
     * returns the pairId associated with a given strategyId
     */
    function _pairIdByStrategyId(uint256 strategyId) internal pure returns (uint128) {
        return uint128(strategyId >> 128);
    }
```

The strategyId is a uint256 number that uniquely identifies a specific strategy. The function uses bit shifting to extract the first 128 bits of the strategyId, which represents the pairId.The function then returns this pairId as a uint128 type.

The reason for doing this is that the strategyId is a 256-bit integer that encodes both the pairId and the strategyIndex. The pairId is the most significant 128 bits of the strategyId, while the strategyIndex is the least significant 128 bits. By extracting the first 128 bits of the strategyId, the \_pairIdByStrategyId function effectively returns the pairId associated with the given strategyId. This pairId is used to look up the Pair struct associated with the pair in the \_pairs mapping.

```solidity
        if (msg.sender != _voucher.ownerOf(strategyId)) {
            revert AccessDenied();
        }
```

It checks whether the caller of the function is the owner of the voucher associated with the given strategyId. If the caller is not the owner, the function reverts with an AccessDenied() error.

```solidity
        if (msg.value > 0 && !strategyPair.tokens[0].isNative() && !strategyPair.tokens[1].isNative()) {
            revert UnnecessaryNativeTokenReceived();
                }
```

It checks whether any ETH was sent with the function call and whether both tokens in the strategyPair are non-native tokens (i.e., not ETH). If both conditions are true, the function reverts with an UnnecessaryNativeTokenReceived() error.

```solidity
        _validateOrders(newOrders);
```

It checks whether the orders in newOrders are valid. If any order is invalid, the function reverts with an error.

```solidity
        _updateStrategy(strategyId, currentOrders, newOrders, strategyPair, msg.sender, msg.value);
```

It performs the actual update of the strategy with the new orders.
