### LendingTerm::getLoanDebt()
The below loan object can be loaded in memory
 ```
 Loan storage loan = loans[loanId];
```

### LendingTerm::debtCeiling()
The gaugeWeightTolerance can be read after the if condition incase the call does not return.
This is save some gas.
```
        // @audit this call can be moved down below the if condition to save some gas.    
        uint256 gaugeWeightTolerance = ProfitManager(refs.profitManager)
            .gaugeWeightTolerance();
        if (totalBorrowedCredit == 0 && gaugeWeight != 0) {
            // first-ever CREDIT mint on a non-zero gauge weight term
            // does not check the relative debt ceilings
            // returns min(hardCap, creditMinterBuffer)
            return
                _hardCap < creditMinterBuffer ? _hardCap : creditMinterBuffer;
        }

```