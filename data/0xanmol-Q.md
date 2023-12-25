## [L-1]  `MinStake` is constant in `SurplusGuildMinter` and it may not be feasible for all markets

### Code Line

https://github.com/volt-protocol/ethereum-credit-guild/blob/4d33abf95fee69391af0652e3cbe5e0cffa25f9f/src/loan/SurplusGuildMinter.sol#L26C46-L26C46

### Impact

In the `SurplusGuildMinter` contract, `minStake` is the minimum amount of credit that users need to stake in order to increase the gauge weight. Currently, this variable is set as a constant, with a value of 1e18, meaning it cannot be changed and always represents 1e18 GUSDC. This works fine when dealing with the USDC:GUSDC market, as GUSDC is assumed to be pegged to USDC. However, a problem arises when there are multiple markets.

### Proof of Concept

Let's consider the scenario where the team decides to add a new market for WETH:GWETH, where GWETH is pegged to WETH. If the same `SurplusGuildMinter` contract is deployed with changes to the constructor variable (which will happen because the contract is immutable), the 1e18 `minStake` amount will not be feasible for GWETH. 1e18 represents 1 GWETH, which is a significant amount to be set as the minimum.

### Recommendation

To address this issue, it is recommended to make `minStake` an immutable variable and set the feasible minimum stake amount when deploying the contract.


