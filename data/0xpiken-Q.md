## [L&#x2011;01] [`claimRewards()`](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/SurplusGuildMinter.sol#L239) could be reverted due to Out Of GAS
There's a possibility of [`SurplusGuildMinter#getRewards()`](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/SurplusGuildMinter.sol#L216-L290) failing due to running out of gas, caused by [`ProfitManager#claimRewards()`](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/ProfitManager.sol#L439-L449) because there's no maximum gauge limitation on `SurplusGuildMinter`. (`canExceedMaxGauges` for `SurplusGuildMinter` is set to `true` in [deployment scripts](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/test/proposals/gips/GIP_0.sol#L480-L483))

To mitigate this, consider using `ProfitManager#claimGaugeRewards()` instead:
```diff
-       ProfitManager(profitManager).claimRewards(address(this)); // this will update profit indexes
+       ProfitManager(profitManager).claimGaugeRewards(address(this), term);
```

## [L&#x2011;02] Anyone can `distribute()` only 1 wei to extend rebase time to `DISTRIBUTION_PERIOD`
consider introducing a `PROFIT_DISTRIBUTION` role and adding access control on `distribute()`.

Update the visibility of `distribute()` in `ERC20RebaseDistributor`:
```diff
-   function distribute(uint256 amount) external {
+   function distribute(uint256 amount) public {
```
Add access control by overriding `distribute()` in `CreditToken`:
```solidity
    function distribute(uint256 amount) public override onlyCoreRole(CoreRoles.PROFIT_DISTRIBUTION) {
        super.distribute(amount);
    }
```
`PROFIT_DISTRIBUTION` role should be granted to `ProfitManager` when deployed.