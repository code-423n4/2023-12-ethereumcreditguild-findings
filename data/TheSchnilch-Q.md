# debtCeiling() does not determine the correct minimum which results in debtCeiling being higher than it should be
### Description
At the end of the function debtCeiling in LendingTerm.sol the minimum of creditMinterBuffer, _debtCeiling and _hardCap will be determined like this:
```solidity
        // return min(creditMinterBuffer, hardCap, debtCeiling)
        if (creditMinterBuffer < _debtCeiling) {
            return creditMinterBuffer;
        }
        if (_hardCap < _debtCeiling) {
            return _hardCap;
        }
        return _debtCeiling;
```
The problem here is that if creditMinterBuffer < _debtCeiling, then it is not checked whether _hardCap is smaller than creditMinterBuffer. So it could be, although very unlikely, that the wrong debtCeiling will be returned. This would result in the wrong debtCeiling being used in the _decrementGaugeWeight function in GuildToken.sol, making it possible to decrement more tokens than you should be able to:
```solidity
if (issuance != 0) {
            uint256 debtCeilingAfterDecrement = LendingTerm(gauge).debtCeiling(-int256(weight));
            require(
                issuance <= debtCeilingAfterDecrement,
                "GuildToken: debt ceiling used"
            );
        }
```
### Recommendation
It should also be checked whether _hardCap is smaller than creditMinterBuffer. The minimum should then be determined in this way:
```diff
// return min(creditMinterBuffer, hardCap, debtCeiling)
if (creditMinterBuffer < _debtCeiling) {
+		if(_hardCap < creditMinterBuffer) return _hardCap;
		return creditMinterBuffer;
}
if (_hardCap < _debtCeiling) {
    return _hardCap;
}
return _debtCeiling;
```