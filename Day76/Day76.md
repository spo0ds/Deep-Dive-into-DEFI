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

```solidity
/// @inheritdoc ISablierV2Lockup
function refundableAmountOf(uint256 streamId) external view override notNull(streamId) returns (uint128 refundableAmount) {}
```

This function calculates the amount that the sender would be refunded if the stream were canceled.

```solidity
// These checks are needed because {_calculateStreamedAmount} does not look up the stream's status. Note that
// checking for `isCancelable` also checks if the stream `wasCanceled` thanks to the protocol invariant that
// canceled streams are not cancelable anymore.
if (_streams[streamId].isCancelable && !_streams[streamId].isDepleted) {
    refundableAmount = _streams[streamId].amounts.deposited - _calculateStreamedAmount(streamId);
}
```

The function first checks if the stream is cancelable and not depleted. If both conditions are met, it calculates the refundable amount by subtracting the already streamed amount from the total deposited amount.

```solidity
/// @inheritdoc ISablierV2Lockup
function statusOf(uint256 streamId) external view override notNull(streamId) returns (Lockup.Status status) {
    status = _statusOf(streamId);
}
```

This function retrieves the status of the specified stream using the internal function \_statusOf.

```solidity
/// @inheritdoc ISablierV2LockupLinear
function streamedAmountOf(uint256 streamId) public view override(ISablierV2Lockup, ISablierV2LockupLinear) notNull(streamId) returns (uint128 streamedAmount) {
    streamedAmount = _streamedAmountOf(streamId);
}
```

This function retrieves the streamed amount for the recipient from the specified stream using the internal function \_streamedAmountOf.

```solidity
/// @inheritdoc ISablierV2Lockup
function wasCanceled(uint256 streamId) public view override(ISablierV2Lockup, SablierV2Lockup) notNull(streamId) returns (bool result) {
    result = _streams[streamId].wasCanceled;
}
```

This function checks whether the specified stream was canceled.

```solidity
/// @inheritdoc ISablierV2LockupLinear
function createWithDurations(LockupLinear.CreateWithDurations calldata params) external override noDelegateCall returns (uint256 streamId) {}
```

This function creates a new linear stream with the specified parameters.

```solidity
// Set the current block timestamp as the stream's start time.
LockupLinear.Range memory range;
range.start = uint40(block.timestamp);
```

The function starts by setting the current block timestamp as the start time of the stream.

```solidity
// Calculate the cliff time and the end time. It is safe to use unchecked arithmetic because
// {_createWithRange} will nonetheless check that the end time is greater than the cliff time,
// and also that the cliff time is greater than or equal to the start time.
unchecked {
    range.cliff = range.start + params.durations.cliff;
    range.end = range.start + params.durations.total;
}
```

It calculates the cliff time and the end time of the stream. It does this by adding the specified cliff duration and the total duration to the start time, respectively. The use of unchecked arithmetic is safe because the \_createWithRange function will perform additional checks to ensure the correctness of the calculated times.

```solidity
// Checks, Effects and Interactions: create the stream.
streamId = _createWithRange(
    LockupLinear.CreateWithRange({
        asset: params.asset,
        broker: params.broker,
        cancelable: params.cancelable,
        range: range,
        recipient: params.recipient,
        sender: params.sender,
        totalAmount: params.totalAmount
    })
);
```

The function then calls the internal function \_createWithRange to create the linear stream with the specified range and other parameters.

Now let's explore what \_createWithRange does.

```solidity
/// @dev See the documentation for the user-facing functions that call this internal function.
function _createWithRange(LockupLinear.CreateWithRange memory params) internal returns (uint256 streamId) {}
```

This internal function is responsible for creating a linear stream with the provided range.

```solidity
// Safe Interactions: query the protocol fee. This is safe because it's a known Sablier contract that does
// not call other unknown contracts.
UD60x18 protocolFee = comptroller.protocolFees(params.asset);
```

It starts by querying the protocol fee for the specified asset. The protocol fee is retrieved from the comptroller contract and represents the percentage of the total amount that will be deducted as a fee to support and sustain the operation and development of the Sablier protocol.

```solidity
// Checks: check the fees and calculate the fee amounts.
Lockup.CreateAmounts memory createAmounts =
            Helpers.checkAndCalculateFees(params.totalAmount, protocolFee, params.broker.fee, MAX_FEE);
```

Next, the function calls the helper function Helpers.checkAndCalculateFees to validate the fees and calculate the actual fee amounts.The function ensures that the protocol fee and broker fee do not exceed the maximum allowable fee (MAX_FEE). Then, it calculates the protocol fee amount and the broker fee amount based on the total amount and the fee percentages.

The protocol fee is a fee that is collected by the Sablier protocol itself. It is deducted from the total amount of ERC-20 tokens being streamed in a lockup stream. The protocol fee is intended to support and sustain the operation and development of the Sablier protocol.

The broker fee is an additional fee that can be charged by a third-party broker. Unlike the protocol fee, the broker fee is not collected by the Sablier protocol itself but rather by an external entity acting as a broker. The broker fee is also deducted from the total amount of ERC-20 tokens being streamed, similar to the protocol fee. However, the broker fee is optional and can be customized by the broker, allowing them to charge a fee for their services in facilitating the creation and management of lockup streams.

Now we'll go through checkAndCalculateFees and come back to \_createWithRange.

```solidity
// When the total amount is zero, the fees are also zero.
if (totalAmount == 0) {
    return Lockup.CreateAmounts(0, 0, 0);
}
```

If the totalAmount is zero, it means there are no funds to be streamed, so the function returns all fees as zero.

```solidity
// Checks: the protocol fee is not greater than `maxFee`.
if (protocolFee.gt(maxFee)) {
    revert Errors.SablierV2Lockup_ProtocolFeeTooHigh(protocolFee, maxFee);
}
// Checks: the broker fee is not greater than `maxFee`.
if (brokerFee.gt(maxFee)) {
    revert Errors.SablierV2Lockup_BrokerFeeTooHigh(brokerFee, maxFee);
}
```

It checks whether both the protocolFee and brokerFee are greater than the maxFee. If either of the fees exceeds this maximum, the function will revert with an error message indicating that the fee is too high.

```solidity
// Calculate the protocol fee amount.
// The cast to uint128 is safe because the maximum fee is hard coded.
amounts.protocolFee = uint128(ud(totalAmount).mul(protocolFee).intoUint256());

// Calculate the broker fee amount.
// The cast to uint128 is safe because the maximum fee is hard coded.
amounts.brokerFee = uint128(ud(totalAmount).mul(brokerFee).intoUint256());
```

Next, the function calculates the actual amounts for both the protocolFee and brokerFee. It does this by first converting the totalAmount to a fixed-point number using the ud() function, which preserves high precision. Then, it multiplies this fixed-point representation of the total amount with the respective fee percentages protocolFee and brokerFee. The result is then converted back to a regular uint128 data type, effectively removing the decimal representation. This calculated fee amount is then stored in the amounts.protocolFee and amounts.brokerFee variables.

```solidity
// Assert that the total amount is strictly greater than the sum of the protocol fee amount and the
// broker fee amount.
assert(totalAmount > amounts.protocolFee + amounts.brokerFee);
```

After calculating the fees, the function ensures that the totalAmount is strictly greater than the sum of the protocolFee and brokerFee. This assertion is essential to guarantee that there are sufficient funds left in the total amount after deducting both fees.

```solidity
// Calculate the deposit amount (the amount to stream, net of fees).
amounts.deposit = totalAmount - amounts.protocolFee - amounts.brokerFee;
```

Finally, the function calculates the actual deposit amount that will be streamed, net of fees. It subtracts the protocolFee and brokerFee amounts from the totalAmount, leaving only the actual deposit amount to be stored in the amounts.deposit variable. This deposit amount represents the funds that will be available for streaming in the lockup stream after deducting the protocol and broker fees.

In the Sablier protocol, the protocol fee is optional and can be set to 0 if the user chooses not to specify a protocol fee.

Now back to the \_createWithRange function.

```solidity
// Checks: validate the user-provided parameters.
Helpers.checkCreateWithRange(createAmounts.deposit, params.range);
```

The function then calls the helper function Helpers.checkCreateWithRange to validate the user-provided parameters, such as the deposit amount and the range (start time, cliff time, and end time) of the stream.

Let's explore

``solidity
// Checks: the deposit amount is not zero.
if (depositAmount == 0) {
revert Errors.SablierV2Lockup_DepositAmountZero();
}

````

Deposit amount can't be zero.

```solidity
    // Checks: the start time is less than or equal to the cliff time.
        if (range.start > range.cliff) {
            revert Errors.SablierV2LockupLinear_StartTimeGreaterThanCliffTime(range.start, range.cliff);
        }
````

Start time can't be greater than cliff time.

```solidity
         // Checks: the cliff time is strictly less than the end time.
        if (range.cliff >= range.end) {
            revert Errors.SablierV2LockupLinear_CliffTimeNotLessThanEndTime(range.cliff, range.end);
        }
```

Cliff time must not be greater than end time.

```solidity
        // Checks: the end time is in the future.
        uint40 currentTime = uint40(block.timestamp);
        if (currentTime >= range.end) {
            revert Errors.SablierV2Lockup_EndTimeNotInTheFuture(currentTime, range.end);
        }
```

End time must be somewhere in the future.

```solidity
// Load the stream id.
streamId = nextStreamId;
```

We load the next stream id.

```solidity
// Effects: create the stream.
_streams[streamId] = LockupLinear.Stream({
    amounts: Lockup.Amounts({ deposited: createAmounts.deposit, refunded: 0, withdrawn: 0 }),
    asset: params.asset,
    cliffTime: params.range.cliff,
    endTime: params.range.end,
    isCancelable: params.cancelable,
    isDepleted: false,
    isStream: true,
    sender: params.sender,
    startTime: params.range.start,
    wasCanceled: false
});
```

Next, the function creates the linear stream by initializing a new LockupLinear.Stream struct with the provided parameters. The struct contains information about the stream, such as the deposited amount, asset, cliff time, end time, sender, and recipient.

```solidity
// Effects: bump the next stream id and record the protocol fee.
// Using unchecked arithmetic because these calculations cannot realistically overflow, ever.
unchecked {
    nextStreamId = streamId + 1;
    protocolRevenues[params.asset] = protocolRevenues[params.asset] + createAmounts.protocolFee;
}
```

After creating the stream, the function updates the nextStreamId to ensure that the next stream will have a unique identifier. Additionally, it records the protocol fee amount in the protocolRevenues mapping, associating it with the corresponding asset.protocolRevenues is the mapping defined in SablierV2Base abstract contract.

```solidity
// Effects: mint the NFT to the recipient.
_mint({ to: params.recipient, tokenId: streamId });
```

The function then mints an ERC-721 token (NFT) representing the right to receive funds from the stream and assigns it to the recipient.

```solidity
// Interactions: transfer the deposit and the protocol fee.
// Using unchecked arithmetic because the deposit and the protocol fee are bounded by the total amount.
unchecked {
    params.asset.safeTransferFrom({
        from: msg.sender,
        to: address(this),
        value: createAmounts.deposit + createAmounts.protocolFee
    });
}
```

Finally, the function transfers the total deposit amount and protocol fee from the sender to the contract's address. This ensures that the contract holds the funds and manages the streaming process.

```solidity
// Interactions: pay the broker fee, if not zero.
if (createAmounts.brokerFee > 0) {
    params.asset.safeTransferFrom({ from: msg.sender, to: params.broker.account, value: createAmounts.brokerFee });
}
```

Additionally, if a broker fee is specified and greater than zero, the function transfers the broker fee from the sender to the broker's account.

```solidity
// Log the newly created stream.
emit ISablierV2LockupLinear.CreateLockupLinearStream({
    streamId: streamId,
    funder: msg.sender,
    sender: params.sender,
    recipient: params.recipient,
    amounts: createAmounts,
    asset: params.asset,
    cancelable: params.cancelable,
    range: params.range,
    broker: params.broker.account
});
```

The function emits an event to log the creation of the new linear stream, providing information about the stream, such as the stream ID, sender, recipient, amounts, asset, and other relevant details.

```solidity
   /// @inheritdoc SablierV2Lockup
    function _isCallerStreamSender(uint256 streamId) internal view override returns (bool) {
        return msg.sender == _streams[streamId].sender;
    }
```

Returns whether the caller is the stream sender or not.

```solidity
/// @dev See the documentation for the user-facing functions that call this internal function.
    function _streamedAmountOf(uint256 streamId) internal view returns (uint128) {}
```

This function determine the total amount of ERC-20 tokens that have been streamed from a specific lockup stream (identified by its streamId) up until the current moment.

```solidity
Lockup.Amounts memory amounts = _streams[streamId].amounts;
```

This struct contains information about the total deposited amount, total withdrawn amount, and total refunded amount for the stream.

```solidity
 if (_streams[streamId].isDepleted) {
            return amounts.withdrawn;
        } else if (_streams[streamId].wasCanceled) {
            return amounts.deposited - amounts.refunded;
        }
```

If the stream is marked as isDepleted, it means that all funds have been withdrawn from the stream. In this case, the function returns the amounts.withdrawn, which represents the total amount of funds that have been withdrawn from the stream.

If the stream is marked as wasCanceled, it means that the stream was canceled before being fully depleted. In this case, the function returns the difference between the amounts.deposited (total amount deposited) and the amounts.refunded (total amount refunded). This difference represents the total amount of funds that were not refunded before the stream was canceled.

```solidity
return _calculateStreamedAmount(streamId);
```

Returns the ERC-20 tokens that have been streamed from a given lockup stream (identified by its streamId) up until the current moment.

```solidity
/// @dev See the documentation for the user-facing functions that call this internal function.
    function _withdrawableAmountOf(uint256 streamId) internal view override returns (uint128) {
        return _streamedAmountOf(streamId) - _streams[streamId].amounts.withdrawn;
    }
```

Returns the amount of ERC-20 tokens that can be withdrawn by the recipient from a given lockup stream (identified by its streamId).It calls the \_streamedAmountOf function, which returns the total amount of funds that have been streamed from the lockup stream up until the current moment.It then subtracts the amounts.withdrawn value from the stream. This amounts.withdrawn represents the total amount of funds that have already been withdrawn by the recipient from the stream.

The result represents the amount of funds that are currently available for withdrawal by the recipient.

```solidity
function _cancel(uint256 streamId) internal override {}
```

This is an internal function used to cancel a lockup stream identified by streamId.

```solidity
// Calculate the streamed amount.
uint128 streamedAmount = _calculateStreamedAmount(streamId);
```

It first calculates the amount of tokens that have been streamed from the lockup stream using the \_calculateStreamedAmount function.\_calculateStreamedAmount returns the total amount of tokens streamed so far.

```solidity
// Retrieve the amounts from storage.
Lockup.Amounts memory amounts = _streams[streamId].amounts;
```

it retrieves the relevant Amounts struct associated with the given streamId from storage.

```solidity
// Checks: the stream is not settled.
if (streamedAmount >= amounts.deposited) {
    revert Errors.SablierV2Lockup_StreamSettled(streamId);
}
```

it means the stream has already been settled, and the function reverts with an error.

```solidity
// Checks: the stream is cancelable.
if (!_streams[streamId].isCancelable) {
    revert Errors.SablierV2Lockup_StreamNotCancelable(streamId);
}
```

If the stream is not marked as cancelable, the function reverts with an error using the StreamNotCancelable error message.

```solidity
// Calculate the sender's and the recipient's amount.
uint128 senderAmount = amounts.deposited - streamedAmount;
uint128 recipientAmount = streamedAmount - amounts.withdrawn;
```

Next, it calculates the remaining amounts for the sender and the recipient. The sender's amount is the total amount deposited minus the streamed amount, and the recipient's amount is the streamed amount minus the already withdrawn amount.

```solidity
// Effects: mark the stream as canceled.
_streams[streamId].wasCanceled = true;
```

The function marks the stream as canceled.

```solidity
// Effects: make the stream not cancelable anymore, because a stream can only be canceled once.
_streams[streamId].isCancelable = false;
```

It also makes the stream not cancelable anymore.A stream can only be canceled once.

```solidity
// Effects: If there are no assets left for the recipient to withdraw, mark the stream as depleted.
if (recipientAmount == 0) {
    _streams[streamId].isDepleted = true;
}
```

If the recipient has already withdrawn all their funds (i.e., recipientAmount is zero), the function marks the stream as depleted.

```solidity
// Effects: set the refunded amount.
_streams[streamId].amounts.refunded = senderAmount;
```

It sets the refunded field of the Amounts struct to the remaining amount to be refunded to the sender.

```solidity
// Retrieve the sender and the recipient from storage.
address sender = _streams[streamId].sender;
address recipient = _ownerOf(streamId);
```

The function retrieves the sender and recipient addresses associated with the given streamId from storage.

```solidity
// Interactions: refund the sender.
_streams[streamId].asset.safeTransfer({ to: sender, value: senderAmount });
```

This transfers the remaining amount back to the sender's address.

```solidity
    // Interactions: if `msg.sender` is the sender and the recipient is a contract, try to invoke the cancel
    // hook on the recipient without reverting if the hook is not implemented, and without bubbling up any
    // potential revert.
    if (msg.sender == sender) {
        if (recipient.code.length > 0) {
            try ISablierV2LockupRecipient(recipient).onStreamCanceled({
                streamId: streamId,
                sender: sender,
                senderAmount: senderAmount,
                recipientAmount: recipientAmount
            }) { } catch { }
        }
    }
    // Interactions: if `msg.sender` is the recipient and the sender is a contract, try to invoke the cancel
    // hook on the sender without reverting if the hook is not implemented, and also without bubbling up any
    // potential revert.
    else {
        if (sender.code.length > 0) {
            try ISablierV2LockupSender(sender).onStreamCanceled({
                streamId: streamId,
                recipient: recipient,
                senderAmount: senderAmount,
                recipientAmount: recipientAmount
            }) { } catch { }
        }
    }
```

Depending on who called the cancel function (either the sender or the recipient), it checks if the other party is a smart contract (i.e., has non-zero code.length). If so, it tries to invoke the onStreamCanceled hook on the respective smart contract without reverting if the hook is not implemented and without bubbling up any potential revert from the hook.

```solidity
// Log the cancellation.
emit ISablierV2Lockup.CancelLockupStream(streamId, sender, recipient, senderAmount, recipientAmount);
```

Finally, it emits an event to log the cancellation of the stream.

> Hooks in Solidity are a way to allow contracts to interact with each other in a flexible and decentralized manner. They provide a mechanism for contract A to notify contract B about certain events or conditions and allow contract B to take specific actions in response to those events.

> In the Sablier Linear contract, when a stream is canceled, it checks if the sender or recipient of the stream is a smart contract (i.e., has non-zero code.length).If the sender or recipient is a smart contract, the contract tries to invoke the onStreamCanceled hook on the respective smart contract.Invoking the hook is done using a low-level try-catch approach. This means that the contract tries to call the hook, and if the hook is not implemented or reverts, it does not cause the whole transaction to revert. Instead, any potential revert from the hook is caught, and the rest of the contract execution continues without interruption.The invoked hook, such as onStreamCanceled, contains custom logic defined by the smart contract implementer. For example, it could be used to trigger additional actions, such as updating balances, recording events, or even interacting with other smart contracts.

```solidity
function _renounce(uint256 streamId) internal override {}
```

This function removes the right of the stream's sender to cancel the stream. It is used to prevent the sender from canceling the stream once it has been created.

```solidity
// Checks: the stream is cancelable.
if (!_streams[streamId].isCancelable) {
    revert Errors.SablierV2Lockup_StreamNotCancelable(streamId);
}
```

Before renouncing the stream, the function first checks if the stream is cancelable. If the stream is not cancelable, it means that it cannot be canceled, and the function reverts with an error using the StreamNotCancelable error message.

```solidity
// Effects: renounce the stream by making it not cancelable.
_streams[streamId].isCancelable = false;
```

If the stream is cancelable, the function proceeds to mark the stream as not cancelable.

```solidity
// Interactions: if the recipient is a contract, try to invoke the renounce hook on the recipient without
// reverting if the hook is not implemented, and also without bubbling up any potential revert.
address recipient = _ownerOf(streamId);
if (recipient.code.length > 0) {
    try ISablierV2LockupRecipient(recipient).onStreamRenounced(streamId) { } catch { }
}
```

Next, the function interacts with the recipient of the lockup stream. It checks if the recipient is a smart contract by verifying if the recipient's code.length is greater than zero. If the recipient is a smart contract, it tries to invoke the onStreamRenounced hook on the recipient without reverting if the hook is not implemented or if it reverts.

```solidity
// Log the renouncement.
emit ISablierV2Lockup.RenounceLockupStream(streamId);
```

This event contains information about the streamId that was renounced.

```solidity
function _withdraw(uint256 streamId, address to, uint128 amount) internal override {}
```

This function is used to withdraw funds from a lockup stream identified by its streamId. The sender can withdraw funds from the stream and transfer them to the specified to address.

```solidity
// Checks: the withdraw amount is not greater than the withdrawable amount.
uint128 withdrawableAmount = _withdrawableAmountOf(streamId);
if (amount > withdrawableAmount) {
    revert Errors.SablierV2Lockup_Overdraw(streamId, amount, withdrawableAmount);
}
```

Before performing the withdrawal, the function checks whether the withdrawal amount is not greater than the withdrawable amount from the lockup stream. The withdrawable amount is the maximum amount of ERC-20 tokens that can be withdrawn by the recipient from the lockup stream. It is calculated using the \_withdrawableAmountOf function.

If the specified withdrawal amount exceeds the withdrawable amount, it means the user is trying to withdraw more than what is available in the stream. In such a case, the function reverts with an error.

```solidity
// Effects: update the withdrawn amount.
_streams[streamId].amounts.withdrawn = _streams[streamId].amounts.withdrawn + amount;
```

After the withdrawal amount is validated, the function updates the withdrawn amount for the lockup stream.

```solidity
// Retrieve the amounts from storage.
Lockup.Amounts memory amounts = _streams[streamId].amounts;
```

This struct contains information about the total deposited amount, total withdrawn amount, and total refunded amount for the stream.

```solidity
// Using ">=" instead of "==" for additional safety reasons. In the event of an unforeseen increase in the
// withdrawn amount, the stream will still be marked as depleted.
if (amounts.withdrawn >= amounts.deposited - amounts.refunded) {
    // Effects: mark the stream as depleted.
    _streams[streamId].isDepleted = true;

    // Effects: make the stream not cancelable anymore, because a depleted stream cannot be canceled.
    _streams[streamId].isCancelable = false;
}
```

It means the entire deposited amount has been withdrawn or that the withdrawn amount has exceeded the total deposited amount for some unforeseen reason.

In such cases, the function marks the stream as depleted. It also makes the stream not cancelable anymore, as a depleted stream cannot be canceled.

```solidity
// Interactions: perform the ERC-20 transfer.
_streams[streamId].asset.safeTransfer({ to: to, value: amount });
```

It transfers the specified amount of ERC-20 tokens from the lockup contract to the recipient's address.

```solidity
// Retrieve the recipient from storage.
address recipient = _ownerOf(streamId);
```

It retrieves the recipient's address associated with the given streamId from storage.

```solidity
// Interactions: if `msg.sender` is not the recipient and the recipient is a contract, try to invoke the
// withdraw hook on it without reverting if the hook is not implemented, and also without bubbling up
// any potential revert.
if (msg.sender != recipient && recipient.code.length > 0) {
    try ISablierV2LockupRecipient(recipient).onStreamWithdrawn({
        streamId: streamId,
        caller: msg.sender,
        to: to,
        amount: amount
    }) { } catch { }
}
```

If both conditions are met, it tries to invoke the onStreamWithdrawn hook.

```solidity
// Log the withdrawal.
emit ISablierV2Lockup.WithdrawFromLockupStream(streamId, to, amount);
```

Finally, the function emits an event called WithdrawFromLockupStream to log the withdrawal from the lockup stream.
