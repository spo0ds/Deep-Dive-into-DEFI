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
