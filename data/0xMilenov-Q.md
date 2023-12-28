## Wrong check in buffer depletion
[RateLimitedV2.sol](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/utils/RateLimitedV2.sol#L89-L116)

### Impact

This issue can make the contract stop working temporarily. If someone uses up all the actions allowed, no one else can use the contract until it gets more allowances. This can be annoying and make the contract seem unreliable.

### Description

In the `RateLimitedV2` contract, there's a limit on how often you can do things like creating new tokens. A specific function, `_depleteBuffer(uint256 amount)`, controls how much action is allowed. The problem happens when someone uses exactly the maximum amount allowed by the contract. This uses up all the action allowance and leaves nothing for others until it resets.

```solidity
/// @notice the method that enforces the rate limit.
/// Decreases buffer by "amount".
/// If buffer is <= amount, revert
/// @param amount to decrease buffer by
function _depleteBuffer(uint256 amount) internal {
    uint256 newBuffer = buffer();

    require(newBuffer != 0, "RateLimited: no rate limit buffer");
    // @audit Natspec comment says revert if buffer is <= amount, but the require statement
    // is checking if the newBuffer is bigger or equal than the amount
    require(amount <= newBuffer, "RateLimited: rate limit hit");
    // the buffer becomes fully depleted (buffer goes to zero).
    // This is the edge case where you're using the maximum allowed quota
    // in a single transaction without exceeding it.

    uint32 blockTimestamp = block.timestamp.safeCastTo32();
    uint224 newBufferStored = (newBuffer - amount).safeCastTo224();

    /// gas optimization to only use a single SSTORE
    lastBufferUsedTime = blockTimestamp;
    bufferStored = newBufferStored;

    emit BufferUsed(amount, bufferStored);
}
```

This can happen because the contract checks if the requested action is more than what's allowed. If it's equal to or less, the contract goes ahead. So, if someone repeatedly requests the maximum amount allowed, they can keep using up all the allowance on their own. This stops other people from using the contract until the allowance is replenished.While this behavior is in line with standard rate-limiting practices, it opens up the possibility for griefing. A user or group could repeatedly perform actions that exactly match the available buffer, thus continually depleting it and preventing other users from accessing the functionality reliant on this buffer.

### Recomendation

Remove '=' in the require statement:

```solidity
require(amount < newBuffer, "RateLimited: rate limit hit");
```