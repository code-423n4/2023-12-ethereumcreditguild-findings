### [G&#x2011;01]  saves gas by removing duplicate code while setting `rateLimitPerSecond `

In `RateLimitedV2.sol`, constructor checks argument `_rateLimitPerSecond` should be less than `_maxRateLimitPerSecond`. This checks in middle of constructor thereby has to go through bit more computation in order to pass this condition. This similar condition is also used in `setRateLimitPerSecond()`. These extra bytes can be removed as the removal of duplicating code will result in gas saving.

**Recommendation**:

```diff
    constructor(
        uint256 _maxRateLimitPerSecond,
        uint128 _rateLimitPerSecond,
        uint128 _bufferCap
    ) {
        lastBufferUsedTime = block.timestamp.safeCastTo32();

        _setBufferCap(_bufferCap);
        bufferStored = _bufferCap;

-        require(
-            _rateLimitPerSecond <= _maxRateLimitPerSecond,
-            "RateLimited: rateLimitPerSecond too high"
-        );
        _setRateLimitPerSecond(_rateLimitPerSecond);  @audit // This require condition is now a part of internal function

        MAX_RATE_LIMIT_PER_SECOND = _maxRateLimitPerSecond;
    }


    function setRateLimitPerSecond(
        uint128 newRateLimitPerSecond
    ) external virtual onlyCoreRole(CoreRoles.GOVERNOR) {
-        require(
-            newRateLimitPerSecond <= MAX_RATE_LIMIT_PER_SECOND,
-            "RateLimited: rateLimitPerSecond too high"
-        );
        _updateBufferStored(bufferCap);

        _setRateLimitPerSecond(newRateLimitPerSecond);
    }


   // some code

    function _setRateLimitPerSecond(uint128 newRateLimitPerSecond) internal {
+        require(
+            newRateLimitPerSecond <= MAX_RATE_LIMIT_PER_SECOND,
+            "RateLimited: rateLimitPerSecond too high"
+        );
        uint256 oldRateLimitPerSecond = rateLimitPerSecond;
        rateLimitPerSecond = newRateLimitPerSecond;

        emit RateLimitPerSecondUpdate(
            oldRateLimitPerSecond,
            newRateLimitPerSecond
        );
    }
```

### [G&#x2011;02]  Gas can be saved by avoiding external call to contract
In `SimplePSM.sol`, the value of [creditMultiplier](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/ProfitManager.sol#L87) is fetched from `ProfitManager.sol`. `creditMultiplier` is used in functions like `getMintAmountOut()` and `getRedeemAmountOut()`. Per `ProfitManager.sol`, there is no functionlity to change the `creditMultiplier` value and it is `1e18` and the contract is not upgradeable too. Therefore, `creditMultiplier` value can be directly used in `SimplePSM.sol` contract without fetching it from external contract.

**Recommendation**:

```diff
    // value of creditMultiplier is being directly used here from ProfitManager.sol to avoid external calls which also helps in gas saving and removes few bytes of code from contract
+   uint256 public creditMultiplier = 1e18;

    /// @notice calculate the amount of CREDIT out for a given `amountIn` of underlying
    function getMintAmountOut(uint256 amountIn) public view returns (uint256) {
-        uint256 creditMultiplier = ProfitManager(profitManager)
-            .creditMultiplier();
        return (amountIn * decimalCorrection * 1e18) / creditMultiplier;
    }


    /// @notice calculate the amount of underlying out for a given `amountIn` of CREDIT
    function getRedeemAmountOut(
        uint256 amountIn
    ) public view returns (uint256) {
-        uint256 creditMultiplier = ProfitManager(profitManager)
-            .creditMultiplier();
        return (amountIn * creditMultiplier) / 1e18 / decimalCorrection;
    }
```

### [G&#x2011;03]  Refactor `forceEnterRebase()` function to avoid code duplication
In `CreditToken.sol`, [forceEnterRebase()](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/CreditToken.sol#L72) has used below condition to check the account has subscribed to rebasing or not.

```solidity
        require(
            rebasingState[account].isRebasing == 0,
            "CreditToken: already rebasing"
        );
```

Since, `CreditTokens.sol` inherits [ERC20RebaseDistributor.sol](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/CreditToken.sol#L22C5-L22C27) contract therefore there is no need to have above condition as [ERC20RebaseDistributor.isRebasing()](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/ERC20RebaseDistributor.sol#L389) can be directly checked instead of writing new code which unnecessarily increase conytract deployment cost.

**Recommendation**:

```diff

    function forceEnterRebase(
        address account
    ) external onlyCoreRole(CoreRoles.CREDIT_REBASE_PARAMETERS) {
-        require(
-            rebasingState[account].isRebasing == 0,
-            "CreditToken: already rebasing"
-        );
+        require(!isRebasing(account), "CreditToken: already rebasing");
        _enterRebase(account);
    }
```

### [G&#x2011;04]  Save gas by optimizing `emergencyAction()`
`emergencyAction()` can be very much optimized.

Gas usage before- 65225
Gas usage after code optimization- 64772
Difference- 453

**Recommendation**:

```diff

    function emergencyAction(
        Call[] calldata calls
    )
        external
        payable
        onlyCoreRole(CoreRoles.GOVERNOR)
        returns (bytes[] memory returnData)
    {
        returnData = new bytes[](calls.length);
-        for (uint256 i = 0; i < calls.length; i++) {
+        for (uint256 i; i < calls.length;) {
            address payable target = payable(calls[i].target);
            uint256 value = calls[i].value;
            bytes calldata callData = calls[i].callData;

            (bool success, bytes memory returned) = target.call{value: value}(
                callData
            );
            require(success, "CoreRef: underlying call reverted");
            returnData[i] = returned;
+            unchecked {
+                ++i;
+            }
        }
    }
```
