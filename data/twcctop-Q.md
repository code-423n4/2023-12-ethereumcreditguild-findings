### L1 debtCeiling don't check _hardCap when  no more borrows allowed 


https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/e3d3e581e0e51a9ecf3a5a0c4e4bd4af32552fc0/src/loan/LendingTerm.sol#L305-L311



```solidity 
   if (_issuance >= debtCeilingBefore) {
            return debtCeilingBefore; // no more borrows allowed
        }

```

When no more borrow allowed,  it will return the minimum value of `_issuance` and `debtCeilingBefore`, but don't check the `_hardCap`. 

