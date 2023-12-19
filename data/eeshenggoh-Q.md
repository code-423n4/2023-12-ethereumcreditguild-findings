The way the `ERC20RebaseDistributor::transfer`, `ERC20RebaseDistributor::transferFrom` & `ERC20RebaseDistributor::_burn` function is coded can be optimized by ensuring the execution of code be minimized. Instead of calling the _mint function, can just combined to one transfer function with deducted amount if required.
```diff 
File: src\tokens\ERC20RebaseDistributor.sol::transfer
            uint256 mintAmount = rebasedBalance - fromBalanceBefore;
            if (mintAmount != 0) {
--                ERC20._mint(msg.sender, mintAmount);
                fromBalanceBefore += mintAmount;
                decreaseUnmintedRebaseRewards(mintAmount);
                emit RebaseReward(msg.sender, block.timestamp, mintAmount);
            }
        }

        // do ERC20.transfer()
--      bool success = ERC20.transfer(to, amount);
++      bool success = ERC20.transfer(to, amount - mintAmount);
```
```diff
File: src\tokens\ERC20RebaseDistributor.sol::transferFrom
            uint256 mintAmount = rebasedBalance - fromBalanceBefore;
            if (mintAmount != 0) {
--              ERC20._mint(from, mintAmount);
                fromBalanceBefore += mintAmount;
                decreaseUnmintedRebaseRewards(mintAmount);
                emit RebaseReward(from, block.timestamp, mintAmount);
            }
        }

        // do ERC20.transferFrom()
--      bool success = ERC20.transferFrom(from, to, amount);
++      bool success = ERC20.transferFrom(from, to, amount-mintAmount);

```
```diff
File: src\tokens\ERC20RebaseDistributor.sol::_burn
            uint256 mintAmount = rebasedBalance - balanceBefore;
            if (mintAmount != 0) {
                ERC20._mint(account, mintAmount);
                balanceBefore += mintAmount;
                decreaseUnmintedRebaseRewards(mintAmount);
                emit RebaseReward(account, block.timestamp, mintAmount);
            }
        }

        // do ERC20._burn()
--        ERC20._burn(account, amount);
++        ERC20._burn(account, amount-mintAmount);

```

=======================
In `GuildToken::notifyGaugeLoss`, it is not using the RBAC method `CoreRoles Gauage_PNL_NOTIFER`. Developers should have the same standard in writing codes.
```
//CoreRoles.sol
    /// @notice can notify of profits & losses in a given gauge
    bytes32 internal constant GAUGE_PNL_NOTIFIER =
        keccak256("GAUGE_PNL_NOTIFIER_ROLE");
```

```diff
++    function notifyGaugeLoss(address gauge) external onlyCoreRole(CoreRoles.GAUGE_PNL_NOTIFIER){
--    function notifyGaugeLoss(address gauge) external  {
--      require(msg.sender == profitManager, "UNAUTHORIZED");

        // save gauge loss
        lastGaugeLoss[gauge] = block.timestamp;
        emit GaugeLoss(gauge, block. Timestamp);
    }
```
=========================

In the `RateLimitedV2::_setRateLimitPerSecond`, should just use uint128, dont need upcast waste gas The uint128 type is used to save gas when the maximum value of the variable does not exceed 2^128 - 1. However, when the value is converted to uint256, it will consume more gas because uint256 is larger than uint128 1.
```
    function _setRateLimitPerSecond(uint128 newRateLimitPerSecond) internal {
        uint256 oldRateLimitPerSecond = rateLimitPerSecond; //@audit (
        rateLimitPerSecond = newRateLimitPerSecond;

        emit RateLimitPerSecondUpdate(
            oldRateLimitPerSecond,
            newRateLimitPerSecond
        );
    }
```