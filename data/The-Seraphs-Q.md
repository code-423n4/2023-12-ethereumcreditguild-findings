# [Issue-01]: Incorrect documentation

## Code References

- https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOnboarding.sol#L246
- https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOnboarding.sol#L255

### Description

The documentation states the following:

```solidity
// 2nd call: core.grantRole(term, RATE_LIMITED_CREDIT_MINTER)
...
...
// 3rd call: core.grantRole(term, GAUGE_PNL_NOTIFIER)
```

The function `Core::grantRole()` comes from `AccessControl::grantRole()` which has a signature and definition as follows:

```solidity
function grantRole(bytes32 role, address account) public virtual override onlyRole(getRoleAdmin(role)) {
    _grantRole(role, account);
}
```

We can see that in the documentation the parameters are swapped - It mentioned account first, and then the role, whereas in the actual `grantRole()` function it is role and then account.

### Recommendation

Swap the arguments in the documentation:


```diff
-    // 2nd call: core.grantRole(term, RATE_LIMITED_CREDIT_MINTER)
+    // 2nd call: core.grantRole(RATE_LIMITED_CREDIT_MINTER, term)
...
...
-    // 3rd call: core.grantRole(term, GAUGE_PNL_NOTIFIER)
+    // 3rd call: core.grantRole(GAUGE_PNL_NOTIFIER, term)
```

---

# [Issue-02]: Missing equality in condition leads to a more complex code path being executed unnecessarily

## Code References

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/ProfitManager.sol#L315

### Description

In `ProfitManager::notifyPnL()`, we have the following condition:

```solidity
if (loss < _surplusBuffer) {
    ...
    ...
} else {
    ...
    ...
}
```

If the `loss` is more than `_surplusBuffer`, then the `else` block gets triggered, where all the `_surplusBuffer` is burnt, and the `creditMultiplier` is recalculated.

If the `loss` is less than `_surplusBuffer`, then the `if` block gets triggered, and the loss amount of credit tokens are burnt.

But if `loss` is equal to `_surplusBuffer`, then the `else` block gets executed where all the `_surplusBuffer` is burnt, and `creditMultiplier` is recalculated, which does not change as demonstrated below:

```solidity
// @audit loss == _surplusBuffer
//        so after the below line, loss becomes 0:
//        => loss -= _surplusBuffer;
//        => loss = 0
loss -= _surplusBuffer; // loss is 0 now
surplusBuffer = 0;

...
...

uint256 creditTotalSupply = CreditToken(_credit).totalSupply();

// @audit In the below line loss is 0,
//        so newCreditMultiplier remains to be creditMultiplier
//        => uint256 newCreditMultiplier = (creditMultiplier * (creditTotalSupply - loss)) / creditTotalSupply;
//        => uint256 newCreditMultiplier = (creditMultiplier * (creditTotalSupply - 0)) / creditTotalSupply;
//        => uint256 newCreditMultiplier = (creditMultiplier * (creditTotalSupply)) / creditTotalSupply;
//        => uint256 newCreditMultiplier = (creditMultiplier * creditTotalSupply) / creditTotalSupply;
//        => uint256 newCreditMultiplier = creditMultiplier;
                    
uint256 newCreditMultiplier = (creditMultiplier *
    (creditTotalSupply - loss)) / creditTotalSupply;
...
...
```

### Recommendation

Update the if condition to include equality, so that if `loss == _surplusBuffer`, then its treated the same as `loss < _surplusBuffer`


```diff
-   if (loss < _surplusBuffer) {
+   if (loss <= _surplusBuffer) {
        ...
        ...
    } else {
        ...
        ...
    }
```

---

# [Issue-03]: Use `ERC20Gauges::numLiveGauges()` function instead of duplicating logic

## Code References

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20Gauges.sol#L130

### Description

In `ERC20Gauges::liveGauges()`, the following code exists:

```solidity
function liveGauges() external view returns (address[] memory _liveGauges) {
    _liveGauges = new address[](
@>      _gauges.length() - _deprecatedGauges.length()
    );
    ...
    ...
}
```

There is a function `ERC20Gauges::numLiveGauges()`, which looks like:

```solidity
function numLiveGauges() external view returns (uint256) {
    return _gauges.length() - _deprecatedGauges.length();
}
```

The `ERC20Gauges::liveGauges()` can reuse the `ERC20Gauges::numLiveGauges()` function instead of duplicating the code.


### Recommendation

Update the function `ERC20Gauges::liveGauges()` to use `ERC20Gauges::numLiveGauges()`, as follows:

```diff
    function liveGauges() external view returns (address[] memory _liveGauges) {
        _liveGauges = new address[](
-           _gauges.length() - _deprecatedGauges.length()
+           numLiveGauges()
        );
        ...
        ...
    }
```

---

# [Issue-04]: Disallow repayment of 0 debt earlier in _partialRepay() function

## Code References

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L519C1-L522C11

### Description

Require statement placed after multiple calls and calculations that could be at the beginning to save time and potentially gas costs.

The `function _partialRepay()` functions implements a `require()` statement close to the end of the function call which subsequently checks wether the debt to repay amount is above 0.
```solidity
    require(
        principalRepaid != 0 && interestRepaid != 0, 
        "LendingTerm: repay too small"
    );
```
However, this check is after multiple calculations and checks that could be avoided, with a potential gas cost saving, if a check at the beginning of the function was implemented for the debt to repay amount.

## Proof of Concept
1. **Run the test without any changes to the protocol.**
    - **Test Contract:** `LendingTerm.t.sol`
    - **Test Created:** `testFailPartialRepay()`

    ```solidity
        function testFailPartialRepay() public {
            // prepare & borrow
            uint256 borrowAmount = 20_000e18;
            uint256 collateralAmount = 15e18;
            collateral.mint(address(this), collateralAmount);
            collateral.approve(address(term), collateralAmount);
            bytes32 loanId = term.borrow(borrowAmount, collateralAmount);
            assertEq(term.getLoan(loanId).collateralAmount, collateralAmount);

            // partialRepay
            vm.warp(block.timestamp + term.YEAR());
            vm.roll(block.number + 1);
            assertEq(term.getLoanDebt(loanId), 22_000e18);
            credit.mint(address(this), 11_000e18);
            credit.approve(address(term), 11_000e18);
            term.partialRepay(loanId, 0);

        }
    ```
    #### Results: Gas 471 217

    ```json
    [⠆] Compiling...
    [⠰] Compiling 1 files with 0.8.13
    [⠔] Solc 0.8.13 finished in 7.03s
    Compiler run successful!

    Running 1 test for test/unit/loan/LendingTerm.t.sol:LendingTermUnitTest
    [PASS] testFailPartialRepay() (gas: 471217)
    Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 5.76ms

    Ran 1 test suites: 1 tests passed, 0 failed, 0 skipped (1 total tests)
    ```
    #### Revert reason:

    ```json
       ├─ [5519] term::partialRepay(0xef76992452dcc745eb4787bfb358222c5c1152ec5ff5ecf953055b33567fde0a, 0)
        │   ├─ [5330] LendingTerm::partialRepay(0xef76992452dcc745eb4787bfb358222c5c1152ec5ff5ecf953055b33567fde0a, 0) [delegatecall]
        │   │   ├─ [394] profitManager::creditMultiplier() [staticcall]
        │   │   │   └─ ← 1000000000000000000 [1e18]
        │   │   ├─ [394] profitManager::creditMultiplier() [staticcall]
        │   │   │   └─ ← 1000000000000000000 [1e18]
        │   │   └─ ← revert: LendingTerm: repay too small
    ```

1. Amend the `LendingTerm::_partialRepay()` function, adding the below `require()` statement at the top: 
    ```solidity
    require(debtToRepay != 0, "LendingTerm: cannot repay 0 [pxng0lin]");
    ```

1. **Then run the test again.**

    #### Results: Gas 466 375
    ```json
    [⠢] Compiling...
    No files changed, compilation skipped

    Running 1 test for test/unit/loan/LendingTerm.t.sol:LendingTermUnitTest
    [PASS] testFailPartialRepay() (gas: 466375)
    Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 5.80ms

    Ran 1 test suites: 1 tests passed, 0 failed, 0 skipped (1 total tests)
    ```
    #### Revert reason:
    ```json
    ├─ [677] term::partialRepay(0xef76992452dcc745eb4787bfb358222c5c1152ec5ff5ecf953055b33567fde0a, 0)
    │   ├─ [488] LendingTerm::partialRepay(0xef76992452dcc745eb4787bfb358222c5c1152ec5ff5ecf953055b33567fde0a, 0) [delegatecall]
    │   │   └─ ← revert: LendingTerm: cannot repay 0 [pxng0lin]
    ```

## Recommendation

As described in the PoC, adding a require statement at the beginning of the function would be more efficient and give a better UX, with the potential gas cost saving.

```diff
    function _partialRepay(
        address repayer,
        bytes32 loanId,
        uint256 debtToRepay
    ) internal {
+       require(debtToRepay != 0, "LendingTerm: cannot repay 0 [pxng0lin]"); // @audit Disallow repayment of 0
        Loan storage loan = loans[loanId];
    ...
}
```
---

# [Issue-05]: Reuse `ERC20Gauges::decrementGauge()` in `ERC20Gauges::decrementGauges` to avoid code duplication

## Code References

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20Gauges.sol#L356-L360

### Description

The `ERC20Gauges::decrementGauge()` function decrements a single gauge.

```solidity
function decrementGauge(
    address gauge,
    uint256 weight
) public virtual returns (uint256 newUserWeight) {
    // All operations will revert on underflow, protecting against bad inputs
    _decrementGaugeWeight(msg.sender, gauge, weight);
    if (!_deprecatedGauges.contains(gauge)) {
        totalTypeWeight[gaugeType[gauge]] -= weight;
        totalWeight -= weight;
    }
    return getUserWeight[msg.sender];
}
```

The `ERC20Gauges::decrementGauges()` function decrements a multiple gauges.

```solidity
function decrementGauges(
    address[] calldata gaugeList,
    uint256[] calldata weights
) public virtual returns (uint256 newUserWeight) {
    uint256 size = gaugeList.length;
    require(weights.length == size, "ERC20Gauges: size mismatch");

    // store total in summary for batch update on user/global state
    uint256 weightsSum;

    // Update gauge specific state
    // All operations will revert on underflow, protecting against bad inputs
    for (uint256 i = 0; i < size; ) {
        address gauge = gaugeList[i];
        uint256 weight = weights[i];

@>      _decrementGaugeWeight(msg.sender, gauge, weight);
@>      if (!_deprecatedGauges.contains(gauge)) {
@>          totalTypeWeight[gaugeType[gauge]] -= weight;
@>          weightsSum += weight;
@>      }
        unchecked {
            ++i;
        }
    }
    totalWeight -= weightsSum;
    return getUserWeight[msg.sender];
}
```

The `ERC20Gauges::decrementGauge()` function can be reused in `ERC20Gauges::decrementGauges` as they share the same logic.

### Recommendation

Update the code as follows to re-use the code.

```diff
function decrementGauges(
    address[] calldata gaugeList,
    uint256[] calldata weights
) public virtual returns (uint256 newUserWeight) {
    uint256 size = gaugeList.length;
    require(weights.length == size, "ERC20Gauges: size mismatch");

    // store total in summary for batch update on user/global state
    uint256 weightsSum;

    // Update gauge specific state
    // All operations will revert on underflow, protecting against bad inputs
    for (uint256 i = 0; i < size; ) {
        address gauge = gaugeList[i];
        uint256 weight = weights[i];

+       decrementGauge(gauge, weight);
-       _decrementGaugeWeight(msg.sender, gauge, weight);
-       if (!_deprecatedGauges.contains(gauge)) {
-           totalTypeWeight[gaugeType[gauge]] -= weight;
-           weightsSum += weight;
-       }
        unchecked {
            ++i;
        }
    }
-   totalWeight -= weightsSum;
    return getUserWeight[msg.sender];
}
```