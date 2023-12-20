| Finding | Summary                                                                                                                                                     |
|---------|-------------------------------------------------------------------------------------------------------------------------------------------------------------|
| L-01    |  No slippage used in [_shares2balance](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20RebaseDistributor.sol#L260-L273) |
| L-02 | Market Utilization Targeted at 100% |
| I-01 | ImplementationAllowChanged event is emitted even when there is no change in the allow status of an implementation |


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

Here is the revised text with improved grammar and syntax:

### [L-02] Market Utilization Targeted at 100%

The protocol aims to achieve a market utilization of 100% through the setting of [maxDebtPerCollateralToken](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L125). However, in certain markets, utilization can and will reach 100%. This is possible due to the fact that lenders have high liquidity and can exit at will, whereas borrowers do not enjoy the same flexibility.

Markets with 100% utilization pose a challenge for lenders, as they become illiquid. If lenders decide to leave, they must wait for someone to mint credit tokens, leading to significant market queues. When capital enters the market through [**PSM**](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/SimplePSM.sol), it is swiftly swapped out as lenders front-run each other to exit.

My suggestion is to implement a targeting mechanism similar to AAVE, where the borrowing fee increases as more of the market is utilized. This approach will encourage faster borrowing and attract deposits from external lenders as APRs rise.

### [I-01] ImplementationAllowChanged event is emitted even when there is no change in the allow status of an implementation
The allowImplementation() method does not perform a check of the allow status of an implementation. It always emits the ImplementationAllowChanged in all cases even when there is no change in the allow status. This could potentially impact external systems which rely on the events as an integration mechanism.
