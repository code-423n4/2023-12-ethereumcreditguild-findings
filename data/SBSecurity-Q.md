### Low Risk

| Count | Title |
| --- | --- |
| [L-01] | ProfitManager::donateToTermSurplusBuffer() does not check if the term is from the same market |
| [L-02] | ProfitManager::NotifyPnL will revert in case of loss == (_surplusBuffer + termSurplusBuffer) |
| [L-03] | MinBorrow must be based on the market token |
| [L-04] | gUSDC onboarder can allow implementation of other markets (eg. gWETH) |
| [L-05] | CoreRef::emergencyAction is susceptible to returnbomb attack |
| [L-06] | If borrower become blacklisted for his collateral his loan need to be forgiven in case to receive it |
| [L-07] | IncrementGauge can be called with 0 weight |

| Total Low Risk Issues | 7 |
| --- | --- |

### Non-Critical

| Count | Title |
| --- | --- |
| [NC-01] | Setters should prevent re-setting of the same value |

| Total Non-Critical Issues | 1 |
| --- | --- |

## Low Risks

## [L-01] `ProfitManager::donateToTermSurplusBuffer()` does not check if the term is from the same market

**Issue Description:**

Users can lose their `CREDIT` tokens by calling or `donateToTermSurplusBuffer` for a non-existing term and transferring their tokens to the wrong or eventually non-existing address.

[ProfitManager.sol#L258-L264](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/ProfitManager.sol#L258-L264)

```solidity
/// @notice donate to surplus buffer of a given term
function donateToTermSurplusBuffer(address term, uint256 amount) external {
    CreditToken(credit).transferFrom(msg.sender, address(this), amount);
    uint256 newSurplusBuffer = termSurplusBuffer[term] + amount;
    termSurplusBuffer[term] = newSurplusBuffer;
    emit TermSurplusBufferUpdate(block.timestamp, term, newSurplusBuffer);
}
```

They won’t be able to rescue their tokens unless contract/user with `GUILD_SURPLUS_BUFFER_WITHDRAW` role calls it for a user who has lost his tokens.

[ProfitManager.sol#L277-L287](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/ProfitManager.sol#L277-L287)

```solidity
/// @notice withdraw from surplus buffer
function withdrawFromSurplusBuffer(address to, uint256 amount)
    external
    onlyCoreRole(CoreRoles.GUILD_SURPLUS_BUFFER_WITHDRAW)
{
    uint256 newSurplusBuffer = surplusBuffer - amount; // this would revert due to underflow if withdrawing > surplusBuffer
    surplusBuffer = newSurplusBuffer;
    CreditToken(credit).transfer(to, amount);
    emit SurplusBufferUpdate(block.timestamp, newSurplusBuffer);
}
```

**Recommendation:**

Consider adding check if term is actually a `LendingTerm` contract, modify `donateToTermSurplusBuffer`:

```diff
/// @notice donate to surplus buffer of a given term
function donateToTermSurplusBuffer(address term, uint256 amount) external {
+   require(LendingTerm(term).getReferences().creditToken == credit, "ProfitManager: term from wrong market!");     
    CreditToken(credit).transferFrom(msg.sender, address(this), amount);
    uint256 newSurplusBuffer = termSurplusBuffer[term] + amount;
    termSurplusBuffer[term] = newSurplusBuffer;
    emit TermSurplusBufferUpdate(block.timestamp, term, newSurplusBuffer);
}
```

---

## [L-02] `ProfitManager::NotifyPnL` will revert in case of loss == (_surplusBuffer + termSurplusBuffer)

**Issue Description:**

An edge case can occur if `notifyPnL` is called from `LendingTerm` with an amount equal to the sum of `_surplusBuffer` and `termSurplusBuffer`. The result will be a revert because divide by zero will occur since `loss -= _surplusBuffer` that will prevent the bad debt realization:

```solidity
function notifyPnL(address gauge, int256 amount) external onlyCoreRole(CoreRoles.GAUGE_PNL_NOTIFIER) {
        ... More code
            if (_termSurplusBuffer != 0) {
                termSurplusBuffer[gauge] = 0;
                emit TermSurplusBufferUpdate(block.timestamp, gauge, 0);
                _surplusBuffer += _termSurplusBuffer;
            }

            if (loss < _surplusBuffer) {
                // deplete the surplus buffer
                surplusBuffer = _surplusBuffer - loss;
                emit SurplusBufferUpdate(block.timestamp, _surplusBuffer - loss);
                CreditToken(_credit).burn(loss);
            } else {
                // empty the surplus buffer
                loss -= _surplusBuffer; //@audit will be equal to zero
                surplusBuffer = 0;
                CreditToken(_credit).burn(_surplusBuffer);
                emit SurplusBufferUpdate(block.timestamp, 0);

                // update the CREDIT multiplier
                uint256 creditTotalSupply = CreditToken(_credit).totalSupply();
                uint256 newCreditMultiplier = (creditMultiplier * (creditTotalSupply - loss)) / creditTotalSupply; //@audit 0 / creditTotalSupply
                creditMultiplier = newCreditMultiplier;
                emit CreditMultiplierUpdate(block.timestamp, newCreditMultiplier);
            }
        }
```

**Proof of Concept:**

1. navigate to `src/unit/loan/SurplusGuildMinter.t.sol`
2. place the test
3. execute the test: `forge test --match-contract "SurplusGuildMinterUnitTest" --match-test "testNotifyPnLDivideByZero"`

```solidity
function testNotifyPnLDivideByZero() public {
    // setup
    credit.mint(address(this), 150e18);
    credit.approve(address(sgm), 150e18);
    sgm.stake(term, 150e18);
    
    profitManager.notifyPnL(term, -150e18);
}
```

**Recommendation:**

While this is extremely rare scenario to occur, consider to handle the case when `loss == 0` after the subtraction and take the appropriate actions, as this will indicate that loans are insolvent and term most likely will have to be off-boarded. 

---

## [L-03] `MinBorrow` must be based on the market token

**Issue Description:**

In `LendingTerm.sol`, `minBorrow` is set to 100e18 on deployment which will be a problem when `collateralToken` is an expensive asset such as ETH(~$2000 x 100) or BTC (~$42,000 x 100). This highly decreases the target group of people who are willing to put in that much collateral and open a loan, especially in an immature protocol such as ECG.

[LendingTerm.sol#L365-L368](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L365-L368)

```solidity
require(borrowAmount >= ProfitManager(refs.profitManager).minBorrow(), "LendingTerm: borrow amount too low");
```

[ProfitManager.sol#L151-L153](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/ProfitManager.sol#L151-L153)

```solidity
uint256 internal _minBorrow = 100e18;

function minBorrow() external view returns (uint256) {
    return (_minBorrow * 1e18) / creditMultiplier;
}
```

In addition, changing the `minBorrow` argument from `ProfitManager::setMinBorrow()` proposal should be made and it has to meet a certain quorum, which will take some time.

**Recommendation:**

Consider setting the `minBorrow` from the constructor of the `ProfitManager` that will make the contracts more versatile and will remove the wait period for executing the `setMinBorrow()` function.

[ProfitManager.sol#106-L108](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/ProfitManager.sol#L106-L108)

```diff
constructor(address _core, uint minBorrow) CoreRef(_core) {
        emit MinBorrowUpdate(block.timestamp, 100e18);
+       _minBorrow = minBorrow //should be carefully chosen by the contract deployer considering the price of collateral token
}
```

---

## [L-04] `gUSDC` onboarder can allow implementation of other markets (eg. `gWETH`)

**Issue Description:**

There will be many `LendingTermOnboarding.sol` contracts for every `CREDIT` token used, but there is no check whether the implementation passed to `allowImplementation` function is from the same market. 

There can be case when certain `gWETH` implementation is not allowed in the `LendingTermOnboarding` for `gWETH`, but mistakenly allowed in the `LendingTermOnboarding` for `gUSDC`. The result will be fully functioning `LendingTerm` onboarded from wrong contract and potentially malicious behaviour. 

[LendingTermOnboarding.sol#L92-L102](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOnboarding.sol#L92-L102)

```solidity
function allowImplementation(
        address implementation,
        bool allowed
    ) external onlyCoreRole(CoreRoles.GOVERNOR) {
        implementations[implementation] = allowed;
        emit ImplementationAllowChanged(
            block.timestamp,
            implementation,
            allowed
        );
    }
```

**Recommendation:**

Consider checking if the implementation passed is with the same `CREDIT` token as one in the used in this particular `LendingTermOnboarding`.

---

## [L-05] `CoreRef::emergencyAction` is susceptible to returnbomb attack

**Issue Description:**

Emergency action defined in `CoreRef.sol` which is inherited from all the contracts is a good addition which can be used for various purposes (rescuing tokens, executing certain functions, etc.) but it does not use assembly for consuming the returned data which makes it vulnerable to [returnbomb](https://gist.github.com/pcaversaccio/3b487a24922c839df22f925babd3c809) attack. Since this action can be used for various purposes, including calling untrusted external contracts this is a real possibility for griefing attack: 

[CoreRef.sol#L87-L107](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/core/CoreRef.sol#L87-L107)

```solidity
/// @notice due to inflexibility of current smart contracts,
/// add this ability to be able to execute arbitrary calldata
/// against arbitrary addresses.
/// callable only by governor
function emergencyAction(Call[] calldata calls)
    external
    payable
    onlyCoreRole(CoreRoles.GOVERNOR)
    returns (bytes[] memory returnData)
{
    returnData = new bytes[](calls.length);
    for (uint256 i = 0; i < calls.length; i++) {
        address payable target = payable(calls[i].target);
        uint256 value = calls[i].value;
        bytes calldata callData = calls[i].callData;

        (bool success, bytes memory returned) = target.call{value: value}(callData);
        require(success, "CoreRef: underlying call reverted");
        returnData[i] = returned;
    }
}
```

**Recommendation:**

Consider using [ExcessivelySafeCall](https://github.com/nomad-xyz/ExcessivelySafeCall/blob/main/src/ExcessivelySafeCall.sol) library or assembly to remove the potential vulnerability.

## [L-06] If borrower become blacklisted for his collateral his loan need to be forgiven in case to receive it

**Issue Description:**

If a user is blacklisted for his collateral, trying to repay() will result in a revert. The only way to settle this loan is for the Governor to initiate forgive(), and then the Governor must transfer the tokens to the user. This design choice may not be ideal, as the user might require these tokens immediately.

**Recommendation:**

Consider implement a parameter when full repay is called from the borrower to be able to give another collateral receiver.

## **[L‑07] `IncrementGauge` can be called with 0 weight**

**Issue Description:**

There is no check whether the passed weight from `GUILD` holder is greater than 0.

[ERC20Gauges.sol#L219-226](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/ERC20Gauges.sol#L219-L226)

```solidity
function incrementGauge(address gauge, uint256 weight) public virtual returns (uint256 newUserWeight) {
        require(isGauge(gauge), "ERC20Gauges: invalid gauge");
        _incrementGaugeWeight(msg.sender, gauge, weight);
        return _incrementUserAndGlobalWeights(msg.sender, weight);
    }
```

We’ve been discussing potential scenario where this can be used to create an infinite loop in `_decrementWeightUntilFree` as the increment of `i` is done only in case `userGaugeWeight` is different than 0:

[ERC20Gauges.sol#L500-L539](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20Gauges.sol#L500-L539)

```solidity
function _decrementWeightUntilFree(address user, uint256 weight) internal {
    uint256 userFreeWeight = balanceOf(user) - getUserWeight[user];

		// early return if already free
    if (userFreeWeight >= weight) return;

    // cache totals for batch updates
    uint256 userFreed;
    uint256 totalFreed;

    // Loop through all user gauges, live and deprecated
    address[] memory gaugeList = _userGauges[user].values();

    // Free gauges until through entire list or under weight
    uint256 size = gaugeList.length;
    for (
        uint256 i = 0;
        i < size && (userFreeWeight + userFreed) < weight;

    ) {
        address gauge = gaugeList[i];
        uint256 userGaugeWeight = getUserGaugeWeight[user][gauge];
        if (userGaugeWeight != 0) {
            userFreed += userGaugeWeight;
            _decrementGaugeWeight(user, gauge, userGaugeWeight);

            // If the gauge is live (not deprecated), include its weight in the total to remove
            if (!_deprecatedGauges.contains(gauge)) {
                totalTypeWeight[gaugeType[gauge]] -= userGaugeWeight;
                totalFreed += userGaugeWeight;
            }

            unchecked {
                ++i; //@audit only in case userGaugeWeight != 0
            }
        }
    }

    totalWeight -= totalFreed;
}
```

That way if user manages to put a gauge with 0 weight at 0 position in his `userGauges` and after that all the others, he can effectively **avoid slashing and grief the applier** who calls `applyGaugeLoss` for him.

Other than that `transfer`, `transferFrom` and `burn` will always lead to infinite loop if user doesn’t have enough balance and weight has to be freed, but there is weight of 0 in the first iteration of the for loop above.

**Recommendation:**

Verify that the weight passed is greater than 0, that will mitigate the possible griefing possibility.

## Non-Critical

## **[N‑01]** Setters should check if set the same value

**Issue Description:**

The setter functions listed don't check if the new value is different from the old value.

[CoreRef::_setCore](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/core/CoreRef.sol#L49-L54)

```solidity
function _setCore(address newCore) internal {
    address oldCore = address(_core);
    // @audit-info no check if old == new 
    _core = Core(newCore);

    emit CoreUpdate(oldCore, newCore);
}
```

[GuildGovernor::_setQuorum](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/GuildGovernor.sol#L68-L71)

```solidity
function _setQuorum(uint256 newQuorum) internal {
    emit QuorumUpdated(_quorum, newQuorum);
    // @audit-info no check if old == new
    _quorum = newQuorum;
}
```

[GuildVetoGovernor::_setQuorum](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/GuildVetoGovernor.sol#L71-L74)

```solidity
function _setQuorum(uint256 newQuorum) internal virtual {
    emit QuorumUpdated(_quorum, newQuorum);
    // @audit-info old == new check
    _quorum = newQuorum;
}
```

[GuildVetoGovernor::_updateTimelock](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/GuildVetoGovernor.sol#L99-L102)

```
function _updateTimelock(address newTimelock) private {
    emit TimelockChange(timelock, newTimelock);
    // @audit-info old == new check
    timelock = newTimelock;
}
```

**Recommendation:**

Add checks to verify that the new value is not equal to the old value