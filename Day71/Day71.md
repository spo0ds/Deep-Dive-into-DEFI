# Bancor

Bancor is a DEX similar to Uniswap made up of liquidity pools and AMM. It's the first protocol to be a DEX. The native token of this protocol is BNT, which is the very first ERC20 token. You are depositing your token in the liquidity pool with this BNT token. Two massive advantages of Bancor are single-sided liquidity and extra liquidity farming rewards, which are paid out in BNT tokens, and liquidity protection (impermanent loss insurance). If you're a LINK owner and not interested in any other tokens but want to earn some fees and passive income on your LINK, you can enter the LINK pool on Bancor and just put LINK into that pool. So you don't have to swap 50% of your LINK into any other token, and the BNT token that was minted by the pool will be burned once you withdraw your LINK, and the trading fee earned by the protocol-minted BNT will also be burned.

Let's say you want to trade LINK for ETH through Bancor pools. Well, technically, you'll be making trades from LINK to BNT and from BNT to ETH, which makes BNT one of the most liquid and most traded tokens. All trades are routed through the BNT token, which generates a fee each time a user uses it and for the protocol itself. A secondary benefit of this is that all traders pay a fee for their trades. Since the Bancor protocol minted half of the liquidity for each pool, it means they're entitled to half of the trading fees that were generated, which are used to provide impermanent loss insurance.

Impermanent loss insurance means if I put some liquidity into a pool, prices can go up and down, so I would suffer impermanent loss, but Bancor actually has protected pools that, if you keep your liquidity in these pools for a minimum of 30 days, after that 30 days you start to earn protection, so that no matter what happens to the value of your stake, you'll never actually come out with less than you put in, regardless of what's going on with impermanent loss. After 30 days, you get 30% protection, and then for every additional day, you get an extra 1%. Once I reach 100%, I can keep earning fees regardless of the price fluctuation, and even if the price goes down, I would still get my initial stake.

Liquidity providers pay monthly to protect themselves if something really bad happens. With Bancor, you pay 15% of the trading fees that you earn as a liquidity provider into a pool that protects you and your fellow liquidity providers from impermanent losses. As mentioned, Bancor also uses fees earned on its own BNT supplied to its pool to help fund its insurance payouts, and after covering the cost of insurance, it then burns any excess fees to keep the total supply in check.

BNT is not a governance token on its own. You need to actually stake BNT in Bancor in order to get voting rights to vote in the Bancor DAO. Whenever you stake your BNT tokens, you technically receive vBNT. One could technically say Bancor has a POS governance model. All the voters in the Bancor DAO are both BNT holders and liquidity providers, whereas in a DEX like Uniswap, token holders who vote on the changes don't have to be liquidity providers. This basically means all BNT holders will be voting in a way that is most likely beneficial to all the other liquidity providers, while on the other hand, Uniswap holders will vote in a way that's exclusively beneficial for the Uni token and not necessarily the liquidity providers.

## Smart contract walkthrough

### Vault.sol

Vault allows for the secure management and withdrawal of tokens.

First we can see alots of imports in the vault contract.

```solidity
// SPDX-License-Identifier: SEE LICENSE IN LICENSE
pragma solidity 0.8.13;

import { Address } from "@openzeppelin/contracts/utils/Address.sol";
import { IERC20 } from "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import { SafeERC20 } from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
import { ReentrancyGuardUpgradeable } from "@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol";
import { PausableUpgradeable } from "@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol";

import { ITokenGovernance } from "@bancor/token-governance/contracts/ITokenGovernance.sol";

import { IVault, ROLE_ASSET_MANAGER } from "./interfaces/IVault.sol";
import { Upgradeable } from "../utility/Upgradeable.sol";
import { IERC20Burnable } from "../token/interfaces/IERC20Burnable.sol";
import { Token } from "../token/Token.sol";
import { TokenLibrary } from "../token/TokenLibrary.sol";

import { Utils, AccessDenied, NotPayable, InvalidToken } from "../utility/Utils.sol";
```

Here's a brief explanation of each import:

    Address from @openzeppelin/contracts/utils/Address.sol: This is a library provided by the OpenZeppelin framework that provides various utility functions for working with Ethereum addresses.

    IERC20 from @openzeppelin/contracts/token/ERC20/IERC20.sol: This is an interface for the ERC20 token standard, which defines a set of functions that a token contract should implement.

    SafeERC20 from @openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol: This is a library provided by the OpenZeppelin framework that provides a safe way to interact with ERC20 tokens, preventing common errors such as reentrancy attacks and incorrect token transfers.

    ReentrancyGuardUpgradeable from @openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol: This is a contract provided by the OpenZeppelin framework that helps prevent reentrancy attacks by using a mutex lock.

    PausableUpgradeable from @openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol: This is a contract provided by the OpenZeppelin framework that allows the contract owner to pause certain functions in case of emergency.

    ITokenGovernance from @bancor/token-governance/contracts/ITokenGovernance.sol: This is an interface for the governance contract of the Bancor token.

    IVault and ROLE_ASSET_MANAGER from ./interfaces/IVault.sol: These are interfaces defined in the IVault.sol file in the same directory, which define the functions and roles of the Vault smart contract.

    Upgradeable from ../utility/Upgradeable.sol: This is a contract that provides the basic functionality for upgradeable contracts, allowing the contract owner to upgrade the contract logic without losing the state.

    IERC20Burnable from ../token/interfaces/IERC20Burnable.sol: This is an interface for ERC20 tokens that can be burned (destroyed).

    Token and TokenLibrary from ../token/Token.sol: These are contracts that define the implementation of the Token contract, which represents an ERC20 token with added functionality for burning and minting.

    Utils, AccessDenied, NotPayable, and InvalidToken from ../utility/Utils.sol: These are utility contracts that define various error messages and other utility functions used throughout the contract.

Now we define the Vault contract.

```solidity
abstract contract Vault is IVault, Upgradeable, PausableUpgradeable, ReentrancyGuardUpgradeable, Utils {}
```

The abstract keyword means that the Vault contract cannot be instantiated on its own, but needs to be inherited by another contract that will provide the necessary implementation for its abstract functions. An abstract contract is a way to define a contract with unimplemented functions and variables that can be used as a parent contract for other contracts to inherit from and implement the missing functionality.

```solidity
using Address for address payable;
using SafeERC20 for IERC20;
using TokenLibrary for Token;
```

The using keyword is used to bring in functionality from a library contract.

```solidity
// the address of the BNT token
    IERC20 internal immutable _bnt;

    // the address of the BNT token governance
    ITokenGovernance internal immutable _bntGovernance;

    // the address of the vBNT token
    IERC20 internal immutable _vbnt;

    // the address of the vBNT token governance
    ITokenGovernance internal immutable _vbntGovernance;
```

The purpose of each variable declaration is as follows:

    _bnt: This is a variable of type IERC20, which is an interface for the Bancor Network Token (BNT) ERC20 contract.

    _bntGovernance: This is a variable of type ITokenGovernance, which is an interface for the BNT token governance contract.

    _vbnt: This is a variable of type IERC20, which is an interface for the Bancor Vortex (vBNT) ERC20 contract.

    _vbntGovernance: This is a variable of type ITokenGovernance, which is an interface for the vBNT token governance contract.

The variable are marked as internal which means they are only accessible within the contract and immutable, meaning that their value cannot be changed once they are initialized.

```solidity
/**
     * @dev a "virtual" constructor that is only used to set immutable state variables
     */
    constructor(
        ITokenGovernance initBNTGovernance,
        ITokenGovernance initVBNTGovernance
    ) validAddress(address(initBNTGovernance)) validAddress(address(initVBNTGovernance)) {
        _bntGovernance = initBNTGovernance;
        _bnt = initBNTGovernance.token();
        _vbntGovernance = initVBNTGovernance;
        _vbnt = initVBNTGovernance.token();
    }
```

The constructor also has two modifiers: validAddress(address(initBNTGovernance)) and validAddress(address(initVBNTGovernance)). These modifiers ensure that both parameters are valid Ethereum addresses by checking if they are not null. If either of the addresses is null, the constructor will revert with an InvalidAddress() error message, which is defined in the Utils contract. It's a upgradeable contract so we can set modifier in the constructor.

```solidity
function __Vault_init() internal onlyInitializing {
        __Upgradeable_init();
        __Pausable_init();
        __ReentrancyGuard_init();

        __Vault_init_unchained();
    }
```

It is marked with the onlyInitializing modifier, which restricts its execution to occur only during contract initialization. This is achieved by using a boolean flag that is set to true when the contract is being initialized and then set to false when the initialization is complete.

The function itself calls three other initialization functions: **Upgradeable_init(), **Pausable_init(), and \_\_ReentrancyGuard_init(). These functions are inherited from other contracts and perform their respective initialization tasks.

Finally, the function calls \_\_Vault_init_unchained(), which is another internal function that contains the remaining initialization logic specific to the Vault contract. This separation of initialization logic into two functions is a common pattern in upgradeable contracts to allow for easier upgrades and modifications of initialization logic in the future.

```solidity
/**
     * @dev performs contract-specific initialization
     */
    function __Vault_init_unchained() internal onlyInitializing {}

    // solhint-enable func-name-mixedcase

    /**
     * @dev returns the asset manager role
     */
    function roleAssetManager() external pure returns (bytes32) {
        return ROLE_ASSET_MANAGER;
    }

    // allows execution only by an authorized operation
    modifier whenAuthorized(
        address caller,
        Token token,
        address payable target,
        uint256 amount
    ) {
        if (!isAuthorizedWithdrawal(caller, token, target, amount)) {
            revert AccessDenied();
        }

        _;
    }

    /**
     * @dev returns whether withdrawals are currently paused
     */
    function isPaused() external view returns (bool) {
        return paused();
    }

    /**
     * @dev pauses withdrawals
     *
     * requirements:
     *
     * - the caller must have the ROLE_ADMIN privileges
     */
    function pause() external onlyAdmin {
        _pause();
    }

    /**
     * @dev unpauses withdrawals
     *
     * requirements:
     *
     * - the caller must have the ROLE_ADMIN privileges
     */
    function unpause() external onlyAdmin {
        _unpause();
    }
```

The function **Vault_init_unchained() is an internal function that is used for contract-specific initialization. It is called by the **Vault_init() function which is also internal and is only callable during contract initialization. This function calls some other initialization functions inherited from other contracts such as **Upgradeable_init(), **Pausable_init(), and \_\_ReentrancyGuard_init(). The onlyInitializing modifier restricts the access to the function to only be available during initialization.

The roleAssetManager() function is a public function that returns a bytes32 value which represents the asset manager role.

The whenAuthorized() modifier is used to restrict the execution of a function only to authorized operations. It takes four arguments, including the caller's address, a token address, a target address, and a uint256 amount. It reverts with an AccessDenied() error if the isAuthorizedWithdrawal() function returns false.

The isPaused() function is a public function that returns a boolean value indicating whether withdrawals are currently paused.

The pause() function is a public function that pauses withdrawals. It requires that the caller has the ROLE_ADMIN privilege.

The unpause() function is a public function that unpauses withdrawals. It also requires that the caller has the ROLE_ADMIN privilege.
