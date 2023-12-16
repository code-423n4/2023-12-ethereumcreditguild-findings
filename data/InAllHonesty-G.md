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
        uint256 toleratedGaugeWeight = (gaugeWeight * gaugeWeightTolerance) / 1e18;
        uint256 debtCeilingBefore = (totalBorrowedCredit *
            toleratedGaugeWeight) / totalWeight; 
        if (_issuance >= debtCeilingBefore) {
            return debtCeilingBefore; // no more borrows allowed
        }
--      uint256 remainingDebtCeiling = debtCeilingBefore - _issuance; // always >0 @audit-info GAS this can be moved to improve efficiency, maybe we hit the below condition and return and we don't need to initialize this
        if (toleratedGaugeWeight >= totalWeight) {
            // if the gauge weight is above 100% when we include tolerance,
            // the gauge relative debt ceilings are not constraining.
            return
                _hardCap < creditMinterBuffer ? _hardCap : creditMinterBuffer;
        }
++      uint256 remainingDebtCeiling = debtCeilingBefore - _issuance; // always >0 
        uint256 otherGaugesWeight = totalWeight - toleratedGaugeWeight; // always >0
```

## [G-02] `LendingTerm.getLoanDebt()` can be optimized

```
        uint256 borrowAmount = loan.borrowAmount;
--      uint256 interest = (borrowAmount *  //@audit-info GAS unpack interest variable into loanDebt, it's used only once
--          params.interestRate *
--          (block.timestamp - borrowTime)) /
--          YEAR /
--            1e18;
--      uint256 loanDebt = borrowAmount + interest;
++      uint256 loanDebt = borrowAmount + (borrowAmount * params.interestRate * (block.timestamp - borrowTime)) / YEAR / 1e18; 
        uint256 _openingFee = params.openingFee;
```

`Overall gas change: (gas: -32 (-0.004%))`