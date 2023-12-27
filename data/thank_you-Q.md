### L-01

LendingTerm._borrow() prevents third-party users from utilizing multiple loans within the same timestamp. This can result in third-party users completely unable to use the borrow feature as they may want to create separate distinguishable loans in the same block.

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L347-L349

```solidity
loanId = keccak256(
    abi.encode(borrower, address(this), block.timestamp)
);
```

### L-02

LendingTermOffboarding.POLL_DURATION_BLOCKS constant is used to determine a length of a poll. The value is currently set to:

```solidity
uint256 public constant POLL_DURATION_BLOCKS = 46523; // ~7 days @ 13s/block
```

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOffboarding.sol#L36

Unfortunately, because ECG plans to deploy to multiple chains, this poll duration must be dynamic as different chains may have different block lengths. For example, [Polygon](https://polygonscan.com/chart/blocktime) on average has 2 seconds per block.

The ECG should consider setting this value to be dynamic per chain.

### L-03

The LendingTerm.debtCeiling() function returns the debt ceiling for the given term. Unfortunately, the code is designed such that there is an off-chance that if `_hardCap < creditMinterBuffer < (_issuance + maxBorrow)`, then the creditMintBuffer will be returned. In this case, _hardCap should be returned.

This can cause GuildToken._decrementGaugeWeight to decrement when it should if the above edge case occurs and the issuance is greater than the _hardCap.

The LendingTerm.debtCeiling() runs a series of calculations to determine the Term's debt ceiling. The debt ceiling should always lean towards returning the smallest acceptable value to ensure protocol safety and prevent runaway borrowing. You can see this safety check occur throughout the function where in most scenarios the debtCeiling() returns the following logical statement:

`_hardCap < creditMinterBuffer ? _hardCap : creditMinterBuffer;`


Unfortunately, at the end of the function the final set of code logic runs:

```
// return min(creditMinterBuffer, hardCap, debtCeiling)
if (creditMinterBuffer < _debtCeiling) {
    return creditMinterBuffer;
}
if (_hardCap < _debtCeiling) {
    return _hardCap;
}
return _debtCeiling;
```

This logic dictates that as long as creditMinterBuffer is lower than _debtCeiling, then creditMinterBuffer will be returned in the function. However, it does not consider if _hardCap is lower than the creditMinterBuffer. This can lead to a scenario where the debtCeiling() returns a value larger than expected. 


We can see this in action in the following sample code below:

```solidity
function debtCeiling() public returns (uint) {
    uint creditMinterBuffer = 100;
    uint _debtCeiling = 101;
    uint _hardCap = 99;
    
    uint returnValue;
    
    
    if (creditMinterBuffer < _debtCeiling) {
        return creditMinterBuffer;
    }
    if (_hardCap < _debtCeiling) {
        return _hardCap;
    }
}

uint ceiling = debtCeiling(); // will return 100
```



The protocol should re-work the function to do the following:

```solidity
uint ceiling = _hardCap;

if (creditMinterBuffer < ceiling) {
    ceiling = creditMinterBuffer;
}
if (_debtCeiling < ceiling) {
    ceiling = _debtCeiling;
}

return ceiling;
```
 
 



 