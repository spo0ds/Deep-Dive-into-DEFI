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
