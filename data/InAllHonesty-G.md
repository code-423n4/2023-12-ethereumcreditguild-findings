## [G-01] `LendingTerm.debtCeiling()` can be optimized

By placing the cached SLOAD later in the execution flow, we create a possibility of not caching it in case the `if (totalBorrowedCredit == 0 && gaugeWeight != 0)` yields true and returns thus halting execution, thereby enhancing the gas efficiency of the  `debtCeiling()` function.

```diff
--      uint256 _issuance = issuance; // cached SLOAD @audit-info GAS this can be moved to improve efficiency, maybe we hit the below condition and return and we don't need to cache this
        uint256 totalBorrowedCredit = ProfitManager(refs.profitManager)
            .totalBorrowedCredit();
        uint256 gaugeWeightTolerance = ProfitManager(refs.profitManager)
            .gaugeWeightTolerance();
        if (totalBorrowedCredit == 0 && gaugeWeight != 0) {
            // first-ever CREDIT mint on a non-zero gauge weight term
            // does not check the relative debt ceilings
            // returns min(hardCap, creditMinterBuffer)
            return
                _hardCap < creditMinterBuffer ? _hardCap : creditMinterBuffer;
        }
++      uint256 _issuance = issuance; // cached SLOAD
```