# [L-1] Must approve 0 first
#### Impact:
Some tokens (like USDT) do not work when changing the allowance from an existing non-zero allowance value.They must first be approved by zero and then the actual allowance must be approved.
*Instances (1)*:
```solidity
File: loan/SurplusGuildMinter.sol

129:         CreditToken(credit).approve(address(profitManager), amount);

```