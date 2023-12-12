# Gas Findings

## [GAS-1] `ERC20Gauges`: `weight` Doesn't Need to Be Checked if a Gauge is a `newAdd`

`weight` doesn't need to be checked if the gauge is a `newAdd` since it won't have a previous `weight`.

![](https://i.imgur.com/QE0a6lw.png)

### Recommendation

Move lines 415-420 into the `else` block just above it.

### Link To Affected Code

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20Gauges.sol#L415-L420
