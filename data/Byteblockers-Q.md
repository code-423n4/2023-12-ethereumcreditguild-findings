| Finding | Summary                                                                                                                                                     |
|---------|-------------------------------------------------------------------------------------------------------------------------------------------------------------|
| L-01    |  No slippage used in [_shares2balance](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20RebaseDistributor.sol#L260-L273) |
| I-01    | ImplementationAllowChanged event is emitted even when there is no change in the allow status of an implementation                                           |

### [L-01] No slippage used in [_shares2balance](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20RebaseDistributor.sol#L260-L273)
When [_shares2balance](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20RebaseDistributor.sol#L260-L273) is used in the place of `minBalance` users current token balance is used. This means that there is 0 slippage protection, as the function can calculate the user's balance as the same even after  burning the shares.

Example is [_exitRebase](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20RebaseDistributor.sol#L312-L321) where the user `rawBalance` (current ERC20 balance) is used as the minimul of `rawBalance` + increse from shares to ERC20.

```solidity
uint256 rawBalance = ERC20.balanceOf(account);
uint256 rebasedBalance = _shares2balance(shares, currentRebasingSharePrice, 0, rawBalance);
uint256 mintAmount = rebasedBalance - rawBalance;
if (mintAmount != 0) {
    ERC20._mint(account, mintAmount);
    decreaseUnmintedRebaseRewards(mintAmount);
    emit RebaseReward(account, block.timestamp, mintAmount);
}
uint256 mintAmount = rebasedBalance - rawBalance;
```

My suggestion is to implement some sort of slippage protection.

### [I-01] ImplementationAllowChanged event is emitted even when there is no change in the allow status of an implementation
The allowImplementation() method does not perform a check of the allow status of an implementation. It always emits the ImplementationAllowChanged in all cases even when there is no change in the allow status. This could potentially impact external systems which rely on the events as an integration mechanism.
