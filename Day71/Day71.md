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

```solidity
function withdrawFunds(
        Token token,
        address payable target,
        uint256 amount
    ) external validAddress(target) nonReentrant whenNotPaused whenAuthorized(msg.sender, token, target, amount) {}
```

The function withdraws funds from the Vault contract and sends them to the target address.The function declaration takes three parameters: Token token, which is an instance of the Token contract, address payable target, which is the address to which the funds will be transferred, and uint256 amount, which is the amount of funds to withdraw.The validAddress modifier checks if the target address is a valid Ethereum address.The nonReentrant modifier ensures that the function cannot be called again before the previous call has completed.The whenNotPaused modifier ensures that the function can only be called when the contract is not paused.The whenAuthorized modifier checks whether the caller is authorized to withdraw funds from the contract.

```solidity
function isAuthorizedWithdrawal(
        address caller,
        Token token,
        address target,
        uint256 amount
    ) internal view virtual returns (bool);
```

The isAuthorizedWithdrawal function is just a declaration of a function that takes four parameters: caller, token, target, and amount. This function is intended to be implemented by contracts that extend the Vault contract and inherit from the AccessControl contract.The actual implementation of the isAuthorizedWithdrawal function is left to the inheriting contract that implements it. In this way, the Vault contract can be customized to implement authorization constraints specific to the needs of the application.

For example, if a Vault contract is used to hold funds for a decentralized exchange, the isAuthorizedWithdrawal function might be implemented to check whether the caller is a registered trading bot and whether the target address is a whitelisted liquidity pool contract. If these conditions are not met, the function would return false, and the withdrawal would be denied.

```solidity
if (amount == 0) {
            return;
        }
```

This code block checks if the amount to withdraw is zero. If it is, the function returns immediately without doing anything.

 If the withdraw amount is zero, the function simply returns without doing anything. This does cost some gas.A contract that inherits from the Vault contract might want to implement its own withdraw function that has different behavior when the withdraw amount is zero.
 
 ```solidity
 if (token.isNative()) {
            target.sendValue(amount);
        } else {
            token.safeTransfer(target, amount);
        }
```

This checks if the token to withdraw is a native token, which means it is ether. If it is, the funds are transferred using the sendValue function of the target address, which avoids the gas limit issue that occurs when using a regular transfer function. If it is not a native token, the safeTransfer function of the token contract is called to transfer the funds to the target address.

```solidity
function isNative(Token token) internal pure returns (bool) {
        return address(token) == NATIVE_TOKEN_ADDRESS;
    }
```

```
The reason why a regular transfer for a native token (i.e., ETH) exceeds the gas limit is due to a limitation in the Ethereum Virtual Machine (EVM) called the "2300 gas stipend".

When a contract receives a call from another contract, it is only provided with a limited amount of gas (currently 2300 gas) for execution. This limited gas stipend is intended to prevent malicious contracts from consuming too much gas or executing indefinitely, which could cause the network to become congested or even halt.

A regular ETH transfer involves sending a message to the recipient's address, which triggers a fallback function in the recipient's contract. This fallback function consumes gas, and if it consumes more than 2300 gas, the transaction will revert due to an out-of-gas error.

To work around this limitation, the sendValue method is used instead of the regular transfer or send methods. The sendValue method is a low-level method that uses the call function, which does not have the same gas limits as the regular transfer functions.
```

```solidity
function burn(
        Token token,
        uint256 amount
    ) external nonReentrant whenNotPaused whenAuthorized(msg.sender, token, payable(address(0)), amount) {}
```

It has the nonReentrant and whenNotPaused modifiers applied to it, meaning that it cannot be called again until the previous call has completed and that it can only be called when the contract is not paused. Additionally, it has the whenAuthorized modifier applied, which checks if the caller is authorized to perform the given action (burning the specified amount of the specified token).

```solidity
        if (amount == 0) {
            return;
        }

        if (token.isNative()) {
            revert InvalidToken();
        }
```

This checks if the amount argument is zero and specified token is the native token of the network.If it is, then the function reverts with an InvalidToken error. This is because the Vault contract only supports ERC-20 tokens, so native tokens cannot be burned.

```
The reason the contract reverts if someone tries to burn the native token is because native tokens cannot be burned, i.e., destroyed. Native tokens, such as Ether, are a fundamental part of the Ethereum network, and they cannot be destroyed or removed from circulation.

In the case of the Vault contract, native tokens are handled differently from other ERC20 tokens because they don't have a burn function. Therefore, attempting to burn a native token doesn't make sense and would result in an error, so the contract is designed to revert the transaction and prevent any loss of funds.

On the other hand, users can withdraw native tokens from the Vault, because the contract can simply transfer them to the user's address, without destroying them.
```

The native token address for the Ethereum is `0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE`.

```solidity
        if (token.isEqual(_bnt)) {
            _bntGovernance.burn(amount);
        } else if (token.isEqual(_vbnt)) {
            _vbntGovernance.burn(amount);
        } else {
            IERC20Burnable(address(token)).burn(amount);
        }
```

First checks if the specified token is equal to the _bnt or _vbnt tokens. These are special tokens that have their own governance modules, which are contracts responsible for managing the supply and burning of these tokens. If the token is equal to _bnt or _vbnt, then the corresponding governance module's burn function is called with the specified amount.

If the token is not equal to _bnt or _vbnt, then the burn function of the IERC20Burnable interface is called on the token instance with the specified amount. This assumes that the specified token implements the IERC20Burnable interface, which is an extension of the IERC20 interface that adds a burn function to the standard ERC-20 interface.

```solidity
function isPayable() public view virtual returns (bool);
```

The isPayable function returns a boolean value indicating whether the contract can receive Ether or not.The purpose of the isPayable function is to allow a contract to indicate whether or not it can receive native tokens, such as Ether. If a contract wants to receive native tokens, it should override the isPayable function to return true. If the contract does not want to receive native tokens, it should override the function to return false.

```solidity
receive() external payable {
        if (!isPayable()) {
            revert NotPayable();
        }
    }
```

The receive function is a fallback function that is triggered when someone tries to send Ether to the contract without specifying a function to call. The receive function checks whether the contract is allowed to receive Ether by calling the isPayable function. If the isPayable function returns false, the receive function reverts with an error message. If the isPayable function returns true, the receive function accepts the Ether and does nothing with it.

## MasterVault.sol

The Master Vault contract is the main contract for managing the funds of the Bancor protocol and ensuring that only authorized parties can access and manage those funds.It inherits from the Vault contract and implements the IMasterVault interface, which defines the functions that can be used to manage the funds held by the Master Vault.

IVersioned: This import is an interface from the utility/interfaces/IVersioned.sol file and is used by the MasterVault contract to provide a version number for the contract.

ROLE_ASSET_MANAGER: This is a constant defined in the interfaces/IVault.sol file and is used as a role for the MasterVault contract.

```solidity
// the asset manager role is required to access all the funds
bytes32 constant ROLE_ASSET_MANAGER = keccak256("ROLE_ASSET_MANAGER");
```

```
No, the fact that a role is required to access funds does not necessarily mean that Bancor is centralized.In the case of Bancor, the role-based access control is used to manage access to the BNT reserve, which is a critical component of the Bancor ecosystem. By controlling access to the reserve, Bancor can ensure that only authorized parties are able to interact with the reserve and that the reserve is protected from unauthorized withdrawals or misuse. 
```

`What's the difference between Uniswap and Bancor based on role based access mechanism?`

```
Uniswap, being a completely decentralized protocol, does not have a role-based access control mechanism. Instead, anyone with an Ethereum wallet can interact with the protocol and use its functions, such as swapping tokens or providing liquidity. There is no privileged access to the protocol's functions or funds, and everything is done through smart contracts that execute autonomously on the blockchain.

On the other hand, Bancor has a more centralized approach, where certain roles have specific privileges and permissions to access and manage the funds of the protocol. For example, the Asset Manager role in Bancor has the authority to manage and withdraw funds from the protocol's vault contract. Bancor's approach can be considered more centralized compared to Uniswap, as the protocol has specific roles and permissions that can access its functions and funds. However, it's worth noting that Bancor's governance model is constantly evolving, and the protocol has been working to become more decentralized over time.
```

```
The role-based access manager ensures that only authorized parties are able to withdraw funds from the reserve, such as the asset manager or the BNT manager. So, in theory, your deposited funds cannot be utilized by the role-based manager without your explicit consent.

That being said, it's important to note that all smart contracts are executed on the blockchain, which is a public ledger. This means that all transactions, including those involving the reserve, are publicly visible and cannot be reversed or modified. So, while the role-based access manager helps to ensure that only authorized parties can interact with the reserve, it's ultimately up to the user to determine whether they trust the protocol and its administrators to manage their funds appropriately.
```

```solidity
contract MasterVault is IMasterVault, Vault {}
```

MasterVault implements all of the functions and events defined in both of these interfaces.

```solidity
// the BNT manager role is only required to access the BNT reserve
bytes32 private constant ROLE_BNT_MANAGER = keccak256("ROLE_BNT_MANAGER");
```

The constant is used to define a role-based access control mechanism where certain functions can only be called by an account that has been granted the ROLE_BNT_MANAGER role. Specifically, the comment suggests that the ROLE_BNT_MANAGER role is only required to access the BNT reserve.

`What's the difference between "ROLE_ASSET_MANAGER" and "ROLE_BNT_MANAGER"?`

"ROLE_ASSET_MANAGER" and "ROLE_BNT_MANAGER" are two different roles defined in the MasterVault contract.

The "ROLE_ASSET_MANAGER" role is used to manage all other assets held by the MasterVault contract besides the BNT token. The holder of this role can deposit and withdraw any other asset held by the contract.

On the other hand, the "ROLE_BNT_MANAGER" role is used specifically to manage the BNT token held by the contract. The holder of this role can deposit and withdraw BNT tokens, as well as perform other operations that involve the BNT reserve, such as minting or burning the Bancor Network Token (BNT) in exchange for other assets.

```solidity
// upgrade forward-compatibility storage gap
uint256[MAX_GAP - 0] private __gap;
```

It is a technique used to ensure forward-compatibility of smart contracts, which is called "Upgradeability". The upgradeability approach allows smart contracts to be updated in a way that does not break their original functionality or require the creation of a new smart contract.

The uint256[MAX_GAP - 0] private __gap; line of code is creating a storage gap, which is an unused storage variable that can be used to add new variables to the smart contract in future upgrades without overwriting existing storage variables.

`MAX_GAP` is define in Upgradeable.sol.

```solidity
uint32 internal constant MAX_GAP = 50;
```

 By reserving this storage gap, the smart contract is allowing for the addition of new storage variables in future upgrades without breaking the existing functionality or overwriting existing storage variables.
 
 `Doesn't this cost more gas?`
 
 Declaring a large array in a contract can have some gas cost associated with it, but in the case of using the array to create a storage gap for forward compatibility, the gas cost is a one-time expense that is only incurred during contract deployment.
 
 `How does it provides the storage slot for upgrades? `
 
 When a contract is deployed, the EVM (Ethereum Virtual Machine) reserves a continuous block of storage slots for that contract. The contract can then store data in these reserved storage slots as it needs.

When the contract is upgraded, the upgraded contract might need to store additional data, or might need to modify the existing data structures. To prevent the upgraded contract from overwriting the data already stored by the original contract, a "gap" is left between the original data structures and the new data structures. The gap is left empty so that the new data structures can be added without overwriting the old ones.

To reserve the gap, the contract includes a private storage array with a large number of unused elements. These unused elements are essentially "dead" storage slots that can be used as a gap. Since the unused elements are not being used by the contract, the gap will not overwrite any existing data when the upgraded contract is deployed.

This method of reserving storage slots is gas-efficient, since the EVM only charges for the amount of storage actually used by the contract. The unused storage slots in the gap are not charged for, since they are never used by the contract.

```solidity
constructor(
        ITokenGovernance initBNTGovernance,
        ITokenGovernance initVBNTGovernance
    ) Vault(initBNTGovernance, initVBNTGovernance) {}
```

The constructor of the MasterVault contract initializes the parent Vault contract by calling its constructor with two arguments, initBNTGovernance and initVBNTGovernance.

```solidity
/**
     * @dev fully initializes the contract and its parents
     */
    function initialize() external initializer {
        __MasterVault_init();
    }

    // solhint-disable func-name-mixedcase

    /**
     * @dev initializes the contract and its parents
     */
    function __MasterVault_init() internal onlyInitializing {
        __Vault_init();

        __MasterVault_init_unchained();
    }

    /**
     * @dev performs contract-specific initialization
     */
    function __MasterVault_init_unchained() internal onlyInitializing {
        // set up administrative roles
        _setRoleAdmin(ROLE_ASSET_MANAGER, ROLE_ADMIN);
        _setRoleAdmin(ROLE_BNT_MANAGER, ROLE_ADMIN);
    }

```

    initialize(): This is an external function that calls the __MasterVault_init() function to fully initialize the contract and its parents.

    __MasterVault_init(): This is an internal function that calls the __Vault_init() function to initialize the parent Vault contract, and then calls __MasterVault_init_unchained() to perform contract-specific initialization.

    __MasterVault_init_unchained(): This is an internal function that sets up the administrative roles for the contract using _setRoleAdmin(). Specifically, it sets the role admin for ROLE_ASSET_MANAGER and ROLE_BNT_MANAGER to be ROLE_ADMIN.

Overall, the purpose of these functions is to ensure that the MasterVault contract is fully initialized and set up with the appropriate administrative roles.

```solidity
/**
     * @inheritdoc Upgradeable
     */
    function version() public pure override(IVersioned, Upgradeable) returns (uint16) {
        return 1;
    }

    /**
     * @inheritdoc Vault
     */
    function isPayable() public pure override(IVault, Vault) returns (bool) {
        return true;
    }

    /**
     * @dev returns the BNT manager role
     */
    function roleBNTManager() external pure returns (bytes32) {
        return ROLE_BNT_MANAGER;
    }
```

    version(): This function overrides the version() function in both the IVersioned and Upgradeable interfaces. It returns a uint16 indicating the version of the contract. This is used to track upgrades to the contract over time.

    isPayable(): This function overrides the isPayable() function in both the IVault and Vault contracts. It returns a boolean value indicating whether the contract can receive ETH payments.

    roleBNTManager(): This function returns the bytes32 value of the ROLE_BNT_MANAGER constant. It is an external function, which means it can be called from outside the contract, and it is marked as pure because it does not modify the state of the contract. The purpose of this function is to provide a way for other contracts to access the ROLE_BNT_MANAGER value.
    
```solidity
function isAuthorizedWithdrawal(
        address caller,
        Token token,
        address /* target */,
        uint256 /* amount */
    ) internal view override returns (bool) {
        return (token.isEqual(_bnt) && hasRole(ROLE_BNT_MANAGER, caller)) || hasRole(ROLE_ASSET_MANAGER, caller);
    }
```

It is overriding the isAuthorizedWithdrawal() function defined in the Vault contract and is used to implement role-based access control for withdrawals.

The function first checks if the token being withdrawn is the Bancor Network Token (BNT) by calling the isEqual() function of the Token struct and comparing it with the _bnt variable, which represents the BNT token instance. If the token is the BNT token, it then checks if the caller has the ROLE_BNT_MANAGER or ROLE_ASSET_MANAGER role by calling the hasRole() function inherited from the AccessControlUpgradeable contract. If the token is not the BNT token, it checks if the caller has the ROLE_ASSET_MANAGER role.

If the caller has the required role, the function returns true, indicating that the withdrawal is authorized. If the caller does not have the required role, the function returns false, indicating that the withdrawal is not authorized.
