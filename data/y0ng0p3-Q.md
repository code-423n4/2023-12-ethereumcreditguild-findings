# QA Report

## Table of contents

- ### [L-01. Misleading NatSpec Comments](#L-01)

- ### [L-02. Free up deprecated gauges first](#L-02)

- ### [L-03. Use latest stable version of Solidity](#L-03)

- ## [NC-01. Missing NatSpec @notice and @param tags in events, modifiers #and functions](#NC-01)

- ### [NC-02. Missing NatSpec function @return tag](#NC-02)

## Low Risk Issues

### <a id='L-01'></a>[L-01] Misleading NatSpec Comments

Comments should align with the code

#### 1- [ProfitManager.#178](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/ProfitManager.sol#L178)

```text
File: src/governance/ProfitManager.sol

178:    /// @notice set the minimum borrow amount
```

The above comment should be corrected as follows:

```text
File: src/governance/ProfitManager.sol

178:    set the minimum size of CREDIT loans
```

#### 2- [LendingTerm.sol#-L40](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L34-L40)

```text
File: src/loan/LendingTerm.sol

34: /// @notice emitted when a loan is closed (repay, onBid after a call, forgive).
```

The above comment should be move to line 39 as follows:

```text
File: src/loan/LendingTerm.sol

34: enum LoanCloseType {

39: /// @notice emitted when a loan is closed (repay, onBid after a call, forgive).
40: event LoanClose(
```

### <a id='L-02'></a>[L-02] Free up deprecated gauges first

Gauges are freed up according to their index in gaugeList
It is better to free up deprecated gauges first as they are, effectively deprecated. Then free up live gauges if they are still needed.

[ERC20Gauges#L514-L536](<https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L514-L536>)

### <a id='L-03'></a>[L-03] Use latest stable version of Solidity

Many files use solidity version 0.8.13. Consider using at least version 0.8.18 solidity pragma version because newer versions have bug fixes and security updates.

Files:

- [Core.sol](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/Core.sol#L2)
- [CoreRef.sol](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRef.sol#L2)
- [CoreRoles.sol](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRoles.sol#L2)
- [GuildGovernor.sol](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildGovernor.sol#L2)
- [GuildTimelockController.sol](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildTimelockController.sol#L2)
- [LendingTermOffboarding.sol](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOffboarding.sol#L2)
- [LendingTermOnboarding.sol](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOnboarding.sol#L2)
- [ProfitManager.sol](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L2)
- [GuildVetoGovernor.sol](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L2)
- [ERC20Gauges.sol](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L2)
- [ERC20MultiVotes.sol](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L2)
- [ERC20RebaseDistributor.sol](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L2)
- [CreditToken.sol](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/CreditToken.sol#L2)
- [GuildToken.sol](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L2)
- [AuctionHouse.sol](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/AuctionHouse.sol#L2)
- [LendingTerm.sol](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L2)
- [SimplePSM.sol](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SimplePSM.sol#L2)
- [SurplusGuildMinter.sol](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L2)
- [RateLimitedMinter.sol](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/limits/RateLimitedMinter.sol#L2)
- [RateLimitedV2.sol](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/utils/RateLimitedV2.sol#L2)

## Non-critical Issues

### <a id='NC-01'></a>[NC-01] Missing NatSpec @notice and @param tags in events, modifiers and functions

Many functions, structures, and events are missing @notice tag and @param tag.

### <a id='NC-02'></a>[NC-02] Missing NatSpec function @return tag

Many functions in the protocol contrats are missing @return tag.
