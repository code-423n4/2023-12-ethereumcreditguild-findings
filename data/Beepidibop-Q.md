# Low-risk and Non-critical Findings

## [L-1] `LendingTermOffboarding`: Using `block.number` to Determine Duration on Multiple Chains

The team expressed interest in deploying the project on multiple L2s, but different L2s have different behaviour for `block.number`. For example, Arbitrum return an approximate L1 block number while OP returns the L2 block number.
![](https://i.imgur.com/H4HZF6V.png)
![](https://i.imgur.com/RQAq2LJ.png)
![](https://i.imgur.com/trnLWvM.png)

### Recommendation

Change `block.number` comparison for `block.timestamp`.

### Link To Affected Code

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOffboarding.sol#L36
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOffboarding.sol#L95
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOffboarding.sol#L121