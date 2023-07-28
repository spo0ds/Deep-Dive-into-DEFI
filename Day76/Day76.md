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

```solidity
/// @inheritdoc ISablierV2Lockup
function getDepositedAmount(uint256 streamId)
    external
    view
    override
    notNull(streamId)
    returns (uint128 depositedAmount)
{
    depositedAmount = _streams[streamId].amounts.deposited;
}
```

This function allows you to retrieve the total amount of ERC20 tokens that were deposited into the specified stream. The deposited amount represents the total quantity of tokens committed to the stream by the sender when creating it.

```solidity
/// @inheritdoc ISablierV2Lockup
function getEndTime(uint256 streamId) external view override notNull(streamId) returns (uint40 endTime) {
    endTime = _streams[streamId].endTime;
}
```

This function retrieves the Unix timestamp indicating when the stream will come to an end.

```solidity
/// @inheritdoc ISablierV2LockupLinear
function getRange(uint256 streamId)
    external
    view
    override
    notNull(streamId)
    returns (LockupLinear.Range memory range)
{
    range = LockupLinear.Range({
        start: _streams[streamId].startTime,
        cliff: _streams[streamId].cliffTime,
        end: _streams[streamId].endTime
    });
}
```

The Range struct returned by this function contains three timestamps: start, cliff, and end.

```solidity
/// @inheritdoc ISablierV2Lockup
function getRefundedAmount(uint256 streamId)
    external
    view
    override
    notNull(streamId)
    returns (uint128 refundedAmount)
{
    refundedAmount = _streams[streamId].amounts.refunded;
}
```

This function retrieves the amount of ERC20 tokens refunded to the sender after a stream has been canceled. The refunded amount will be zero unless the stream was canceled. When a stream is canceled, any unused funds (tokens that were not withdrawn by the recipient) can be refunded back to the sender.

```solidity
/// @inheritdoc ISablierV2Lockup
function getSender(uint256 streamId) external view override notNull(streamId) returns (address sender) {
    sender = _streams[streamId].sender;
}

/// @inheritdoc ISablierV2Lockup
function getStartTime(uint256 streamId) external view override notNull(streamId) returns (uint40 startTime) {
    startTime = _streams[streamId].startTime;
}
```

These functions retrieve the address of the sender who initiated the stream (the one streaming the assets) and the Unix timestamp indicating the start time of the stream, respectively.

```solidity
/// @inheritdoc ISablierV2LockupLinear
function getStream(uint256 streamId)
    external
    view
    override
    notNull(streamId)
    returns (LockupLinear.Stream memory stream)
{
    stream = _streams[streamId];

    // Settled streams cannot be canceled.
    if (_statusOf(streamId) == Lockup.Status.SETTLED) {
        stream.isCancelable = false;
    }
}
```

This function retrieves the complete information about the stream with the specified streamId. Additionally, if the stream is in a "settled" state (completed), the isCancelable flag is set to false since settled streams cannot be canceled.

```solidity
/// @inheritdoc ISablierV2Lockup
function getWithdrawnAmount(uint256 streamId)
    external
    view
    override
    notNull(streamId)
    returns (uint128 withdrawnAmount)
{
    withdrawnAmount = _streams[streamId].amounts.withdrawn;
}
```

The withdrawn amount represents the quantity of tokens the recipient has received up until the current point in time.

```solidity
/// @inheritdoc ISablierV2Lockup
function isCancelable(uint256 streamId) external view override notNull(streamId) returns (bool result) {
    if (_statusOf(streamId) != Lockup.Status.SETTLED) {
        result = _streams[streamId].isCancelable;
    }
}
```

This function checks whether the stream with the given streamId can be canceled. If the stream is in a "settled" state, it cannot be canceled, and isCancelable will be false. However, if the stream is not settled (i.e., it's pending or streaming), then isCancelable will be true if the stream was created with the isCancelable flag set to true, indicating that it can be canceled before the recipient's cliff period ends.

```solidity
/// @inheritdoc ISablierV2Lockup
function isCold(uint256 streamId) external view override notNull(streamId) returns (bool result) {
    Lockup.Status status = _statusOf(streamId);
    result = status == Lockup.Status.SETTLED || status == Lockup.Status.CANCELED || status == Lockup.Status.DEPLETED;
}
```

This function checks whether the stream with the given streamId is either settled, canceled, or depleted. A stream is considered "depleted" when all assets have been withdrawn and/or refunded.

```solidity
/// @inheritdoc ISablierV2Lockup
function isStream(uint256 streamId) public view override(ISablierV2Lockup, SablierV2Lockup) returns (bool result) {
    result = _streams[streamId].isStream;
}
```

This function checks whether the stream with the given streamId exists and holds meaningful data. When isStream is set to true, it signifies that the corresponding Stream struct is valid and contains relevant information. This indicates that a lockup stream has been created, and its details, such as the sender, asset, start time, etc., are populated with relevant values.

```solidity
/// @inheritdoc ISablierV2Lockup
function isWarm(uint256 streamId) external view override notNull(streamId) returns (bool result) {
    Lockup.Status status = _statusOf(streamId);
    result = status == Lockup.Status.PENDING || status == Lockup.Status.STREAMING;
}
```

This function checks whether the stream with the given streamId is either pending or currently streaming. A stream is considered "pending" when the stream's start time is in the future and hasn't started yet. Once the stream starts, it transitions to the "streaming" state, and the recipient can begin to withdraw funds based on the cliff time.

Let's examine the internal function \_statusOf, which determines the status of a stream:

```solidity
/// @inheritdoc SablierV2Lockup
function _statusOf(uint256 streamId) internal view override returns (Lockup.Status) {}
```

This internal function calculates and returns the status of the stream with the given streamId. The status is represented by the Lockup.Status enum, which has different values indicating the state of the stream.

```solidity
if (_streams[streamId].isDepleted) {
    return Lockup.Status.DEPLETED;
} else if (_streams[streamId].wasCanceled) {
    return Lockup.Status.CANCELED;
}
```

The first check determines if the stream has been depleted. If isDepleted is true, it means all the assets in the stream have been withdrawn and/or refunded, resulting in the Lockup.Status.DEPLETED.

The second check verifies if the stream was canceled. If wasCanceled is true, it means the stream has been canceled before it reached its end time, leading to the Lockup.Status.CANCELED.

```solidity
if (block.timestamp < _streams[streamId].startTime) {
    return Lockup.Status.PENDING;
}
```

Next, the function checks if the current time is before the stream's start time. If true, it indicates that the stream is still pending and has not yet started. Thus, it returns the Lockup.Status.PENDING.

```solidity
if (_calculateStreamedAmount(streamId) < _streams[streamId].amounts.deposited) {
    return Lockup.Status.STREAMING;
} else {
    return Lockup.Status.SETTLED;
}
```

The final check determines whether the stream is currently streaming or has already settled. It does this by calculating the total amount of tokens streamed so far using the \_calculateStreamedAmount function. If the streamed amount is less than the total amount deposited into the stream (amounts.deposited), it implies that the stream is still ongoing, resulting in the Lockup.Status.STREAMING.

Conversely, if the streamed amount is equal to or greater than the total deposited amount, it means the stream has already concluded, and the recipient hasn't withdrawn all the funds yet. In this case, the function returns Lockup.Status.SETTLED.

Let's explore the internal function "\_calculateStreamedAmount" in detail.

```solidity
/// @dev Calculates the streamed amount without looking up the stream's status.
function _calculateStreamedAmount(uint256 streamId) internal view returns (uint128) {}
```

This function is used to determine the amount of ERC-20 tokens that have been streamed from a given lockup stream, identified by its unique streamId, up until the current time.

```solidity
// If the cliff time is in the future, return zero.
uint256 cliffTime = uint256(_streams[streamId].cliffTime);
uint256 currentTime = block.timestamp;
if (cliffTime > currentTime) {
return 0;
}
```

The first check ensures that the cliff time (the time when the recipient becomes eligible to start receiving funds from the stream) has not yet passed. If the cliff time is in the future, it means that tokens cannot be streamed before the cliff time, and the function returns 0.

```solidity
// If the end time is not in the future, return the deposited amount.
uint256 endTime = uint256(_streams[streamId].endTime);
if (currentTime >= endTime) {
return _streams[streamId].amounts.deposited;
}
```

Next, it checks if the current time has surpassed the end time of the stream. If true, it means that the entire deposited amount has been streamed, and the function returns the original amounts.deposited.

```solidity
// Calculate how much time has passed since the stream started, and the stream's total duration.
uint256 startTime = uint256(_streams[streamId].startTime);
UD60x18 elapsedTime = ud(currentTime - startTime);
UD60x18 totalTime = ud(endTime - startTime);
```

The code then calculates how much time has passed since the stream started and the total duration of the stream. The values are converted into the custom data type UD60x18, which represents fixed-point numbers with 18 decimal places. This conversion preserves high precision and avoids rounding errors during subsequent calculations.

```solidity
// Divide the elapsed time by the stream's total duration.
UD60x18 elapsedTimePercentage = elapsedTime.div(totalTime);
```

This percentage represents how much of the stream's total duration has passed, which is crucial for calculating the amount of funds that should be streamed at the current moment.

```solidity
// Cast the deposited amount to UD60x18.
UD60x18 depositedAmount = ud(_streams[streamId].amounts.deposited);
```

The deposited amount from the lockup stream is then cast from uint128 to the custom data type UD60x18.

```solidity
// Calculate the streamed amount by multiplying the elapsed time percentage by the deposited amount.
UD60x18 streamedAmount = elapsedTimePercentage.mul(depositedAmount);
```

The calculation of the streamed amount is a key step in determining the current amount of funds that have been streamed and are eligible for withdrawal by the recipient.

```solidity
// Although the streamed amount should never exceed the deposited amount, this condition is checked
// without asserting to avoid locking funds in case of a bug. If this situation occurs, the withdrawn
// amount is considered to be the streamed amount, and the stream is effectively frozen.
if (streamedAmount.gt(depositedAmount)) {
return _streams[streamId].amounts.withdrawn;
}
```

After calculating the streamed amount, the function checks if it exceeds the total deposited amount. In normal circumstances, the streamed amount should never exceed the deposited amount. However, this check is included to safeguard against potential bugs or unexpected behavior. If, for some reason, the streamed amount were to exceed the deposited amount, it would indicate an issue, and to prevent locking funds, the function returns the already withdrawn amount from the lockup stream, effectively freezing the stream.

```solidity
// Cast the streamed amount to uint128. This is safe due to the check above.
return uint128(streamedAmount.intoUint256());
```

Finally, before returning the calculated streamed amount from the function, it is converted back to a regular uint128 to remove the decimal representation and make it compatible with other ERC-20 token interactions.
