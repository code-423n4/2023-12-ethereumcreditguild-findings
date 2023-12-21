## Summary
[I-1] Typo.
[I-2] `YEAR` is too long.
[L-1] Unsafe call to decimals().

### [I-1] Typo.

```solidity
@audit [0, 100]%
interest rate [0, 100[% APR
```
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/LendingTermOnboarding.sol#L128

```solidity
@audit [0, 100]%
periodic payment sizes [0, 100[%
```

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/LendingTermOnboarding.sol#L139

```solidity
@audit [0, midPoint]
[0, midPoint[
```

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/AuctionHouse.sol#L139

```solidity
@audit [0, PHASE_2_DURATION]
[0, PHASE_2_DURATION[
```

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/AuctionHouse.sol#L150

```solidity
@audit [0, 1e18]
[0, 1e18[
```

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L510C69-L510C78

```solidity
@audit au -> an
@notice callback from the auctionHouse when au auction concludes
```

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L724

### [I-2] `YEAR` is too long.
The constant `YEAR` is set to `31_557_600`. However in one year there are `31_536_000` seconds. There is a difference of `21_600` seconds.

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/SurplusGuildMinter.sol#L29

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L68

### [L-1] Unsafe call to `decimals()`.
The `decimals` function is optional in the initial ERC20 and might fail for old tokens that do not implement it. Consider using it here as well, also this solution might be helpful: https://github.com/boringcrypto/BoringSolidity/blob/c73ed73afa9273fbce93095ef177513191782254/contracts/libraries/BoringERC20.sol#L49-L55

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/SimplePSM.sol#L75