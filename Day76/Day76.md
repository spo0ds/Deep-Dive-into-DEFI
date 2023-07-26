## Sablier Protocol

The Sablier protocol is a collection of enduring, non-upgradable smart contracts designed to enable seamless streaming of ERC-20 assets on both Ethereum and other EVM blockchains. Sablier V2 is a binary smart contract system, comprising several contracts, libraries, and types that collectively form the Core and Periphery components.

- Core provides the fundamental streaming logic of the Sablier V2 Protocol. It contains LockupLinear and LockupDynamic, which are the primary contracts that users will interact with.
- Periphery contracts interact with one or more Core contracts but are not part of the Core. They are an abstraction layer that enhance the security and the extensibility of the protocol without introducing upgradeability.

**Core Contracts**

The Core component of Sablier V2 includes the streaming contracts "LockupLinear" and "LockupDynamic," an NFT descriptor, and the Comptroller (an on-chain configuration module).

**LockupLinear**

LockupLinear is a contract responsible for creating and managing Lockup streams with a linear streaming function.

```solidity
/// @title SablierV2LockupLinear
/// @notice See the documentation in {ISablierV2LockupLinear}.
contract SablierV2LockupLinear is
    ISablierV2LockupLinear, // 5 inherited components
    SablierV2Lockup // 14 inherited components
{}
```

It inherits from the ISablierV2LockupLinear interface, which specifies a set of external function signatures that the contract must implement to conform to the interface. Additionally, LockupLinear inherits from the SablierV2Lockup contract, which contains the core functionality of the Sablier protocol for creating and managing lockup streams. It serves as the foundational component that SablierV2LockupLinear builds upon.

```solidity
using SafeERC20 for IERC20;
```

To enable safe ERC20 token transfers, the contract uses the SafeERC20 library, which ensures secure operations when dealing with ERC20 tokens.

```solidity
/// @dev Sablier V2 Lockup Linear streams mapped by unsigned integers.
mapping(uint256 id => LockupLinear.Stream stream) private _streams;
```

Streams created by LockupLinear are mapped by unsigned integers using the \_streams mapping, which holds detailed information about each stream. Each stream is represented by the Stream struct, which contains various attributes related to the streaming process. Some of the key attributes include:

```solidity
struct Stream {
        // slot 0
        address sender;
        uint40 startTime;
        uint40 cliffTime;
        bool isCancelable;
        bool wasCanceled;
        // slot 1
        IERC20 asset;
        uint40 endTime;
        bool isDepleted;
        bool isStream;
        // slot 2 and 3
        Lockup.Amounts amounts;
    }
```

cliffTime: The Unix timestamp indicating when the recipient becomes eligible to start receiving funds from the stream (the cliff period's end).

```solidity
/// @dev Emits a {TransferAdmin} event.
    /// @param initialAdmin The address of the initial contract admin.
    /// @param initialComptroller The address of the initial comptroller.
    /// @param initialNFTDescriptor The address of the initial NFT descriptor.
    constructor(
        address initialAdmin,
        ISablierV2Comptroller initialComptroller,
        ISablierV2NFTDescriptor initialNFTDescriptor
    )
        ERC721("Sablier V2 Lockup Linear NFT", "SAB-V2-LOCKUP-LIN")
        SablierV2Lockup(initialAdmin, initialComptroller, initialNFTDescriptor)
    {
        nextStreamId = 1;
    }
```

The constructor of the SablierV2LockupLinear contract sets the initial NFT name and symbol, admin address, comptroller, and NFT descriptor. It also initializes the nextStreamId variable to 1, indicating that the first lockup stream created will have an ID of 1.

```solidity
 /// @inheritdoc ISablierV2Lockup
    function getAsset(uint256 streamId) external view override notNull(streamId) returns (IERC20 asset) {
        asset = _streams[streamId].asset;
    }
```

Retrieves the address of the ERC-20 asset used for streaming.

```solidity
/// @inheritdoc ISablierV2LockupLinear
    function getCliffTime(uint256 streamId) external view override notNull(streamId) returns (uint40 cliffTime) {
        cliffTime = _streams[streamId].cliffTime;
    }
```

Retrieves the time when recipient will be eligible to receive the fund.
