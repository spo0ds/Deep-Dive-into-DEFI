## BancorArbitrage.sol

The Bancor protocol is a decentralized liquidity network that allows for automated token exchange without requiring a counterparty to the trade. Bancor introduced the concept of the “arbitrage contract”, which is a smart contract that uses flash loans to execute arbitrage trades between different decentralized exchanges.

```solidity
import { ReentrancyGuardUpgradeable } from "@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol";
import { IERC20 } from "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import { SafeERC20 } from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
import { Address } from "@openzeppelin/contracts/utils/Address.sol";

import { IUniswapV2Router02 } from "@uniswap/v2-periphery/contracts/interfaces/IUniswapV2Router02.sol";
import { ISwapRouter } from "@uniswap/v3-periphery/contracts/interfaces/ISwapRouter.sol";
import { IWETH } from "@uniswap/v2-periphery/contracts/interfaces/IWETH.sol";

import { Token } from "../token/Token.sol";
import { TokenLibrary } from "../token/TokenLibrary.sol";
import { IVersioned } from "../utility/interfaces/IVersioned.sol";
import { Upgradeable } from "../utility/Upgradeable.sol";
import { Utils } from "../utility/Utils.sol";
import { IBancorNetwork, IFlashLoanRecipient } from "../network/interfaces/IBancorNetwork.sol";
import { PPM_RESOLUTION } from "../utility/Constants.sol";
import { MathEx } from "../utility/MathEx.sol";
```

```    
    ReentrancyGuardUpgradeable from "@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol": This is a contract that provides protection against reentrancy attacks.

    IERC20 from "@openzeppelin/contracts/token/ERC20/IERC20.sol": This is an interface that defines the standard functions for interacting with ERC20 tokens.

    SafeERC20 from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol": This is a library that provides safe methods for transferring ERC20 tokens, protecting against common vulnerabilities such as integer overflow and underflow.

    Address from "@openzeppelin/contracts/utils/Address.sol": This is a library that provides utility functions for working with Ethereum addresses.

    IUniswapV2Router02 from "@uniswap/v2-periphery/contracts/interfaces/IUniswapV2Router02.sol": This is an interface that defines the functions for interacting with the Uniswap V2 router.

    ISwapRouter from "@uniswap/v3-periphery/contracts/interfaces/ISwapRouter.sol": This is an interface that defines the functions for interacting with the Uniswap V3 router.

    IWETH from "@uniswap/v2-periphery/contracts/interfaces/IWETH.sol": This is an interface that defines the functions for interacting with the WETH token contract.

    Token from "../token/Token.sol": This is a contract that defines a custom token.

    TokenLibrary from "../token/TokenLibrary.sol": This is a library that provides utility functions for working with custom tokens.

    IVersioned from "../utility/interfaces/IVersioned.sol": This is an interface that defines the functions for checking the version of a contract.

    Upgradeable from "../utility/Upgradeable.sol": This is a contract that provides functionality for upgrading contracts.

    Utils from "../utility/Utils.sol": This is a library that provides various utility functions.

    IBancorNetwork and IFlashLoanRecipient from "../network/interfaces/IBancorNetwork.sol": These are interfaces that define the functions for interacting with the Bancor Network.

    PPM_RESOLUTION from "../utility/Constants.sol": This is a constant that defines the precision used for calculations.

    MathEx from "../utility/MathEx.sol": This is a library that provides advanced math functions.
```

```solidity
// interface to support Bancor V2 trades
interface IBancorNetworkV2 {
    function convertByPath(
        address[] memory _path,
        uint256 _amount,
        uint256 _minReturn,
        address _beneficiary,
        address _affiliateAccount,
        uint256 _affiliateFee
    ) external payable returns (uint256);

    function conversionPath(Token _sourceToken, Token _targetToken) external view returns (address[] memory);
}
```

The IBancorNetworkV2 interface is used in the arbitrage contract to interact with the Bancor Network version 2 smart contract.

`convertByPath` function allows users to convert a token into another token via a path of intermediate tokens.The input parameters of the function are:

    _path: An array of addresses representing the token path, where the first address is the source token and the last address is the target token.

    _amount: The amount of source tokens to be converted.
    
    _minReturn: The minimum amount of target tokens expected to be received after the conversion.

    _beneficiary: The address that will receive the target tokens after the conversion.

    _affiliateAccount: The address of the affiliate account that will receive the affiliate fee.

    _affiliateFee: The percentage of the conversion that will be paid as an affiliate fee.
    
`conversionPath` This function returns the token conversion path between two tokens. 

    _sourceToken: The address of the source token.
    _targetToken: The address of the target token.
    
```solidity
    // trade args
    struct Route {
        uint16 exchangeId;
        Token targetToken;
        uint256 minTargetAmount;
        uint256 deadline;
        address customAddress;
        uint256 customInt;
    }
```

The Route struct is used to define the parameters for a specific trade route. It includes the following fields:

        exchangeId: an integer that identifies the exchange where the trade will occur. This is used to determine which trade function to call within the contract.
    
        targetToken: an instance of the Token struct that represents the token being traded for.
    
        minTargetAmount: the minimum amount of the targetToken that the trader is willing to accept in exchange for the input token.
    
        deadline: a timestamp that specifies the latest time at which the trade can be executed.
    
        customAddress: an optional address parameter that can be used for specific exchange functions that require additional arguments.
    
        customInt: an optional integer parameter that can be used for specific exchange functions that require additional arguments.
        
```solidity
    // rewards settings
    struct Rewards {
        uint32 percentagePPM;
        uint256 maxAmount;
    }
```

The Rewards struct is used to define the settings for the rewards that will be given to the user who calls the arbitrage() function. The percentagePPM field represents the percentage of the profit that will be given as a reward, in parts per million (PPM). The maxAmount field represents the maximum amount of tokens that can be given as a reward to the user. These settings are used in the calculateRewards() function to calculate the amount of rewards to be given to the user based on the profit earned from the arbitrage trade.

```solidity
    // exchange ids
    uint16 public constant EXCHANGE_ID_BANCOR_V2 = 1;
    uint16 public constant EXCHANGE_ID_BANCOR_V3 = 2;
    uint16 public constant EXCHANGE_ID_UNISWAP_V2 = 3;
    uint16 public constant EXCHANGE_ID_UNISWAP_V3 = 4;
    uint16 public constant EXCHANGE_ID_SUSHISWAP = 5;
```

The purpose of these variables is to define the different exchange ids that can be used for trading. Each exchange id corresponds to a specific DEX, such as Bancor V2, Bancor V3, Uniswap V2, Uniswap V3, and Sushiswap. These exchange ids are used to specify which exchange to use for each trade route defined in the Route struct. By using these exchange ids, the contract can easily determine which specific exchange contract to interact with for each trade, without having to hardcode the addresses of the exchange contracts.

`why ids are used instead of actual address?`

Using IDs instead of actual addresses can make the code more flexible and maintainable. The actual addresses of the exchanges and routers can change over time, and if the contract relies on hardcoded addresses, it would need to be updated every time an address changes. By using IDs, the contract can be updated with the new addresses without having to modify the contract logic. Additionally, using IDs can make the contract more gas-efficient since the IDs can be represented as uint16 values instead of full-length addresses.

```solidity
// maximum number of trade routes supported
uint256 private constant MAX_ROUTE_LENGTH = 10;
```

It is used to define the maximum number of trade routes that the contract can support. This is done to limit the complexity and gas cost of the contract, as allowing an unlimited number of routes could potentially lead to a situation where the contract exceeds the gas limit and cannot be executed.

By setting a maximum number of routes, the contract can handle a reasonable number of arbitrage opportunities without becoming overly complex or expensive to execute. In this case, the maximum number of routes is set to 10, which should be sufficient to handle most potential opportunities while keeping the gas cost reasonable.

```solidity
    // the bnt contract
    IERC20 internal immutable _bnt;

    // WETH9 contract
    IERC20 internal immutable _weth;

    // bancor v2 network contract
    IBancorNetworkV2 internal immutable _bancorNetworkV2;

    // bancor v3 network contract
    IBancorNetwork internal immutable _bancorNetworkV3;

    // uniswap v2 router contract
    IUniswapV2Router02 internal immutable _uniswapV2Router;

    // uniswap v3 router contract
    ISwapRouter internal immutable _uniswapV3Router;

    // sushiSwap router contract
    IUniswapV2Router02 internal immutable _sushiSwapRouter;
```

These variables are used to store the instances of the different smart contracts that are required to execute the arbitrage trades:

    _bnt: An instance of the BNT token contract, which is the native token of the Bancor Network.
    _weth: An instance of the WETH9 contract, which is the wrapped ether token used in Ethereum.
    _bancorNetworkV2: An instance of the Bancor Network V2 contract, which is used to execute trades on the Bancor Network V2.
    _bancorNetworkV3: An instance of the Bancor Network V3 contract, which is used to execute trades on the Bancor Network V3.
    _uniswapV2Router: An instance of the Uniswap V2 Router contract, which is used to execute trades on the Uniswap V2 exchange.
    _uniswapV3Router: An instance of the Uniswap V3 Router contract, which is used to execute trades on the Uniswap V3 exchange.
    _sushiSwapRouter: An instance of the SushiSwap Router contract, which is used to execute trades on the SushiSwap exchange.
    
These contracts are used in different parts of the code to perform the actual trades and to interact with the different liquidity pools.

```solidity
// rewards defaults
Rewards internal _rewards;
```

The code is used to define a struct variable named _rewards that holds two properties percentagePPM and maxAmount.

```solidity
    /**
     * @dev triggered after a successful arb is executed
     */
    event ArbitrageExecuted(
        address indexed caller,
        uint16[] exchangeIds,
        address[] tokenPath,
        uint256 sourceAmount,
        uint256 burnAmount,
        uint256 rewardAmount
    );

    /**
     * @dev triggered when the rewards settings are updated
     */
    event RewardsUpdated(
        uint32 prevPercentagePPM,
        uint32 newPercentagePPM,
        uint256 prevMaxAmount,
        uint256 newMaxAmount
    );
```

The ArbitrageExecuted event is triggered after a successful arbitrage is executed. It provides information about the caller of the function, the exchange IDs used in the trade, the token path followed, the source amount used, the amount of tokens burned in the process, and the reward amount received by the caller.

The RewardsUpdated event is triggered when the rewards settings of the contract are updated. It provides information about the previous and new percentage and maximum amount of rewards that can be earned by the caller of the function.

```solidity
    /**
     * @dev fully initializes the contract and its parents
     */
    function initialize() external initializer {
        __BancorArbitrage_init();
    }

    // solhint-disable func-name-mixedcase

    /**
     * @dev initializes the contract and its parents
     */
    function __BancorArbitrage_init() internal onlyInitializing {
        __ReentrancyGuard_init();
        __Upgradeable_init();

        __BancorArbitrage_init_unchained();
    }

    /**
     * @dev performs contract-specific initialization
     */
    function __BancorArbitrage_init_unchained() internal onlyInitializing {
        _rewards = Rewards({ percentagePPM: 100000, maxAmount: 100 * 1e18 });
    }
```

The initialize function is called externally after the contract has been deployed, and it calls the internal function __BancorArbitrage_init(). This function initializes the contract and its parents by calling their respective initialization functions.When initialize() is called, the initializer modifier checks whether the contract is in the "initializing" state, and if so, it allows the __BancorArbitrage_init() function to be called.This is a common pattern used in upgradeable contracts to ensure that initialization can only be performed once, during the contract's deployment.

The function calls __BancorArbitrage_init(), which sets up the contract by initializing its parent contracts (__ReentrancyGuard_init() and __Upgradeable_init()) and calling the __BancorArbitrage_init_unchained() function to perform the contract-specific initialization.

```
    __ReentrancyGuard_init():
    The __ReentrancyGuard_init() function is a part of the ReentrancyGuard contract, which is used to prevent reentrancy attacks. Reentrancy attacks occur when an attacker repeatedly re-enters a function before the previous invocation has been completed. This can lead to unexpected behavior and can even result in loss of funds. The ReentrancyGuard contract includes a mutex lock that prevents reentrant calls. The __ReentrancyGuard_init() function initializes the mutex lock.

    __Upgradeable_init():
    The __Upgradeable_init() function is a part of the Initializable contract, which is used to enable upgradeability of smart contracts. When a contract is upgradeable, it means that its code can be changed without losing the state data stored in the contract. The Initializable contract includes an initializer modifier that can be used to ensure that the contract is initialized only once. The __Upgradeable_init() function initializes the Initializable contract.

    __BancorArbitrage_init_unchained():
    The __BancorArbitrage_init_unchained() function is a part of the BancorArbitrage contract and performs contract-specific initialization. It sets the default values for the _rewards variable, which is a struct that defines the rewards settings for the contract. In this case, the default rewards are set to a percentage of 100000 (or 10%), with a maximum reward amount of 100 BNT.
```

```solidity
    /**
     * @dev authorize the contract to receive the native token
     */
    receive() external payable {}
```

The receive() function in the arbitrage contract is used to receive ether (ETH) when it is sent to the contract address. It is marked as payable to indicate that it can receive value, and since it has no function signature, it is called when ether is sent to the contract address.

This function is needed in the contract because some of the external functions used in the contract may require ETH as input. For example, when swapping tokens on Uniswap or SushiSwap, the contracts require a certain amount of ETH to pay for gas fees and liquidity fees. Therefore, if the contract does not have any ETH, the swap transaction will fail.

By having a receive() function, the contract can receive ETH from external accounts, such as a user's wallet or a separate smart contract, and use it to perform arbitrage trades on different exchanges.

```solidity
    /**
     * @dev checks whether the specified number of routes is supported
     */
    modifier validRouteLength(Route[] calldata routes) {
        // validate inputs
        _validRouteLength(routes);

        _;
    }

    /**
     * @dev validRouteLength logic for gas optimization
     */
    function _validRouteLength(Route[] calldata routes) internal pure {
        if (routes.length == 0 || routes.length > MAX_ROUTE_LENGTH) {
            revert InvalidRouteLength();
        }
    }
```

The purpose of the validRouteLength modifier in the arbitrage contract is to check whether the specified number of routes is supported before executing a function. This is to prevent invalid inputs that may lead to errors or unexpected behavior in the contract.

```solidity
    /**
     * @dev sets the rewards settings
     *
     * requirements:
     *
     * - the caller must be the admin of the contract
     */
    function setRewards(
        Rewards calldata newRewards
    ) external onlyAdmin validFee(newRewards.percentagePPM) greaterThanZero(newRewards.maxAmount) {}
```

The setRewards function sets the reward settings for the arbitrage transactions. It takes in a Rewards struct containing two parameters: percentagePPM and maxAmount, which determine the percentage fee charged and the maximum amount of tokens that can be earned as rewards.

The function first checks that the caller is the admin of the contract, as specified by the onlyAdmin modifier.

```solidity
    // In Upgradeable.sol
    modifier onlyAdmin() {
        _hasRole(ROLE_ADMIN, msg.sender);

        _;
    }
```

The modifier restricts access to a function to only the accounts that have been assigned the ROLE_ADMIN role.

It then checks that the new percentagePPM value is valid (i.e., between 0 and 1,000,000), and that the maxAmount is greater than zero, as specified by the validFee and greaterThanZero modifiers.

```solidity
    // Utils.sol
    // ensures that the fee is valid
    modifier validFee(uint32 fee) {
        _validFee(fee);

        _;
    }

    // error message binary size optimization
    function _validFee(uint32 fee) internal pure {
        if (fee > PPM_RESOLUTION) {
            revert InvalidFee();
        }
    }

`   // verifies that a value is greater than zero
    modifier greaterThanZero(uint256 value) {
        _greaterThanZero(value);

        _;
    }

    // error message binary size optimization
    function _greaterThanZero(uint256 value) internal pure {
        if (value == 0) {
            revert ZeroValue();
        }
    }
```

This function provides a way for the admin of the contract to adjust the rewards for arbitrage transactions in a flexible and transparent way.

```solidity
        uint32 prevPercentagePPM = newRewards.percentagePPM;
        uint256 prevMaxAmount = _rewards.maxAmount;
```

The two lines are used to store the current values of percentagePPM and maxAmount of the _rewards variable, which represent the previous rewards configuration.This is done before actually setting the new rewards configuration, so that the previous value can be used for reference in case of future changes or to revert back to it if needed.

```solidity
        // return if the rewards are the same
        if (prevPercentagePPM == newRewards.percentagePPM && prevMaxAmount == newRewards.maxAmount) {
            return;
        }
```

The code you provided checks if the new rewards being set are the same as the existing rewards. If the new rewards are the same as the existing rewards, the function returns without making any further changes. This is done to prevent unnecessary gas consumption and to avoid setting duplicate rewards.

```solidity
_rewards = newRewards;
```

This assigns the new reward configuration to the contract's _rewards state variable. This means that the new rewards configuration is now the active one that will be used to calculate rewards for users. 

```solidity
    /**
     * @dev execute multi-step arbitrage trade between exchanges
     */
    function execute(
        Route[] calldata routes,
        uint256 sourceAmount
    ) public payable nonReentrant validRouteLength(routes) greaterThanZero(sourceAmount) {}
```

The execute function is a multi-step arbitrage trade between exchanges in the Bancor protocol. It takes in an array of Route objects and a sourceAmount, which is the amount of the initial token that will be used to perform the arbitrage. 

`Why this function is declared as public instead of external?`

The reason it is declared as public instead of external is likely because it needs to be called from inside the contract as well (specifically, to allocate rewards). 

```solidity
        // verify that the last token in the process is BNT
        if ((address(routes[routes.length - 1].targetToken) != address(_bnt))) {
            revert InvalidInitialAndFinalTokens();
        }
```

It verifies that the last token in the series of trades defined by the routes parameter is BNT (the Bancor Network Token). If the last token is not BNT, it reverts the transaction and throws an InvalidInitialAndFinalTokens error, indicating that the trade cannot be executed as defined.

```solidity
         // take a flashloan for the source amount on Bancor v3 and perform the trades
        _bancorNetworkV3.flashLoan(
            Token(address(_bnt)),
            sourceAmount,
            IFlashLoanRecipient(address(this)),
            abi.encode(routes, sourceAmount)
        );
```

Once this is verified, the next step is to execute the trade. This is done by calling the flashLoan function on the Bancor Network V3 contract. The flashLoan function is used to borrow tokens from the Bancor liquidity pool for the duration of the transaction. The flashLoan function takes four arguments:

```
    The token to be borrowed (in this case, BNT).
    The amount of the token to be borrowed (in this case, sourceAmount).
    The recipient of the borrowed tokens (in this case, the current contract address).
    The data to be passed to the recipient (in this case, the routes array and sourceAmount).
```

The flashLoan function will execute the arbitrage trade using the specified routes, and then return the borrowed BNT token back to the liquidity pool, along with the flashloan fee.

After the trade is executed, the last step is to allocate the rewards. The _allocateRewards function is called to distribute a portion of the profits to the contract owner and the caller of the execute function.

```solidity
        // allocate the rewards
        _allocateRewards(routes, sourceAmount, msg.sender);
```

The _allocateRewards function is called to allocate the rewards to the user who executed the arbitrage trade.

The function takes three arguments:

    routes: an array of Route structs which describe the sequence of trades that were performed to complete the arbitrage trade.
    sourceAmount: the amount of source token that was traded in the arbitrage.
    user: the address of the user who executed the arbitrage trade.
    
The function calculates the total profit made from the arbitrage trade, which is the difference between the final amount of BNT received and the initial amount of source token spent, taking into account any fees charged by the exchanges along the way.

The profit is then split between the user and the protocol based on a pre-determined fee schedule. The user receives a percentage of the profit as a reward, and the rest is retained by the protocol. The reward is transferred to the user's address, and the protocol's share of the profit is added to the protocol's balance in the master vault. 

```solidity
    /**
     * @dev callback function for bancor V3 flashloan
     */
    function onFlashLoan(
        address caller,
        IERC20 erc20Token,
        uint256 amount,
        uint256 feeAmount,
        bytes memory data
    ) external {}
```

The onFlashLoan function is the callback function that is called by the BancorNetwork contract after a flash loan is taken by the execute function.

`Why does it declared as external instead of public?`

When the Bancor V3 contract executes a flash loan, it calls the onFlashLoan function of the recipient contract (in this case, the arbitrage contract) to perform some custom logic with the borrowed funds. The onFlashLoan function needs to be declared as external so that the Bancor V3 contract can call it correctly during the flash loan execution.

Furthermore, the onFlashLoan function doesn't need to be accessed externally by other contracts, so there is no need to declare it as public.

```solidity
        // validate inputs
        if (msg.sender != address(_bancorNetworkV3) || caller != address(this)) {
            revert InvalidFlashLoanCaller();
        }
```

 It ensures that the function is being called by the BancorNetworkV3 contract and that the caller is the current contract (i.e., address(this)). 
 
 ```solidity
 // decode the data
(Route[] memory routes, uint256 sourceAmount) = abi.decode(data, (Route[], uint256));
```

The code decodes the data parameter passed in the flash loan request into a tuple consisting of an array of Route structs and a sourceAmount variable. This is done using the abi.decode() function, which takes the data parameter as input and a tuple type, (Route[], uint256), specifying the expected data types and structure of the decoded output.

The Route struct is a custom data structure defined in the Arbitrage contract, representing a single trade route in the arbitrage sequence. The sourceAmount variable represents the initial amount of the flash loan, which will be used as the source amount in the first trade of the sequence.

Once the data parameter is decoded, the function proceeds to execute the arbitrage trade sequence using the routes array and sourceAmount variable.

abi.decode is a built-in Solidity function that allows you to decode data from a byte array to a specific data type.

```solidity
// perform the trade routes
Token sourceToken = Token(address(_bnt));
```

sourceToken is used to keep track of the current token being traded. In each iteration of the for loop, the sourceToken is swapped for the targetToken of the current trade route, so that the resulting token balance after the trade becomes the sourceToken for the next trade.

`Why do we need to track the current token being traded?`

We need to track the current token being traded because the function is performing multiple trades in a sequence, where the output of one trade is used as the input for the next trade. Each trade is executed on a different token pair, so we need to keep track of the current token being traded so that we can use it as the input token for the next trade. The output token of each trade becomes the input token of the next trade, and this process continues until all the trades in the sequence have been executed. By keeping track of the current token being traded, we can ensure that the correct token pairs are used for each trade, and that the sequence of trades is executed correctly.

`Is it because bancor trades everything into BNT we ned to track the token?`

Yes, that's correct. Bancor's liquidity network is designed to use BNT as the bridge token, which means that every trade route ultimately involves swapping some token for BNT, and then swapping that BNT for the next token in the route. Therefore, it's necessary to track the current token being traded, so that the contract can use it as the source token for the next trade in the route.

```solidity
        for (uint256 i = 0; i < routes.length; i++) {
            // save the current balance
            uint256 previousBalance = routes[i].targetToken.balanceOf(address(this));

            // perform the trade
            _trade(
                routes[i].exchangeId,
                sourceToken,
                routes[i].targetToken,
                sourceAmount,
                routes[i].minTargetAmount,
                routes[i].deadline,
                routes[i].customAddress,
                routes[i].customInt
            );

            // the current iteration target token is the source token in the next iteration
            sourceToken = routes[i].targetToken;

            // the resulting trade amount is the source amount in the next iteration
            sourceAmount = routes[i].targetToken.balanceOf(address(this)) - previousBalance;
        }
```

Code block loops over each trade route defined in the routes array, performs a trade for each route, and updates the source token and source amount for the next trade.

For each route, the current balance of the target token (the token to be acquired) is saved before the trade is performed. The _trade function is then called with the exchange ID, source token, target token, source amount, minimum target amount, deadline, and custom address and integer specified in the route. This function executes the trade on the specified exchange and returns the amount of target token acquired.

After the trade, the source token for the next trade is set to the target token of the current trade. This is because the target token acquired in the current trade will be used as the source token for the next trade. Similarly, the source amount for the next trade is updated to be the difference between the current balance of the target token and the previous balance of the target token. This is because the source amount for the next trade is the amount of target token acquired in the current trade.

This loop continues until all routes have been executed. Once all trades have been executed, the function will have acquired the desired token in the final trade route.

```solidity
        // return the flashloan
        erc20Token.safeTransfer(msg.sender, amount + feeAmount);
```

This code transfers the flashloaned tokens back to the original caller along with the fee amount charged by the protocol for facilitating the flash loan.

msg.sender is the address of the contract that initiated the flash loan, and amount + feeAmount is the total amount of tokens that need to be returned to the lending protocol.

`Doesn't fee amount be subtracted from the amount?`

No, because the fee is transferred to the bancor.


