**Note:** I've filtered out the specific instances found by the winning bot

## Summary

### Low Risk Issues

| |Issue|Instances|
|-|:-|:-:|
| [[L&#x2011;01](#l01-code-does-not-follow-the-best-practice-of-check-effects-interaction)] | Code does not follow the best practice of check-effects-interaction | 121 | 
| [[L&#x2011;02](#l02-file-allows-a-version-of-solidity-that-is-susceptible-to-selector-related-optimizer-bug)] | File allows a version of solidity that is susceptible to `.selector`-related optimizer bug | 2 | 
| [[L&#x2011;03](#l03-initialization-can-be-front-run)] | Initialization can be front-run | 1 | 

Total: 124 instances over 3 issues

### Non-critical Issues

| |Issue|Instances|
|-|:-|:-:|
| [[N&#x2011;01](#n01-if-statement-can-be-converted-to-a-ternary)] | `if`-statement can be converted to a ternary | 3 | 
| [[N&#x2011;02](#n02-add-inline-comments-for-unnamed-variables)] | Add inline comments for unnamed variables | 9 | 
| [[N&#x2011;03](#n03-cast-to-bytes-or-bytes32-for-clearer-semantic-meaning)] | Cast to `bytes` or `bytes32` for clearer semantic meaning | 1 | 
| [[N&#x2011;04](#n04-common-functions-should-be-refactored-to-a-common-base-contract)] | Common functions should be refactored to a common base contract | 2 | 
| [[N&#x2011;05](#n05-consider-emitting-an-event-at-the-end-of-the-constructor)] | Consider emitting an event at the end of the constructor | 15 | 
| [[N&#x2011;06](#n06-consider-making-contracts-upgradeable)] | Consider making contracts `Upgradeable` | 14 | 
| [[N&#x2011;07](#n07-consider-moving-duplicated-strings-to-constants)] | Consider moving duplicated strings to constants | 45 | 
| [[N&#x2011;08](#n08-consider-returning-a-struct-rather-than-having-multiple-return-values)] | Consider returning a `struct` rather than having multiple `return` values | 1 | 
| [[N&#x2011;09](#n09-consider-splitting-complex-checks-into-multiple-steps)] | Consider splitting complex checks into multiple steps | 1 | 
| [[N&#x2011;10](#n10-consider-using-accesscontroldefaultadminrules-rather-than-accesscontrol)] | Consider using `AccessControlDefaultAdminRules` rather than `AccessControl` | 2 | 
| [[N&#x2011;11](#n11-consider-using-descriptive-constants-when-passing-zero-as-a-function-argument)] | Consider using descriptive `constant`s when passing zero as a function argument | 24 | 
| [[N&#x2011;12](#n12-consider-using-named-function-arguments)] | Consider using named function arguments | 31 | 
| [[N&#x2011;13](#n13-consider-using-named-returns)] | Consider using named returns | 80 | 
| [[N&#x2011;14](#n14-consider-using-the-using-for-syntax)] | Consider using the `using`-`for` syntax | 37 | 
| [[N&#x2011;15](#n15-expressions-for-constant-values-should-use-immutable-rather-than-constant)] | Expressions for constant values should use `immutable` rather than `constant` | 2 | 
| [[N&#x2011;16](#n16-function-state-mutability-can-be-restricted-to-view)] | Function state mutability can be restricted to `view` | 3 | 
| [[N&#x2011;17](#n17-interfaces-should-use-floating-version-pragmas)] | Interfaces should use floating version pragmas | 1 | 
| [[N&#x2011;18](#n18-long-functions-should-be-refactored-into-multiple-smaller-functions)] | Long functions should be refactored into multiple, smaller, functions | 11 | 
| [[N&#x2011;19](#n19-multiple-addressid-mappings-can-be-combined-into-a-single-mapping-of-an-addressid-to-a-struct-for-readability)] | Multiple `address`/ID mappings can be combined into a single `mapping` of an `address`/ID to a `struct`, for readability | 12 | 
| [[N&#x2011;20](#n20-natspec-non-public-state-variable-declarations-should-use-dev-tags)] | NatSpec: Non-public state variable declarations should use `@dev` tags | 34 | 
| [[N&#x2011;21](#n21-natspec-state-variable-declarations-should-have-descriptions)] | NatSpec: State variable declarations should have descriptions | 1 | 
| [[N&#x2011;22](#n22-outdated-package-versions)] | Outdated package versions | 25 | 
| [[N&#x2011;23](#n23-pausable-contract-never-uses-whennotposed-modifier)] | Pausable contract never uses `whenNotPosed` modifier | 1 | 
| [[N&#x2011;24](#n24-style-guide-non-externalpublic-variable-names-should-begin-with-an-underscore)] | Style guide: Non-`external`/`public` variable names should begin with an underscore | 17 | 
| [[N&#x2011;25](#n25-use-of-override-is-unnecessary)] | Use of `override` is unnecessary | 49 | 

Total: 421 instances over 25 issues



## Low Risk Issues


### [L&#x2011;01] Code does not follow the best practice of check-effects-interaction
Code should follow the best-practice of [check-effects-interaction](https://blockchain-academy.hs-mittweida.de/courses/solidity-coding-beginners-to-intermediate/lessons/solidity-11-coding-patterns/topic/checks-effects-interactions/), where state variables are updated before any external calls are made. Doing so prevents a large class of reentrancy bugs.

*There are 121 instances of this issue:*

<details>
<summary>see instances</summary>


```solidity
File: src/governance/GuildVetoGovernor.sol

/// @audit getTimestamp() called prior to this assignment in createVeto(), via: createVeto()
340:         _timelockIds[proposalId] = timelockId;

```
*GitHub*: [340](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L340-L340)

```solidity
File: src/governance/LendingTermOffboarding.sol

/// @audit isGauge() called prior to this assignment in proposeOffboard(), via: proposeOffboard()
104:         polls[block.number][term] = 1; // voting power

/// @audit isGauge() called prior to this assignment in proposeOffboard(), via: proposeOffboard()
105:         lastPollBlock[term] = block.number;

/// @audit getPastVotes() called prior to this assignment in supportOffboard(), via: supportOffboard()
136:         userPollVotes[msg.sender][snapshotBlock][term] = userWeight;

/// @audit getPastVotes() called prior to this assignment in supportOffboard(), via: supportOffboard()
137:         polls[snapshotBlock][term] = _weight + userWeight;

/// @audit getPastVotes() called prior to this assignment in supportOffboard(), via: supportOffboard()
139:             canOffboard[term] = true;

/// @audit redemptionsPaused() called prior to this assignment in cleanup(), via: cleanup()
/// @audit setRedemptionsPaused() called prior to this assignment in cleanup(), via: cleanup()
/// @audit revokeRole() called prior to this assignment in cleanup(), via: cleanup()
/// @audit isDeprecatedGauge() called prior to this assignment in cleanup(), via: cleanup()
/// @audit issuance() called prior to this assignment in cleanup(), via: cleanup()
197:         canOffboard[term] = false;

```
*GitHub*: [104](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOffboarding.sol#L104-L104), [105](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOffboarding.sol#L105-L105), [136](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOffboarding.sol#L136-L136), [137](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOffboarding.sol#L137-L137), [139](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOffboarding.sol#L139-L139), [197](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOffboarding.sol#L197-L197), [197](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOffboarding.sol#L197-L197), [197](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOffboarding.sol#L197-L197), [197](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOffboarding.sol#L197-L197), [197](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOffboarding.sol#L197-L197)

```solidity
File: src/governance/LendingTermOnboarding.sol

/// @audit initialize() called prior to this assignment in createTerm(), via: createTerm()
165:         created[term] = block.timestamp;

```
*GitHub*: [165](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOnboarding.sol#L165-L165)

```solidity
File: src/governance/ProfitManager.sol

/// @audit transferFrom() called prior to this assignment in donateToTermSurplusBuffer(), via: donateToTermSurplusBuffer()
262:         termSurplusBuffer[term] = newSurplusBuffer;

/// @audit notifyGaugeLoss() called prior to this assignment in notifyPnL(), via: notifyPnL()
310:                 termSurplusBuffer[gauge] = 0;

/// @audit notifyGaugeLoss() called prior to this assignment in notifyPnL(), via: notifyPnL()
317:                 surplusBuffer = _surplusBuffer - loss;

/// @audit notifyGaugeLoss() called prior to this assignment in notifyPnL(), via: notifyPnL()
326:                 surplusBuffer = 0;

/// @audit totalSupply() called prior to this assignment in notifyPnL(), via: notifyPnL()
/// @audit burn() called prior to this assignment in notifyPnL(), via: notifyPnL()
/// @audit notifyGaugeLoss() called prior to this assignment in notifyPnL(), via: notifyPnL()
334:                 creditMultiplier = newCreditMultiplier;

/// @audit distribute() called prior to this assignment in notifyPnL(), via: notifyPnL()
/// @audit transfer() called prior to this assignment in notifyPnL(), via: notifyPnL()
/// @audit getGaugeWeight() called prior to this assignment in notifyPnL(), via: notifyPnL()
396                      gaugeProfitIndex[gauge] =
397                          _gaugeProfitIndex +
398                          (amountForGuild * 1e18) /
399:                         _gaugeWeight;

/// @audit getUserGaugeWeight() called prior to this assignment in claimGaugeRewards(), via: claimGaugeRewards()
430:             userGaugeProfitIndex[user][gauge] = _gaugeProfitIndex;

```
*GitHub*: [262](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L262-L262), [310](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L310-L310), [317](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L317-L317), [326](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L326-L326), [334](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L334-L334), [334](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L334-L334), [334](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L334-L334), [396](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L396-L399), [396](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L396-L399), [396](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L396-L399), [430](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L430-L430)

```solidity
File: src/loan/AuctionHouse.sol

/// @audit getLoan() called prior to this assignment in startAuction(), via: startAuction()
/// @audit hasRole() called prior to this assignment in startAuction(), via: startAuction()
96           auctions[loanId] = Auction({
97               startTime: block.timestamp,
98               endTime: 0,
99               lendingTerm: msg.sender,
100              collateralAmount: loan.collateralAmount,
101              callDebt: callDebt
102:         });

/// @audit getLoan() called prior to this assignment in startAuction(), via: startAuction()
/// @audit hasRole() called prior to this assignment in startAuction(), via: startAuction()
103:         nAuctionsInProgress++;

```
*GitHub*: [96](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/AuctionHouse.sol#L96-L102), [96](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/AuctionHouse.sol#L96-L102), [103](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/AuctionHouse.sol#L103-L103), [103](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/AuctionHouse.sol#L103-L103)

```solidity
File: src/loan/LendingTerm.sol

/// @audit calculateGaugeAllocation() called prior to this assignment in _borrow(), via: _borrow()
/// @audit gaugeWeightTolerance() called prior to this assignment in _borrow(), via: _borrow()
/// @audit totalBorrowedCredit() called prior to this assignment in _borrow(), via: _borrow()
/// @audit minBorrow() called prior to this assignment in _borrow(), via: _borrow()
/// @audit creditMultiplier() called prior to this assignment in _borrow(), via: _borrow()
401          loans[loanId] = Loan({
402              borrower: borrower,
403              borrowTime: block.timestamp,
404              borrowAmount: borrowAmount,
405              borrowCreditMultiplier: creditMultiplier,
406              collateralAmount: collateralAmount,
407              caller: address(0),
408              callTime: 0,
409              callDebt: 0,
410              closeTime: 0
411:         });

/// @audit calculateGaugeAllocation() called prior to this assignment in _borrow(), via: _borrow()
/// @audit gaugeWeightTolerance() called prior to this assignment in _borrow(), via: _borrow()
/// @audit totalBorrowedCredit() called prior to this assignment in _borrow(), via: _borrow()
/// @audit minBorrow() called prior to this assignment in _borrow(), via: _borrow()
/// @audit creditMultiplier() called prior to this assignment in _borrow(), via: _borrow()
412:         issuance = _postBorrowIssuance;

/// @audit calculateGaugeAllocation() called prior to this assignment in _borrow(), via: _borrow()
/// @audit gaugeWeightTolerance() called prior to this assignment in _borrow(), via: _borrow()
/// @audit totalBorrowedCredit() called prior to this assignment in _borrow(), via: _borrow()
/// @audit minBorrow() called prior to this assignment in _borrow(), via: _borrow()
/// @audit creditMultiplier() called prior to this assignment in _borrow(), via: _borrow()
414:             lastPartialRepay[loanId] = block.timestamp;

/// @audit minBorrow() called prior to this assignment in _partialRepay(), via: _partialRepay()
/// @audit creditMultiplier() called prior to this assignment in _partialRepay(), via: _partialRepay()
/// @audit creditMultiplier() called prior to this assignment in _partialRepay(), via: _partialRepay(), getLoanDebt()
534:         loans[loanId].borrowAmount -= issuanceDecrease;

/// @audit minBorrow() called prior to this assignment in _partialRepay(), via: _partialRepay()
/// @audit creditMultiplier() called prior to this assignment in _partialRepay(), via: _partialRepay()
/// @audit creditMultiplier() called prior to this assignment in _partialRepay(), via: _partialRepay(), getLoanDebt()
535:         lastPartialRepay[loanId] = block.timestamp;

/// @audit minBorrow() called prior to this assignment in _partialRepay(), via: _partialRepay()
/// @audit creditMultiplier() called prior to this assignment in _partialRepay(), via: _partialRepay()
/// @audit creditMultiplier() called prior to this assignment in _partialRepay(), via: _partialRepay(), getLoanDebt()
536:         issuance -= issuanceDecrease;

/// @audit replenishBuffer() called prior to this assignment in _repay(), via: _repay()
/// @audit burn() called prior to this assignment in _repay(), via: _repay()
/// @audit transfer() called prior to this assignment in _repay(), via: _repay()
/// @audit notifyPnL() called prior to this assignment in _repay(), via: _repay()
/// @audit transferFrom() called prior to this assignment in _repay(), via: _repay()
/// @audit creditMultiplier() called prior to this assignment in _repay(), via: _repay()
/// @audit creditMultiplier() called prior to this assignment in _repay(), via: _repay(), getLoanDebt()
614:         loan.closeTime = block.timestamp;

/// @audit replenishBuffer() called prior to this assignment in _repay(), via: _repay()
/// @audit burn() called prior to this assignment in _repay(), via: _repay()
/// @audit transfer() called prior to this assignment in _repay(), via: _repay()
/// @audit notifyPnL() called prior to this assignment in _repay(), via: _repay()
/// @audit transferFrom() called prior to this assignment in _repay(), via: _repay()
/// @audit creditMultiplier() called prior to this assignment in _repay(), via: _repay()
/// @audit creditMultiplier() called prior to this assignment in _repay(), via: _repay(), getLoanDebt()
615:         issuance -= borrowAmount;

/// @audit creditMultiplier() called prior to this assignment in _call(), via: _call(), getLoanDebt()
/// @audit isDeprecatedGauge() called prior to this assignment in _call(), via: _call()
666:         loans[loanId].callTime = block.timestamp;

/// @audit creditMultiplier() called prior to this assignment in _call(), via: _call(), getLoanDebt()
/// @audit isDeprecatedGauge() called prior to this assignment in _call(), via: _call()
667:         loans[loanId].callDebt = loanDebt;

/// @audit creditMultiplier() called prior to this assignment in _call(), via: _call(), getLoanDebt()
/// @audit isDeprecatedGauge() called prior to this assignment in _call(), via: _call()
668:         loans[loanId].caller = caller;

/// @audit notifyPnL() called prior to this assignment in forgive(), via: forgive()
/// @audit creditMultiplier() called prior to this assignment in forgive(), via: forgive()
718:         params.hardCap = 0;

/// @audit creditMultiplier() called prior to this assignment in onBid(), via: onBid()
771:         loans[loanId].closeTime = block.timestamp;

/// @audit notifyPnL() called prior to this assignment in onBid(), via: onBid()
/// @audit transfer() called prior to this assignment in onBid(), via: onBid()
/// @audit replenishBuffer() called prior to this assignment in onBid(), via: onBid()
/// @audit burn() called prior to this assignment in onBid(), via: onBid()
/// @audit transferFrom() called prior to this assignment in onBid(), via: onBid()
/// @audit creditMultiplier() called prior to this assignment in onBid(), via: onBid()
801:         issuance -= borrowAmount;

/// @audit nAuctionsInProgress() called prior to this assignment in setAuctionHouse(), via: setAuctionHouse()
841:         refs.auctionHouse = _newValue;

```
*GitHub*: [401](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L401-L411), [401](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L401-L411), [401](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L401-L411), [401](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L401-L411), [401](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L401-L411), [412](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L412-L412), [412](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L412-L412), [412](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L412-L412), [412](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L412-L412), [412](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L412-L412), [414](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L414-L414), [414](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L414-L414), [414](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L414-L414), [414](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L414-L414), [414](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L414-L414), [534](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L534-L534), [534](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L534-L534), [534](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L534-L534), [535](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L535-L535), [535](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L535-L535), [535](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L535-L535), [536](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L536-L536), [536](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L536-L536), [536](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L536-L536), [614](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L614-L614), [614](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L614-L614), [614](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L614-L614), [614](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L614-L614), [614](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L614-L614), [614](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L614-L614), [614](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L614-L614), [615](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L615-L615), [615](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L615-L615), [615](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L615-L615), [615](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L615-L615), [615](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L615-L615), [615](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L615-L615), [615](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L615-L615), [666](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L666-L666), [666](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L666-L666), [667](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L667-L667), [667](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L667-L667), [668](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L668-L668), [668](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L668-L668), [718](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L718-L718), [718](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L718-L718), [771](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L771-L771), [801](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L801-L801), [801](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L801-L801), [801](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L801-L801), [801](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L801-L801), [801](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L801-L801), [801](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L801-L801), [841](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L841-L841)

```solidity
File: src/loan/SimplePSM.sol

/// @audit decimals() called prior to this assignment in 0(), via: ()
76:          decimalCorrection = 10 ** (18 - decimals);

/// @audit creditMultiplier() called prior to this assignment in mint(), via: mint(), getMintAmountOut()
108:         pegTokenBalance += amountIn;

/// @audit creditMultiplier() called prior to this assignment in mintAndEnterRebase(), via: mintAndEnterRebase(), getMintAmountOut()
/// @audit isRebasing() called prior to this assignment in mintAndEnterRebase(), via: mintAndEnterRebase()
125:         pegTokenBalance += amountIn;

```
*GitHub*: [76](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SimplePSM.sol#L76-L76), [108](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SimplePSM.sol#L108-L108), [125](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SimplePSM.sol#L125-L125), [125](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SimplePSM.sol#L125-L125)

```solidity
File: src/loan/SurplusGuildMinter.sol

/// @audit userGaugeProfitIndex() called prior to this assignment in stake(), via: stake()
/// @audit incrementGauge() called prior to this assignment in stake(), via: stake()
/// @audit mint() called prior to this assignment in stake(), via: stake()
/// @audit donateToTermSurplusBuffer() called prior to this assignment in stake(), via: stake()
/// @audit approve() called prior to this assignment in stake(), via: stake()
/// @audit transferFrom() called prior to this assignment in stake(), via: stake()
/// @audit userGaugeProfitIndex() called prior to this assignment in stake(), via: stake(), getRewards()
/// @audit claimRewards() called prior to this assignment in stake(), via: stake(), getRewards()
/// @audit lastGaugeLoss() called prior to this assignment in stake(), via: stake(), getRewards()
/// @audit mint() called prior to this assignment in stake(), via: stake(), getRewards()
/// @audit transfer() called prior to this assignment in stake(), via: stake(), getRewards()
151:         _stakes[msg.sender][term] = userStake;

/// @audit userGaugeProfitIndex() called prior to this assignment in unstake(), via: unstake(), getRewards()
/// @audit claimRewards() called prior to this assignment in unstake(), via: unstake(), getRewards()
/// @audit lastGaugeLoss() called prior to this assignment in unstake(), via: unstake(), getRewards()
/// @audit mint() called prior to this assignment in unstake(), via: unstake(), getRewards()
/// @audit transfer() called prior to this assignment in unstake(), via: unstake(), getRewards()
196:         _stakes[msg.sender][term] = userStake;

/// @audit mint() called prior to this assignment in getRewards(), via: getRewards()
/// @audit transfer() called prior to this assignment in getRewards(), via: getRewards()
/// @audit userGaugeProfitIndex() called prior to this assignment in getRewards(), via: getRewards()
/// @audit claimRewards() called prior to this assignment in getRewards(), via: getRewards()
/// @audit lastGaugeLoss() called prior to this assignment in getRewards(), via: getRewards()
288:             _stakes[user][term] = userStake;

/// @audit incrementGauge() called prior to this assignment in updateMintRatio(), via: updateMintRatio()
/// @audit mint() called prior to this assignment in updateMintRatio(), via: updateMintRatio()
/// @audit userGaugeProfitIndex() called prior to this assignment in updateMintRatio(), via: updateMintRatio(), getRewards()
/// @audit claimRewards() called prior to this assignment in updateMintRatio(), via: updateMintRatio(), getRewards()
/// @audit lastGaugeLoss() called prior to this assignment in updateMintRatio(), via: updateMintRatio(), getRewards()
/// @audit mint() called prior to this assignment in updateMintRatio(), via: updateMintRatio(), getRewards()
/// @audit transfer() called prior to this assignment in updateMintRatio(), via: updateMintRatio(), getRewards()
307:             _stakes[user][term].guild = SafeCastLib.safeCastTo128(guildAfter);

/// @audit burn() called prior to this assignment in updateMintRatio(), via: updateMintRatio()
/// @audit replenishBuffer() called prior to this assignment in updateMintRatio(), via: updateMintRatio()
/// @audit decrementGauge() called prior to this assignment in updateMintRatio(), via: updateMintRatio()
/// @audit userGaugeProfitIndex() called prior to this assignment in updateMintRatio(), via: updateMintRatio(), getRewards()
/// @audit claimRewards() called prior to this assignment in updateMintRatio(), via: updateMintRatio(), getRewards()
/// @audit lastGaugeLoss() called prior to this assignment in updateMintRatio(), via: updateMintRatio(), getRewards()
/// @audit mint() called prior to this assignment in updateMintRatio(), via: updateMintRatio(), getRewards()
/// @audit transfer() called prior to this assignment in updateMintRatio(), via: updateMintRatio(), getRewards()
313:             _stakes[user][term].guild = SafeCastLib.safeCastTo128(guildAfter);

```
*GitHub*: [151](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L151-L151), [151](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L151-L151), [151](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L151-L151), [151](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L151-L151), [151](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L151-L151), [151](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L151-L151), [151](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L151-L151), [151](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L151-L151), [151](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L151-L151), [151](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L151-L151), [151](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L151-L151), [196](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L196-L196), [196](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L196-L196), [196](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L196-L196), [196](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L196-L196), [196](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L196-L196), [288](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L288-L288), [288](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L288-L288), [288](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L288-L288), [288](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L288-L288), [288](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L288-L288), [307](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L307-L307), [307](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L307-L307), [307](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L307-L307), [307](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L307-L307), [307](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L307-L307), [307](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L307-L307), [307](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L307-L307), [313](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L313-L313), [313](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L313-L313), [313](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L313-L313), [313](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L313-L313), [313](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L313-L313), [313](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L313-L313), [313](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L313-L313), [313](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L313-L313)

</details>





### [L&#x2011;02] File allows a version of solidity that is susceptible to `.selector`-related optimizer bug
In solidity versions prior to 0.8.21, there is a legacy code generation [bug](https://soliditylang.org/blog/2023/07/19/missing-side-effects-on-selector-access-bug/) where if `foo().selector` is called, `foo()` doesn't actually get evaluated. It is listed as low-severity, because projects usually use the contract name rather than a function call to look up the selector. I've flagged all files using `.selector` where the version is vulnerable.

*There are 2 instances of this issue:*

```solidity
File: src/governance/GuildVetoGovernor.sol

2:   pragma solidity 0.8.13;

```
*GitHub*: [2](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L2-L2)

```solidity
File: src/governance/LendingTermOnboarding.sol

2:   pragma solidity 0.8.13;

```
*GitHub*: [2](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOnboarding.sol#L2-L2)



### [L&#x2011;03] Initialization can be front-run
The `initialize()` functions below are not called by another contract atomically after the contract is deployed, so it's possible for a malicious user to call `initialize()` which, if it's noticed in time, would require the project to re-deploy the contract in order to properly initialize. Consider creating a factory contract, which will `new` and `initialize()` each contract atomically.

*There is one instance of this issue:*

```solidity
File: src/governance/ProfitManager.sol

156      function initializeReferences(
157          address _credit,
158          address _guild,
159          address _psm
160:     ) external onlyCoreRole(CoreRoles.GOVERNOR) {

```
*GitHub*: [156](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L156-L160)


## Non-critical Issues


### [N&#x2011;01] `if`-statement can be converted to a ternary
The code can be made more compact while also increasing readability by converting the following `if`-statements to ternaries (e.g. `foo += (x > y) ? a : b`)

*There are 3 instances of this issue:*

```solidity
File: src/governance/GuildVetoGovernor.sol

288              if (_quorumReached(proposalId)) {
289                  return ProposalState.Succeeded;
290              }
291              // need more votes to veto
292              else {
293                  return ProposalState.Active;
294:             }

```
*GitHub*: [288](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L288-L294)

```solidity
File: src/tokens/ERC20RebaseDistributor.sol

406          if (_rebasingSupply > _totalSupply) {
407              return 0;
408          } else {
409              return _totalSupply - _rebasingSupply;
410:         }

430          if (_rebasingState.isRebasing == 0) {
431              return ERC20.balanceOf(account);
432          } else {
433              return
434                  _shares2balance(
435                      _rebasingState.nShares,
436                      rebasingSharePrice(),
437                      0,
438                      ERC20.balanceOf(account)
439                  );
440:         }

```
*GitHub*: [406](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L406-L410), [430](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L430-L440)



### [N&#x2011;02] Add inline comments for unnamed variables
`function foo(address x, address)` -> `function foo(address x, address /* y */)`

*There are 9 instances of this issue:*

```solidity
File: src/tokens/ERC20Gauges.sol

35                - function setMaxGauges(uint256) external
36:               - function setCanExceedMaxGauges(address, bool) external

32            -> permissioned public functions to add in parent:
33:               - function addGauge(address) external returns (uint112)

33                - function addGauge(address) external returns (uint112)
34:               - function removeGauge(address) external

34                - function removeGauge(address) external
35:               - function setMaxGauges(uint256) external

35                - function setMaxGauges(uint256) external
36:               - function setCanExceedMaxGauges(address, bool) external

46        - Rename `calculateGaugeAllocation` to `calculateGaugeStoredAllocation` to make clear that it reads from stored weights.
47        - Add `calculateGaugeAllocation` helper function that reads from current weight.
48:       - Add `isDeprecatedGauge(address)->bool` view function that returns true if gauge is deprecated.

```
*GitHub*: [35](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L35-L36), [32](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L32-L33), [33](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L33-L34), [34](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L34-L35), [35](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L35-L36), [46](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L46-L48)

```solidity
File: src/tokens/ERC20MultiVotes.sol

362   
363       function _writeCheckpoint(
364           address delegatee,
365:          function(uint256, uint256) view returns (uint256) op,

24            -> permissioned public functions to add in parent:
25:               - function setMaxDelegates(uint256) external

25                - function setMaxDelegates(uint256) external
26:               - function setContractExceedMaxDelegates(address,bool) external

```
*GitHub*: [362](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L362-L365), [24](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L24-L25), [25](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L25-L26)



### [N&#x2011;03] Cast to `bytes` or `bytes32` for clearer semantic meaning
Using a [cast](https://ethereum.stackexchange.com/questions/30912/how-to-compare-strings-in-solidity#answer-82739) on a single argument, rather than `abi.encodePacked()` makes the intended operation more clear, leading to less reviewer confusion.

*There is one instance of this issue:*

```solidity
File: src/governance/GuildVetoGovernor.sol

389:             string(abi.encodePacked(timelockId))

```
*GitHub*: [389](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L389-L389)



### [N&#x2011;04] Common functions should be refactored to a common base contract
The functions below have the same implementation as is seen in other files. The functions should be refactored into functions of a common base contract

*There are 2 instances of this issue:*

```solidity
File: src/governance/GuildGovernor.sol

68       function _setQuorum(uint256 newQuorum) internal {
69           emit QuorumUpdated(_quorum, newQuorum);
70           _quorum = newQuorum;
71:      }

```
*GitHub*: [68](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildGovernor.sol#L68-L71)

```solidity
File: src/governance/GuildVetoGovernor.sol

71       function _setQuorum(uint256 newQuorum) internal virtual {
72           emit QuorumUpdated(_quorum, newQuorum);
73           _quorum = newQuorum;
74:      }

```
*GitHub*: [71](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L71-L74)



### [N&#x2011;05] Consider emitting an event at the end of the constructor
This will allow users to easily exactly pinpoint when and by whom a contract was constructed

*There are 15 instances of this issue:*

<details>
<summary>see instances</summary>


```solidity
File: src/core/Core.sol

12:      constructor() {

```
*GitHub*: [12](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/Core.sol#L12-L12)

```solidity
File: src/core/CoreRef.sol

19:      constructor(address coreAddress) {

```
*GitHub*: [19](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRef.sol#L19-L19)

```solidity
File: src/governance/GuildGovernor.sol

30       constructor(
31           address _core,
32           address _timelock,
33           address _token,
34           uint256 initialVotingDelay,
35           uint256 initialVotingPeriod,
36           uint256 initialProposalThreshold,
37           uint256 initialQuorum
38       )
39           CoreRef(_core)
40           Governor("ECG Governor")
41           GovernorVotes(IVotes(_token))
42           GovernorTimelockControl(TimelockController(payable(_timelock)))
43           GovernorSettings(
44               initialVotingDelay,
45               initialVotingPeriod,
46               initialProposalThreshold
47           )
48:      {

```
*GitHub*: [30](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildGovernor.sol#L30-L48)

```solidity
File: src/governance/GuildTimelockController.sol

15       constructor(
16           address _core,
17           uint256 _minDelay
18       )
19           CoreRef(_core)
20           TimelockController(
21               _minDelay,
22               new address[](0),
23               new address[](0),
24               address(0)
25           )
26:      {}

```
*GitHub*: [15](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildTimelockController.sol#L15-L26)

```solidity
File: src/governance/GuildVetoGovernor.sol

32       constructor(
33           address _core,
34           address initialTimelock,
35           address _token,
36           uint256 initialQuorum
37       )
38           CoreRef(_core)
39           Governor("ECG Veto Governor")
40           GovernorVotes(IVotes(_token))
41:      {

```
*GitHub*: [32](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L32-L41)

```solidity
File: src/governance/LendingTermOffboarding.sol

66       constructor(
67           address _core,
68           address _guildToken,
69           address _psm,
70           uint256 _quorum
71:      ) CoreRef(_core) {

```
*GitHub*: [66](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOffboarding.sol#L66-L71)

```solidity
File: src/governance/LendingTermOnboarding.sol

63       constructor(
64           LendingTerm.LendingTermReferences memory _lendingTermReferences,
65           uint256 _gaugeType,
66           address _core,
67           address _timelock,
68           uint256 initialVotingDelay,
69           uint256 initialVotingPeriod,
70           uint256 initialProposalThreshold,
71           uint256 initialQuorum
72       )
73           GuildGovernor(
74               _core,
75               _timelock,
76               _lendingTermReferences.guildToken,
77               initialVotingDelay,
78               initialVotingPeriod,
79               initialProposalThreshold,
80               initialQuorum
81           )
82:      {

```
*GitHub*: [63](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOnboarding.sol#L63-L82)

```solidity
File: src/loan/AuctionHouse.sol

56       constructor(
57           address _core,
58           uint256 _midPoint,
59           uint256 _auctionDuration
60:      ) CoreRef(_core) {

```
*GitHub*: [56](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/AuctionHouse.sol#L56-L60)

```solidity
File: src/loan/LendingTerm.sol

153      constructor() CoreRef(address(1)) {
154          // core is set to address(1) to prevent implementation from being initialized,
155          // only proxies on the implementation can be initialized.
156:     }

```
*GitHub*: [153](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L153-L156)

```solidity
File: src/loan/SimplePSM.sol

65       constructor(
66           address _core,
67           address _profitManager,
68           address _credit,
69           address _pegToken
70:      ) CoreRef(_core) {

```
*GitHub*: [65](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SimplePSM.sol#L65-L70)

```solidity
File: src/loan/SurplusGuildMinter.sol

88       constructor(
89           address _core,
90           address _profitManager,
91           address _credit,
92           address _guild,
93           address _rlgm,
94           uint256 _mintRatio,
95           uint256 _rewardRatio
96:      ) CoreRef(_core) {

```
*GitHub*: [88](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L88-L96)

```solidity
File: src/rate-limits/RateLimitedMinter.sol

30       constructor(
31           address _core,
32           address _token,
33           bytes32 _role,
34           uint256 _maxRateLimitPerSecond,
35           uint128 _rateLimitPerSecond,
36           uint128 _bufferCap
37       )
38           CoreRef(_core)
39           RateLimitedV2(_maxRateLimitPerSecond, _rateLimitPerSecond, _bufferCap)
40:      {

```
*GitHub*: [30](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/rate-limits/RateLimitedMinter.sol#L30-L40)

```solidity
File: src/tokens/CreditToken.sol

24       constructor(
25           address _core,
26           string memory _name,
27           string memory _symbol
28       )
29           CoreRef(_core)
30           ERC20(_name, _symbol)
31           ERC20Permit(_name)
32:      {}

```
*GitHub*: [24](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/CreditToken.sol#L24-L32)

```solidity
File: src/tokens/GuildToken.sol

43       constructor(
44           address _core,
45           address _profitManager
46       )
47           CoreRef(_core)
48           ERC20("Ethereum Credit Guild - GUILD", "GUILD")
49           ERC20Permit("Ethereum Credit Guild - GUILD")
50:      {

```
*GitHub*: [43](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L43-L50)

```solidity
File: src/utils/RateLimitedV2.sol

40       constructor(
41           uint256 _maxRateLimitPerSecond,
42           uint128 _rateLimitPerSecond,
43           uint128 _bufferCap
44:      ) {

```
*GitHub*: [40](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/utils/RateLimitedV2.sol#L40-L44)

</details>





### [N&#x2011;06] Consider making contracts `Upgradeable`
This allows for bugs to be fixed in production, at the expense of _significantly_ increasing centralization.

*There are 14 instances of this issue:*

<details>
<summary>see instances</summary>


```solidity
File: src/core/Core.sol

10   contract Core is AccessControlEnumerable {
11:      /// @notice construct Core

```
*GitHub*: [10](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/Core.sol#L10-L11)

```solidity
File: src/governance/GuildGovernor.sol

16   contract GuildGovernor is
17       CoreRef,
18       Governor,
19       GovernorVotes,
20       GovernorTimelockControl,
21       GovernorSettings,
22       GovernorCountingSimple
23   {
24:      /// @notice Private storage variable for quorum (the minimum number of votes needed for a vote to pass).

```
*GitHub*: [16](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildGovernor.sol#L16-L24)

```solidity
File: src/governance/GuildTimelockController.sol

14:  contract GuildTimelockController is TimelockController, CoreRef {

```
*GitHub*: [14](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildTimelockController.sol#L14-L14)

```solidity
File: src/governance/GuildVetoGovernor.sol

25   contract GuildVetoGovernor is CoreRef, Governor, GovernorVotes {
26:      /// @notice Private storage variable for quorum (the minimum number of votes needed for a vote to pass).

```
*GitHub*: [25](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L25-L26)

```solidity
File: src/governance/LendingTermOffboarding.sol

16   contract LendingTermOffboarding is CoreRef {
17:      /// @notice emitted when a user supports the removal of a lending term

```
*GitHub*: [16](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOffboarding.sol#L16-L17)

```solidity
File: src/governance/LendingTermOnboarding.sol

23   contract LendingTermOnboarding is GuildGovernor {
24:      /// @notice minimum delay between proposals of onboarding of a given term

```
*GitHub*: [23](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOnboarding.sol#L23-L24)

```solidity
File: src/governance/ProfitManager.sol

30   contract ProfitManager is CoreRef {
31:      /// @notice reference to GUILD token.

```
*GitHub*: [30](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L30-L31)

```solidity
File: src/loan/AuctionHouse.sol

10   contract AuctionHouse is CoreRef {
11:      /// @notice emitted when au action starts

```
*GitHub*: [10](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/AuctionHouse.sol#L10-L11)

```solidity
File: src/loan/LendingTerm.sol

20:  contract LendingTerm is CoreRef {

```
*GitHub*: [20](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L20-L20)

```solidity
File: src/loan/SimplePSM.sol

25:  contract SimplePSM is CoreRef {

```
*GitHub*: [25](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SimplePSM.sol#L25-L25)

```solidity
File: src/loan/SurplusGuildMinter.sol

24   contract SurplusGuildMinter is CoreRef {
25:      /// @notice minimum number of CREDIT to stake

```
*GitHub*: [24](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L24-L25)

```solidity
File: src/rate-limits/RateLimitedMinter.sol

17   contract RateLimitedMinter is RateLimitedV2 {
18:      /// @notice the reference to token

```
*GitHub*: [17](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/rate-limits/RateLimitedMinter.sol#L17-L18)

```solidity
File: src/tokens/CreditToken.sol

18   contract CreditToken is
19       CoreRef,
20       ERC20Burnable,
21       ERC20MultiVotes,
22       ERC20RebaseDistributor
23:  {

```
*GitHub*: [18](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/CreditToken.sol#L18-L23)

```solidity
File: src/tokens/GuildToken.sol

37:  contract GuildToken is CoreRef, ERC20Burnable, ERC20Gauges, ERC20MultiVotes {

```
*GitHub*: [37](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L37-L37)

</details>





### [N&#x2011;07] Consider moving duplicated strings to constants


*There are 45 instances of this issue:*

<details>
<summary>see instances</summary>


```solidity
File: src/core/CoreRef.sol

25:          require(_core.hasRole(role, msg.sender), "UNAUTHORIZED");

```
*GitHub*: [25](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRef.sol#L25-L25)

```solidity
File: src/core/CoreRoles.sol

71:      bytes32 internal constant TIMELOCK_PROPOSER = keccak256("PROPOSER_ROLE");

74:      bytes32 internal constant TIMELOCK_EXECUTOR = keccak256("EXECUTOR_ROLE");

77:      bytes32 internal constant TIMELOCK_CANCELLER = keccak256("CANCELLER_ROLE");

```
*GitHub*: [71](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRoles.sol#L71-L71), [74](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRoles.sol#L74-L74), [77](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRoles.sol#L77-L77)

```solidity
File: src/governance/LendingTermOffboarding.sol

154:         require(canOffboard[term], "LendingTermOffboarding: quorum not met");

176:         require(canOffboard[term], "LendingTermOffboarding: quorum not met");

```
*GitHub*: [154](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOffboarding.sol#L154-L154), [176](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOffboarding.sol#L176-L176)

```solidity
File: src/governance/ProfitManager.sol

203:             require(otherSplit == 0, "GuildToken: invalid config");

205:             require(otherSplit != 0, "GuildToken: invalid config");

209:             "GuildToken: invalid config"

```
*GitHub*: [203](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L203-L203), [205](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L205-L205), [209](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L209-L209)

```solidity
File: src/loan/LendingTerm.sol

392:             require(_debtCeiling != 0, "LendingTerm: debt ceiling reached");

396:                 "LendingTerm: debt ceiling reached"

458:         require(loan.borrowTime != 0, "LendingTerm: loan not found");

459:         require(loan.closeTime == 0, "LendingTerm: loan closed");

460:         require(loan.callTime == 0, "LendingTerm: loan called");

499:         require(borrowTime != 0, "LendingTerm: loan not found");

502:             "LendingTerm: loan opened in same block"

504:         require(loan.closeTime == 0, "LendingTerm: loan closed");

505:         require(loan.callTime == 0, "LendingTerm: loan called");

572:         require(borrowTime != 0, "LendingTerm: loan not found");

575:             "LendingTerm: loan opened in same block"

577:         require(loan.closeTime == 0, "LendingTerm: loan closed");

578:         require(loan.callTime == 0, "LendingTerm: loan called");

643:         require(loan.borrowTime != 0, "LendingTerm: loan not found");

646:         require(loan.closeTime == 0, "LendingTerm: loan closed");

649:         require(loan.callTime == 0, "LendingTerm: loan called");

661:             "LendingTerm: loan opened in same block"

699:         require(loan.borrowTime != 0, "LendingTerm: loan not found");

702:         require(loan.closeTime == 0, "LendingTerm: loan closed");

738:         require(loans[loanId].closeTime == 0, "LendingTerm: loan closed");

```
*GitHub*: [392](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L392-L392), [396](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L396-L396), [458](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L458-L458), [459](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L459-L459), [460](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L460-L460), [499](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L499-L499), [502](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L502-L502), [504](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L504-L504), [505](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L505-L505), [572](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L572-L572), [575](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L575-L575), [577](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L577-L577), [578](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L578-L578), [643](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L643-L643), [646](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L646-L646), [649](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L649-L649), [661](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L661-L661), [699](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L699-L699), [702](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L702-L702), [738](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L738-L738)

```solidity
File: src/tokens/ERC20Gauges.sol

223:         require(isGauge(gauge), "ERC20Gauges: invalid gauge");

274:         require(weights.length == size, "ERC20Gauges: size mismatch");

285:             require(isGauge(gauge), "ERC20Gauges: invalid gauge");

345:         require(weights.length == size, "ERC20Gauges: size mismatch");

404:             "ERC20Gauges: invalid gauge"

429:             "ERC20Gauges: invalid gauge"

```
*GitHub*: [223](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L223-L223), [274](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L274-L274), [285](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L285-L285), [345](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L345-L345), [404](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L404-L404), [429](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L429-L429)

```solidity
File: src/tokens/ERC20MultiVotes.sol

297:         require(count < 2, "ERC20MultiVotes: delegation error");

326:             "ERC20MultiVotes: delegation error"

334:             "ERC20MultiVotes: delegation error"

```
*GitHub*: [297](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L297-L297), [326](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L326-L326), [334](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L334-L334)

```solidity
File: src/tokens/GuildToken.sol

48:          ERC20("Ethereum Credit Guild - GUILD", "GUILD")

49:          ERC20Permit("Ethereum Credit Guild - GUILD")

124:         require(msg.sender == profitManager, "UNAUTHORIZED");

216:             "GuildToken: pending loss"

254:                 "GuildToken: pending loss"

```
*GitHub*: [48](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L48-L48), [49](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L49-L49), [124](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L124-L124), [216](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L216-L216), [254](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L254-L254)

```solidity
File: src/utils/RateLimitedV2.sol

52:              "RateLimited: rateLimitPerSecond too high"

66:              "RateLimited: rateLimitPerSecond too high"

```
*GitHub*: [52](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/utils/RateLimitedV2.sol#L52-L52), [66](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/utils/RateLimitedV2.sol#L66-L66)

</details>





### [N&#x2011;08] Consider returning a `struct` rather than having multiple `return` values


*There is one instance of this issue:*

```solidity
File: src/loan/SurplusGuildMinter.sol

216      function getRewards(
217          address user,
218          address term
219      )
220          public
221          returns (
222              uint256 lastGaugeLoss, // GuildToken.lastGaugeLoss(term)
223              UserStake memory userStake, // stake state after execution of getRewards()
224              bool slashed // true if the user has been slashed
225          )
226:     {

```
*GitHub*: [216](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L216-L226)



### [N&#x2011;09] Consider splitting complex checks into multiple steps
Assign the expression's parts to intermediate local variables, and check against those instead

*There is one instance of this issue:*

```solidity
File: src/tokens/ERC20Gauges.sol

403:             gauge != address(0) && (newAdd || previouslyDeprecated),

```
*GitHub*: [403](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L403-L403)



### [N&#x2011;10] Consider using `AccessControlDefaultAdminRules` rather than `AccessControl`
`AccessControlDefaultAdminRules` implements multiple [security best practices](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControlDefaultAdminRules) on top of the normal `AccessControl` rules, so consider using it instead, or adding it to the contract inheritance constructor tree of any contract that extends it

*There are 2 instances of this issue:*

```solidity
File: src/core/Core.sol

10   contract Core is AccessControlEnumerable {
11:      /// @notice construct Core

```
*GitHub*: [10](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/Core.sol#L10-L11)

```solidity
File: src/governance/GuildTimelockController.sol

14:  contract GuildTimelockController is TimelockController, CoreRef {

```
*GitHub*: [14](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildTimelockController.sol#L14-L14)



### [N&#x2011;11] Consider using descriptive `constant`s when passing zero as a function argument
Passing zero as a function argument can sometimes result in a security issue (e.g. passing zero as the slippage parameter). Consider using a `constant` variable with a descriptive name, so it's clear that the argument is intentionally being used, and for the right reasons.

*There are 24 instances of this issue:*

<details>
<summary>see instances</summary>


```solidity
File: src/governance/GuildTimelockController.sol

22:              new address[](0),

23:              new address[](0),

```
*GitHub*: [22](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildTimelockController.sol#L22-L22), [23](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildTimelockController.sol#L23-L23)

```solidity
File: src/governance/LendingTermOffboarding.sol

110:             address(0),

```
*GitHub*: [110](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOffboarding.sol#L110-L110)

```solidity
File: src/governance/ProfitManager.sol

311:                 emit TermSurplusBufferUpdate(block.timestamp, gauge, 0);

328:                 emit SurplusBufferUpdate(block.timestamp, 0);

```
*GitHub*: [311](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L311-L311), [328](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L328-L328)

```solidity
File: src/loan/AuctionHouse.sol

98:              endTime: 0,

227:             0, // collateralSold

228:             0 // debtRecovered

```
*GitHub*: [98](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/AuctionHouse.sol#L98-L98), [227](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/AuctionHouse.sol#L227-L227), [228](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/AuctionHouse.sol#L228-L228)

```solidity
File: src/loan/LendingTerm.sol

407:             caller: address(0),

408:             callTime: 0,

409:             callDebt: 0,

410:             closeTime: 0

721:         emit LoanClose(block.timestamp, loanId, LoanCloseType.Forgive, 0);

```
*GitHub*: [407](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L407-L407), [408](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L408-L408), [409](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L409-L409), [410](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L410-L410), [721](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L721-L721)

```solidity
File: src/loan/SurplusGuildMinter.sol

277:                 stakeTime: uint48(0),

278:                 lastGaugeLoss: uint48(0),

279:                 profitIndex: uint160(0),

280:                 credit: uint128(0),

281:                 guild: uint128(0)

```
*GitHub*: [277](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L277-L277), [278](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L278-L278), [279](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L279-L279), [280](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L280-L280), [281](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L281-L281)

```solidity
File: src/tokens/ERC20RebaseDistributor.sol

121:             lastValue: 0,

123:             targetValue: 0

198:                 lastValue: 0,

200:                 targetValue: 0

329:         rebasingState[account] = RebasingState({isRebasing: 0, nShares: 0});

329:         rebasingState[account] = RebasingState({isRebasing: 0, nShares: 0});

```
*GitHub*: [121](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L121-L121), [123](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L123-L123), [198](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L198-L198), [200](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L200-L200), [329](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L329-L329), [329](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L329-L329)

</details>





### [N&#x2011;12] Consider using named function arguments
When calling functions in external contracts with multiple arguments, consider using [named](https://docs.soliditylang.org/en/latest/control-structures.html#function-calls-with-named-parameters) function parameters, rather than positional ones.

*There are 31 instances of this issue:*

<details>
<summary>see instances</summary>


```solidity
File: src/core/CoreRef.sol

25:          require(_core.hasRole(role, msg.sender), "UNAUTHORIZED");

```
*GitHub*: [25](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRef.sol#L25-L25)

```solidity
File: src/governance/GuildTimelockController.sol

34:          return core().hasRole(role, account);

```
*GitHub*: [34](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildTimelockController.sol#L34-L34)

```solidity
File: src/governance/LendingTermOffboarding.sol

126          uint256 userWeight = GuildToken(guildToken).getPastVotes(
127              msg.sender,
128              snapshotBlock
129:         );

187:         core().revokeRole(CoreRoles.RATE_LIMITED_CREDIT_MINTER, term);

188:         core().revokeRole(CoreRoles.GAUGE_PNL_NOTIFIER, term);

```
*GitHub*: [126](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOffboarding.sol#L126-L129), [187](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOffboarding.sol#L187-L187), [188](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOffboarding.sol#L188-L188)

```solidity
File: src/governance/LendingTermOnboarding.sol

154          LendingTerm(term).initialize(
155              address(core()),
156              LendingTerm.LendingTermReferences({
157                  profitManager: profitManager,
158                  guildToken: guildToken,
159                  auctionHouse: auctionHouse,
160                  creditMinter: creditMinter,
161                  creditToken: creditToken
162              }),
163              params
164:         );

```
*GitHub*: [154](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOnboarding.sol#L154-L164)

```solidity
File: src/loan/AuctionHouse.sol

78:              core().hasRole(CoreRoles.GAUGE_PNL_NOTIFIER, msg.sender),

180          LendingTerm(_lendingTerm).onBid(
181              loanId,
182              msg.sender,
183              auctions[loanId].collateralAmount - collateralReceived, // collateralToBorrower
184              collateralReceived, // collateralToBidder
185              creditAsked // creditFromBidder
186:         );

214          LendingTerm(_lendingTerm).onBid(
215              loanId,
216              msg.sender,
217              0, // collateralToBorrower
218              0, // collateralToBidder
219              0 // creditFromBidder
220:         );

```
*GitHub*: [78](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/AuctionHouse.sol#L78-L78), [180](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/AuctionHouse.sol#L180-L186), [214](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/AuctionHouse.sol#L214-L220)

```solidity
File: src/loan/LendingTerm.sol

383          uint256 _debtCeiling = (GuildToken(refs.guildToken)
384              .calculateGaugeAllocation(
385                  address(this),
386                  totalBorrowedCredit + borrowAmount
387:             ) * gaugeWeightTolerance) / 1e18;

418:         RateLimitedMinter(refs.creditMinter).mint(borrower, borrowAmount);

550          ProfitManager(refs.profitManager).notifyPnL(
551              address(this),
552              int256(interestRepaid)
553:         );

603              ProfitManager(refs.profitManager).notifyPnL(
604                  address(this),
605                  int256(interest)
606:             );

671:         AuctionHouse(_auctionHouse).startAuction(loanId, loanDebt);

715:         ProfitManager(refs.profitManager).notifyPnL(address(this), pnl);

797:             ProfitManager(refs.profitManager).notifyPnL(address(this), pnl);

```
*GitHub*: [383](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L383-L387), [418](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L418-L418), [550](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L550-L553), [603](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L603-L606), [671](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L671-L671), [715](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L715-L715), [797](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L797-L797)

```solidity
File: src/loan/SimplePSM.sol

110:         CreditToken(credit).mint(to, amountOut);

127:         CreditToken(credit).mint(msg.sender, amountOut);

140:         CreditToken(credit).burnFrom(msg.sender, amountIn);

```
*GitHub*: [110](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SimplePSM.sol#L110-L110), [127](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SimplePSM.sol#L127-L127), [140](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SimplePSM.sol#L140-L140)

```solidity
File: src/loan/SurplusGuildMinter.sol

130:         ProfitManager(profitManager).donateToTermSurplusBuffer(term, amount);

135:         RateLimitedMinter(rlgm).mint(address(this), guildAmount);

136:         GuildToken(guild).incrementGauge(term, guildAmount);

199          ProfitManager(profitManager).withdrawFromTermSurplusBuffer(
200              term,
201              msg.sender,
202              amount
203:         );

206:         GuildToken(guild).decrementGauge(term, guildAmount);

259:                 RateLimitedMinter(rlgm).mint(user, guildReward);

305:             RateLimitedMinter(rlgm).mint(address(this), guildAmount);

306:             GuildToken(guild).incrementGauge(term, guildAmount);

310:             GuildToken(guild).decrementGauge(term, guildAmount);

```
*GitHub*: [130](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L130-L130), [135](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L135-L135), [136](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L136-L136), [199](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L199-L203), [206](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L206-L206), [259](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L259-L259), [305](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L305-L305), [306](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L306-L306), [310](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L310-L310)

```solidity
File: src/rate-limits/RateLimitedMinter.sol

54:          IERC20Mintable(token).mint(to, amount); /// interactions

```
*GitHub*: [54](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/rate-limits/RateLimitedMinter.sol#L54-L54)

```solidity
File: src/tokens/GuildToken.sol

220:         ProfitManager(profitManager).claimGaugeRewards(user, gauge);

258:         ProfitManager(profitManager).claimGaugeRewards(user, gauge);

```
*GitHub*: [220](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L220-L220), [258](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L258-L258)

</details>





### [N&#x2011;13] Consider using named returns
Using named returns makes the code more self-documenting, makes it easier to fill out NatSpec, and in some cases can save gas. The cases below are where there currently is at most one return statement, which is ideal for named returns.

*There are 80 instances of this issue:*

<details>
<summary>see instances</summary>


```solidity
File: src/core/CoreRef.sol

30:      function core() public view returns (Core) {

```
*GitHub*: [30](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRef.sol#L30-L30)

```solidity
File: src/governance/GuildGovernor.sol

57       function quorum(
58           uint256 /* blockNumber*/
59:      ) public view override returns (uint256) {

110      function guardianCancel(
111          address[] memory targets,
112          uint256[] memory values,
113          bytes[] memory calldatas,
114          bytes32 descriptionHash
115:     ) public onlyCoreRole(CoreRoles.GUARDIAN) returns (uint256) {

123      function _cancel(
124          address[] memory targets,
125          uint256[] memory values,
126          bytes[] memory calldatas,
127          bytes32 descriptionHash
128:     ) internal override(Governor, GovernorTimelockControl) returns (uint256) {

142      function _executor()
143          internal
144          view
145          override(Governor, GovernorTimelockControl)
146          returns (address)
147:     {

151      function proposalThreshold()
152          public
153          view
154          override(Governor, GovernorSettings)
155          returns (uint256)
156:     {

160      function state(
161          uint256 proposalId
162      )
163          public
164          view
165          override(Governor, GovernorTimelockControl)
166          returns (ProposalState)
167:     {

171      function supportsInterface(
172          bytes4 interfaceId
173:     ) public view override(Governor, GovernorTimelockControl) returns (bool) {

```
*GitHub*: [57](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildGovernor.sol#L57-L59), [110](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildGovernor.sol#L110-L115), [123](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildGovernor.sol#L123-L128), [142](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildGovernor.sol#L142-L147), [151](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildGovernor.sol#L151-L156), [160](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildGovernor.sol#L160-L167), [171](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildGovernor.sol#L171-L173)

```solidity
File: src/governance/GuildTimelockController.sol

30       function hasRole(
31           bytes32 role,
32           address account
33:      ) public view virtual override returns (bool) {

```
*GitHub*: [30](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildTimelockController.sol#L30-L33)

```solidity
File: src/governance/GuildVetoGovernor.sol

53       function quorum(
54           uint256 /* blockNumber*/
55:      ) public view override returns (uint256) {

130      function COUNTING_MODE()
131          public
132          pure
133          virtual
134          override
135          returns (string memory)
136:     {

143      function hasVoted(
144          uint256 proposalId,
145          address account
146:     ) public view virtual override returns (bool) {

172      function _quorumReached(
173          uint256 proposalId
174:     ) internal view virtual override returns (bool) {

186      function _voteSucceeded(
187          uint256 /* proposalId*/
188:     ) internal pure virtual override returns (bool) {

222:     function votingDelay() public pure override returns (uint256) {

230:     function votingPeriod() public pure override returns (uint256) {

304      function propose(
305          address[] memory /* targets*/,
306          uint256[] memory /* values*/,
307          bytes[] memory /* calldatas*/,
308          string memory /* description*/
309:     ) public pure override returns (uint256) {

315:     function createVeto(bytes32 timelockId) external returns (uint256) {

349:     function executeVeto(bytes32 timelockId) external returns (uint256) {

```
*GitHub*: [53](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L53-L55), [130](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L130-L136), [143](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L143-L146), [172](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L172-L174), [186](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L186-L188), [222](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L222-L222), [230](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L230-L230), [304](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L304-L309), [315](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L315-L315), [349](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L349-L349)

```solidity
File: src/governance/LendingTermOnboarding.sol

105      function createTerm(
106          address implementation,
107          LendingTerm.LendingTermParams calldata params
108:     ) external returns (address) {

171      function propose(
172          address[] memory /* targets*/,
173          uint256[] memory /* values*/,
174          bytes[] memory /* calldatas*/,
175          string memory /* description*/
176:     ) public pure override(IGovernor, Governor) returns (uint256) {

```
*GitHub*: [105](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOnboarding.sol#L105-L108), [171](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOnboarding.sol#L171-L176)

```solidity
File: src/governance/ProfitManager.sol

151:     function minBorrow() external view returns (uint256) {

172:     function totalBorrowedCredit() external view returns (uint256) {

```
*GitHub*: [151](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L151-L151), [172](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L172-L172)

```solidity
File: src/loan/AuctionHouse.sol

67:      function getAuction(bytes32 loanId) external view returns (Auction memory) {

```
*GitHub*: [67](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/AuctionHouse.sol#L67-L67)

```solidity
File: src/loan/LendingTerm.sol

176      function getReferences()
177          external
178          view
179          returns (LendingTermReferences memory)
180:     {

185:     function getParameters() external view returns (LendingTermParams memory) {

190:     function collateralToken() external view returns (address) {

195:     function getLoan(bytes32 loanId) external view returns (Loan memory) {

334:     function debtCeiling() external view returns (uint256) {

```
*GitHub*: [176](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L176-L180), [185](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L185-L185), [190](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L190-L190), [195](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L195-L195), [334](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L334-L334)

```solidity
File: src/loan/SimplePSM.sol

80:      function getMintAmountOut(uint256 amountIn) public view returns (uint256) {

87       function getRedeemAmountOut(
88           uint256 amountIn
89:      ) public view returns (uint256) {

97:      function redeemableCredit() public view returns (uint256) {

```
*GitHub*: [80](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SimplePSM.sol#L80-L80), [87](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SimplePSM.sol#L87-L89), [97](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SimplePSM.sol#L97-L97)

```solidity
File: src/loan/SurplusGuildMinter.sol

106      function getUserStake(
107          address user,
108          address term
109:     ) external view returns (UserStake memory) {

```
*GitHub*: [106](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L106-L109)

```solidity
File: src/tokens/CreditToken.sol

98       function balanceOf(
99           address account
100:     ) public view override(ERC20, ERC20RebaseDistributor) returns (uint256) {

104      function totalSupply()
105          public
106          view
107          override(ERC20, ERC20RebaseDistributor)
108          returns (uint256)
109:     {

113      function transfer(
114          address to,
115          uint256 amount
116      )
117          public
118          override(ERC20, ERC20MultiVotes, ERC20RebaseDistributor)
119          returns (bool)
120:     {

125      function transferFrom(
126          address from,
127          address to,
128          uint256 amount
129      )
130          public
131          override(ERC20, ERC20MultiVotes, ERC20RebaseDistributor)
132          returns (bool)
133:     {

```
*GitHub*: [98](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/CreditToken.sol#L98-L100), [104](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/CreditToken.sol#L104-L109), [113](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/CreditToken.sol#L113-L120), [125](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/CreditToken.sol#L125-L133)

```solidity
File: src/tokens/ERC20Gauges.sol

98:      function gauges() external view returns (address[] memory) {

103:     function isGauge(address gauge) public view returns (bool) {

108:     function isDeprecatedGauge(address gauge) public view returns (bool) {

113:     function numGauges() external view returns (uint256) {

118:     function deprecatedGauges() external view returns (address[] memory) {

123:     function numDeprecatedGauges() external view returns (uint256) {

149:     function numLiveGauges() external view returns (uint256) {

154:     function userGauges(address user) external view returns (address[] memory) {

159      function isUserGauge(
160          address user,
161          address gauge
162:     ) external view returns (bool) {

167:     function numUserGauges(address user) external view returns (uint256) {

172:     function userUnusedWeight(address user) external view returns (uint256) {

481      function transfer(
482          address to,
483          uint256 amount
484:     ) public virtual override returns (bool) {

489      function transferFrom(
490          address from,
491          address to,
492          uint256 amount
493:     ) public virtual override returns (bool) {

```
*GitHub*: [98](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L98-L98), [103](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L103-L103), [108](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L108-L108), [113](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L113-L113), [118](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L118-L118), [123](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L123-L123), [149](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L149-L149), [154](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L154-L154), [159](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L159-L162), [167](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L167-L167), [172](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L172-L172), [481](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L481-L484), [489](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L489-L493)

```solidity
File: src/tokens/ERC20MultiVotes.sol

52       function checkpoints(
53           address account,
54           uint32 pos
55:      ) public view virtual returns (Checkpoint memory) {

60       function numCheckpoints(
61           address account
62:      ) public view virtual returns (uint32) {

71:      function freeVotes(address account) public view virtual returns (uint256) {

80:      function getVotes(address account) public view virtual returns (uint256) {

91       function getPastVotes(
92           address account,
93           uint256 blockNumber
94:      ) public view virtual returns (uint256) {

103      function _checkpointsLookup(
104          Checkpoint[] storage ckpts,
105          uint256 blockNumber
106:     ) private view returns (uint256) {

122:     function average(uint256 a, uint256 b) internal pure returns (uint256) {

218      function delegatesVotesCount(
219          address delegator,
220          address delegatee
221:     ) public view virtual returns (uint256) {

230      function delegates(
231          address delegator
232:     ) public view returns (address[] memory) {

242      function containsDelegate(
243          address delegator,
244          address delegatee
245:     ) public view returns (bool) {

254:     function delegateCount(address delegator) public view returns (uint256) {

387:     function _add(uint256 a, uint256 b) private pure returns (uint256) {

391:     function _subtract(uint256 a, uint256 b) private pure returns (uint256) {

408      function transfer(
409          address to,
410          uint256 amount
411:     ) public virtual override returns (bool) {

416      function transferFrom(
417          address from,
418          address to,
419          uint256 amount
420:     ) public virtual override returns (bool) {

```
*GitHub*: [52](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L52-L55), [60](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L60-L62), [71](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L71-L71), [80](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L80-L80), [91](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L91-L94), [103](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L103-L106), [122](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L122-L122), [218](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L218-L221), [230](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L230-L232), [242](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L242-L245), [254](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L254-L254), [387](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L387-L387), [391](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L391-L391), [408](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L408-L411), [416](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L416-L420)

```solidity
File: src/tokens/ERC20RebaseDistributor.sol

242:     function rebasingSharePrice() internal view returns (uint256) {

247:     function unmintedRebaseRewards() internal view returns (uint256) {

252      function _balance2shares(
253          uint256 balance,
254          uint256 sharePrice
255:     ) internal pure returns (uint256) {

260      function _shares2balance(
261          uint256 shares,
262          uint256 sharePrice,
263          uint256 deltaBalance,
264          uint256 minBalance
265:     ) internal pure returns (uint256) {

389:     function isRebasing(address account) public view returns (bool) {

394:     function rebasingSupply() public view returns (uint256) {

415:     function pendingDistributedSupply() external view returns (uint256) {

444:     function totalSupply() public view virtual override returns (uint256) {

451:     function targetTotalSupply() external view returns (uint256) {

553      function transfer(
554          address to,
555          uint256 amount
556:     ) public virtual override returns (bool) {

646      function transferFrom(
647          address from,
648          address to,
649          uint256 amount
650:     ) public virtual override returns (bool) {

```
*GitHub*: [242](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L242-L242), [247](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L247-L247), [252](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L252-L255), [260](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L260-L265), [389](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L389-L389), [394](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L394-L394), [415](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L415-L415), [444](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L444-L444), [451](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L451-L451), [553](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L553-L556), [646](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L646-L650)

```solidity
File: src/tokens/GuildToken.sol

76       function addGauge(
77           uint256 _type,
78           address gauge
79:      ) external onlyCoreRole(CoreRoles.GAUGE_ADD) returns (uint256) {

288      function transfer(
289          address to,
290          uint256 amount
291      )
292          public
293          virtual
294          override(ERC20, ERC20Gauges, ERC20MultiVotes)
295          returns (bool)
296:     {

302      function transferFrom(
303          address from,
304          address to,
305          uint256 amount
306      )
307          public
308          virtual
309          override(ERC20, ERC20Gauges, ERC20MultiVotes)
310          returns (bool)
311:     {

```
*GitHub*: [76](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L76-L79), [288](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L288-L296), [302](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L302-L311)

```solidity
File: src/utils/RateLimitedV2.sol

83:      function buffer() public view returns (uint256) {

```
*GitHub*: [83](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/utils/RateLimitedV2.sol#L83-L83)

</details>





### [N&#x2011;14] Consider using the `using`-`for` syntax
The `using`-`for` [syntax](https://docs.soliditylang.org/en/latest/contracts.html#using-for) is the more common way of calling library functions.

*There are 37 instances of this issue:*

```solidity
File: src/governance/LendingTermOnboarding.sol

153:         address term = Clones.clone(implementation);

232:             Strings.toString(block.number),

235:             Strings.toHexString(term)

```
*GitHub*: [153](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOnboarding.sol#L153-L153), [232](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOnboarding.sol#L232-L232), [235](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOnboarding.sol#L235-L235)

```solidity
File: src/loan/SurplusGuildMinter.sol

140:             stakeTime: SafeCastLib.safeCastTo48(block.timestamp),

141:             lastGaugeLoss: SafeCastLib.safeCastTo48(lastGaugeLoss),

142              profitIndex: SafeCastLib.safeCastTo160(
143                  ProfitManager(profitManager).userGaugeProfitIndex(
144                      address(this),
145                      term
146                  )
147:             ),

148:             credit: userStake.credit + SafeCastLib.safeCastTo128(amount),

149:             guild: userStake.guild + SafeCastLib.safeCastTo128(guildAmount)

181:         userStake.credit -= SafeCastLib.safeCastTo128(amount);

182:         userStake.guild -= SafeCastLib.safeCastTo128(guildAmount);

267:             userStake.profitIndex = SafeCastLib.safeCastTo160(_profitIndex);

307:             _stakes[user][term].guild = SafeCastLib.safeCastTo128(guildAfter);

313:             _stakes[user][term].guild = SafeCastLib.safeCastTo128(guildAfter);

```
*GitHub*: [140](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L140-L140), [141](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L141-L141), [142](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L142-L147), [148](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L148-L148), [149](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L149-L149), [181](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L181-L181), [182](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L182-L182), [267](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L267-L267), [307](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L307-L307), [313](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L313-L313)

```solidity
File: src/tokens/ERC20RebaseDistributor.sol

110:             lastTimestamp: SafeCastLib.safeCastTo32(block.timestamp),

112:             targetTimestamp: SafeCastLib.safeCastTo32(block.timestamp),

120:             lastTimestamp: SafeCastLib.safeCastTo32(block.timestamp),

122:             targetTimestamp: SafeCastLib.safeCastTo32(block.timestamp),

191:                 lastTimestamp: SafeCastLib.safeCastTo32(block.timestamp), // now

193:                 targetTimestamp: SafeCastLib.safeCastTo32(block.timestamp), // now

197:                 lastTimestamp: SafeCastLib.safeCastTo32(block.timestamp),

199:                 targetTimestamp: SafeCastLib.safeCastTo32(block.timestamp),

219:                 lastTimestamp: SafeCastLib.safeCastTo32(block.timestamp), // now

220:                 lastValue: SafeCastLib.safeCastTo224(currentRebasingSharePrice), // current value

222:                 targetValue: SafeCastLib.safeCastTo224(targetNewSharePrice) // adjusted target

232:             lastTimestamp: SafeCastLib.safeCastTo32(block.timestamp), // now

233              lastValue: SafeCastLib.safeCastTo224(
234                  _unmintedRebaseRewards - amount
235:             ), // adjusted current

237:             targetValue: val.targetValue - SafeCastLib.safeCastTo224(amount) // adjusted target

370:                 lastTimestamp: SafeCastLib.safeCastTo32(block.timestamp),

371:                 lastValue: SafeCastLib.safeCastTo224(_rebasingSharePrice),

372:                 targetTimestamp: SafeCastLib.safeCastTo32(endTimestamp),

373:                 targetValue: SafeCastLib.safeCastTo224(newTargetSharePrice)

379:                 lastTimestamp: SafeCastLib.safeCastTo32(block.timestamp),

380:                 lastValue: SafeCastLib.safeCastTo224(_unmintedRebaseRewards),

381:                 targetTimestamp: SafeCastLib.safeCastTo32(endTimestamp),

383:                     SafeCastLib.safeCastTo224(amount)

```
*GitHub*: [110](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L110-L110), [112](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L112-L112), [120](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L120-L120), [122](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L122-L122), [191](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L191-L191), [193](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L193-L193), [197](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L197-L197), [199](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L199-L199), [219](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L219-L219), [220](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L220-L220), [222](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L222-L222), [232](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L232-L232), [233](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L233-L235), [237](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L237-L237), [370](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L370-L370), [371](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L371-L371), [372](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L372-L372), [373](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L373-L373), [379](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L379-L379), [380](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L380-L380), [381](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L381-L381), [383](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L383-L383)

```solidity
File: src/utils/RateLimitedV2.sol

86:              Math.min(bufferStored + (rateLimitPerSecond * elapsed), bufferCap);

126          uint224 newBufferStored = Math
127:             .min(newBuffer + amount, _bufferCap)

```
*GitHub*: [86](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/utils/RateLimitedV2.sol#L86-L86), [126](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/utils/RateLimitedV2.sol#L126-L127)



### [N&#x2011;15] Expressions for constant values should use `immutable` rather than `constant`
While it does not save gas for some simple binary expressions because the compiler knows that developers often make this mistake, it's still best to use the right tool for the task at hand. There is a difference between `constant` variables and `immutable` variables, and they should each be used in their appropriate contexts. `constants` should be used for literal values written into the code, and `immutable` variables should be used for expressions, or values calculated in, or passed into the constructor.

*There are 2 instances of this issue:*

```solidity
File: src/tokens/ERC20RebaseDistributor.sol

108      InterpolatedValue internal __rebasingSharePrice =
109          InterpolatedValue({
110              lastTimestamp: SafeCastLib.safeCastTo32(block.timestamp),
111              lastValue: uint224(START_REBASING_SHARE_PRICE), // safe initial value
112              targetTimestamp: SafeCastLib.safeCastTo32(block.timestamp),
113              targetValue: uint224(START_REBASING_SHARE_PRICE) // safe initial value
114:         });

118      InterpolatedValue internal __unmintedRebaseRewards =
119          InterpolatedValue({
120              lastTimestamp: SafeCastLib.safeCastTo32(block.timestamp),
121              lastValue: 0,
122              targetTimestamp: SafeCastLib.safeCastTo32(block.timestamp),
123              targetValue: 0
124:         });

```
*GitHub*: [108](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L108-L114), [118](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L118-L124)



### [N&#x2011;16] Function state mutability can be restricted to `view`
Note that when overriding, mutability is allowed to be changed to be [more](https://docs.soliditylang.org/en/latest/contracts.html#function-overriding) strict than the parent function's mutability

*There are 3 instances of this issue:*

```solidity
File: src/governance/GuildTimelockController.sol

38:      function _setRoleAdmin(bytes32 role, bytes32 adminRole) internal override {}

41:      function _grantRole(bytes32 role, address account) internal override {}

44:      function _revokeRole(bytes32 role, address account) internal override {}

```
*GitHub*: [38](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildTimelockController.sol#L38-L38), [41](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildTimelockController.sol#L41-L41), [44](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildTimelockController.sol#L44-L44)



### [N&#x2011;17] Interfaces should use floating version pragmas
If the file contains non-interfaces as well, the interfaces should be moved to a separate file.

*There is one instance of this issue:*

```solidity
File: src/rate-limits/RateLimitedMinter.sol

/// @audit IERC20Mintable
2:   pragma solidity 0.8.13;

```
*GitHub*: [2](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/rate-limits/RateLimitedMinter.sol#L2-L2)



### [N&#x2011;18] Long functions should be refactored into multiple, smaller, functions


*There are 11 instances of this issue:*

```solidity
File: src/governance/GuildVetoGovernor.sol

/// @audit 56 lines (52 in the body)
246       function state(
247           uint256 proposalId
248:      ) public view override returns (ProposalState) {

```
*GitHub*: [246](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L246-L248)

```solidity
File: src/governance/LendingTermOnboarding.sol

/// @audit 64 lines (59 in the body)
105       function createTerm(
106           address implementation,
107           LendingTerm.LendingTermParams calldata params
108:      ) external returns (address) {

```
*GitHub*: [105](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOnboarding.sol#L105-L108)

```solidity
File: src/loan/LendingTerm.sol

/// @audit 62 lines (58 in the body)
270       function debtCeiling(
271           int256 gaugeWeightDelta
272:      ) public view returns (uint256) {

/// @audit 97 lines (91 in the body)
339       function _borrow(
340           address borrower,
341           uint256 borrowAmount,
342           uint256 collateralAmount
343:      ) internal returns (bytes32 loanId) {

/// @audit 70 lines (64 in the body)
490       function _partialRepay(
491           address repayer,
492           bytes32 loanId,
493:          uint256 debtToRepay

/// @audit 59 lines (57 in the body)
567:      function _repay(address repayer, bytes32 loanId) internal {

```
*GitHub*: [270](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L270-L272), [339](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L339-L343), [490](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L490-L493), [567](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L567)

```solidity
File: src/loan/SurplusGuildMinter.sol

/// @audit 55 lines (53 in the body)
158:      function unstake(address term, uint256 amount) external {

/// @audit 75 lines (63 in the body)
216       function getRewards(
217           address user,
218           address term
219       )
220           public
221           returns (
222               uint256 lastGaugeLoss, // GuildToken.lastGaugeLoss(term)
223               UserStake memory userStake, // stake state after execution of getRewards()
224:              bool slashed // true if the user has been slashed

```
*GitHub*: [158](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L158), [216](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L216-L224)

```solidity
File: src/tokens/ERC20RebaseDistributor.sol

/// @audit 58 lines (53 in the body)
168       function updateTotalRebasingShares(
169           uint256 currentRebasingSharePrice,
170:          int256 sharesDelta

/// @audit 90 lines (85 in the body)
553       function transfer(
554           address to,
555           uint256 amount
556:      ) public virtual override returns (bool) {

/// @audit 91 lines (85 in the body)
646       function transferFrom(
647           address from,
648           address to,
649           uint256 amount
650:      ) public virtual override returns (bool) {

```
*GitHub*: [168](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L168-L170), [553](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L553-L556), [646](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L646-L650)



### [N&#x2011;19] Multiple `address`/ID mappings can be combined into a single `mapping` of an `address`/ID to a `struct`, for readability
Well-organized data structures make code reviews easier, which may lead to fewer bugs. Consider combining related mappings into mappings to structs, so it's clear what data is related. The instances below refer to both mappings using the same key in the same function, so the mappings are related.

*There are 12 instances of this issue:*

```solidity
File: src/governance/LendingTermOnboarding.sol

/// @audit combine into a `struct`: created,lastProposal
181      function proposeOnboard(
182          address term
183      ) external whenNotPaused returns (uint256 proposalId) {
184          // Check that the term has been created by this factory
185          require(created[term] != 0, "LendingTermOnboarding: invalid term");
186  
187          // Check that the term was not subject to an onboard vote recently
188          require(
189              lastProposal[term] + MIN_DELAY_BETWEEN_PROPOSALS < block.timestamp,
190              "LendingTermOnboarding: recently proposed"
191          );
192          lastProposal[term] = block.timestamp;
193  
194          // Check that the term is not already active
195          // note that terms that have been offboarded in the past can be re-onboarded
196          // and won't fail this check. This is intentional, because some terms might be offboarded
197          // due to specific market conditions, and it might be desirable to re-onboard them
198          // at a later date.
199          bool isGauge = GuildToken(guildToken).isGauge(term);
200          require(!isGauge, "LendingTermOnboarding: active term");
201  
202          // Generate calldata for the proposal
203          (
204              address[] memory targets,
205              uint256[] memory values,
206              bytes[] memory calldatas,
207              string memory description
208          ) = getOnboardProposeArgs(term);
209  
210          // propose
211          return Governor.propose(targets, values, calldatas, description);
212:     }

```
*GitHub*: [181](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOnboarding.sol#L181-L212)

```solidity
File: src/tokens/ERC20Gauges.sol

/// @audit combine into a `struct`: gaugeType,getGaugeWeight
182      function calculateGaugeAllocation(
183          address gauge,
184          uint256 quantity
185      ) external view returns (uint256) {
186          if (_deprecatedGauges.contains(gauge)) return 0;
187  
188          uint256 total = totalTypeWeight[gaugeType[gauge]];
189          if (total == 0) return 0;
190          uint256 weight = getGaugeWeight[gauge];
191  
192          return (quantity * weight) / total;
193:     }

/// @audit combine into a `struct`: gaugeType,getGaugeWeight
/// @audit combine into a `struct`: _userGauges,canExceedMaxGauges,getUserGaugeWeight
230      function _incrementGaugeWeight(
231          address user,
232          address gauge,
233          uint256 weight
234      ) internal virtual {
235          bool added = _userGauges[user].add(gauge); // idempotent add
236          if (added && _userGauges[user].length() > maxGauges) {
237              require(canExceedMaxGauges[user], "ERC20Gauges: exceed max gauges");
238          }
239  
240          getUserGaugeWeight[user][gauge] += weight;
241  
242          getGaugeWeight[gauge] += weight;
243  
244          totalTypeWeight[gaugeType[gauge]] += weight;
245  
246          emit IncrementGaugeWeight(user, gauge, weight);
247:     }

/// @audit combine into a `struct`: _userGauges,getUserGaugeWeight,getUserWeight
314      function _decrementGaugeWeight(
315          address user,
316          address gauge,
317          uint256 weight
318      ) internal virtual {
319          uint256 oldWeight = getUserGaugeWeight[user][gauge];
320  
321          getUserGaugeWeight[user][gauge] = oldWeight - weight;
322          if (oldWeight == weight) {
323              // If removing all weight, remove gauge from user list.
324              require(_userGauges[user].remove(gauge));
325          }
326  
327          getGaugeWeight[gauge] -= weight;
328  
329          getUserWeight[user] -= weight;
330  
331          emit DecrementGaugeWeight(user, gauge, weight);
332:     }

/// @audit combine into a `struct`: gaugeType,getGaugeWeight
395      function _addGauge(
396          uint256 _type,
397          address gauge
398      ) internal returns (uint256 weight) {
399          bool newAdd = _gauges.add(gauge);
400          bool previouslyDeprecated = _deprecatedGauges.remove(gauge);
401          // add and fail loud if zero address or already present and not deprecated
402          require(
403              gauge != address(0) && (newAdd || previouslyDeprecated),
404              "ERC20Gauges: invalid gauge"
405          );
406  
407          if (newAdd) {
408              // save gauge type on first add
409              gaugeType[gauge] = _type;
410          } else {
411              // cannot change gauge type on re-add of a previously deprecated gauge
412              require(gaugeType[gauge] == _type, "ERC20Gauges: invalid type");
413          }
414  
415          // Check if some previous weight exists and re-add to total. Gauge and user weights are preserved.
416          weight = getGaugeWeight[gauge];
417          if (weight != 0) {
418              totalTypeWeight[_type] += weight;
419              totalWeight += weight;
420          }
421  
422          emit AddGauge(gauge, _type);
423:     }

/// @audit combine into a `struct`: gaugeType,getGaugeWeight
425      function _removeGauge(address gauge) internal {
426          // add to deprecated and fail loud if not present
427          require(
428              _gauges.contains(gauge) && _deprecatedGauges.add(gauge),
429              "ERC20Gauges: invalid gauge"
430          );
431  
432          // Remove weight from total but keep the gauge and user weights in storage in case gauge is re-added.
433          uint256 weight = getGaugeWeight[gauge];
434          if (weight != 0) {
435              totalTypeWeight[gaugeType[gauge]] -= weight;
436              totalWeight -= weight;
437          }
438  
439          emit RemoveGauge(gauge);
440:     }

/// @audit combine into a `struct`: _userGauges,getUserGaugeWeight,getUserWeight
500      function _decrementWeightUntilFree(address user, uint256 weight) internal {
501          uint256 userFreeWeight = balanceOf(user) - getUserWeight[user];
502  
503          // early return if already free
504          if (userFreeWeight >= weight) return;
505  
506          // cache totals for batch updates
507          uint256 userFreed;
508          uint256 totalFreed;
509  
510          // Loop through all user gauges, live and deprecated
511          address[] memory gaugeList = _userGauges[user].values();
512  
513          // Free gauges until through entire list or under weight
514          uint256 size = gaugeList.length;
515          for (
516              uint256 i = 0;
517              i < size && (userFreeWeight + userFreed) < weight;
518  
519          ) {
520              address gauge = gaugeList[i];
521              uint256 userGaugeWeight = getUserGaugeWeight[user][gauge];
522              if (userGaugeWeight != 0) {
523                  userFreed += userGaugeWeight;
524                  _decrementGaugeWeight(user, gauge, userGaugeWeight);
525  
526                  // If the gauge is live (not deprecated), include its weight in the total to remove
527                  if (!_deprecatedGauges.contains(gauge)) {
528                      totalTypeWeight[gaugeType[gauge]] -= userGaugeWeight;
529                      totalFreed += userGaugeWeight;
530                  }
531  
532                  unchecked {
533                      ++i;
534                  }
535              }
536          }
537  
538          totalWeight -= totalFreed;
539:     }

```
*GitHub*: [182](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L182-L193), [230](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L230-L247), [230](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L230-L247), [314](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L314-L332), [395](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L395-L423), [425](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L425-L440), [500](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L500-L539)

```solidity
File: src/tokens/ERC20MultiVotes.sol

/// @audit combine into a `struct`: _delegates,_delegatesVotesCount
290      function _delegate(
291          address delegator,
292          address newDelegatee
293      ) internal virtual {
294          uint256 count = delegateCount(delegator);
295  
296          // undefined behavior for delegateCount > 1
297          require(count < 2, "ERC20MultiVotes: delegation error");
298  
299          address oldDelegatee;
300          // if already delegated, undelegate first
301          if (count == 1) {
302              oldDelegatee = _delegates[delegator].at(0);
303              _undelegate(
304                  delegator,
305                  oldDelegatee,
306                  _delegatesVotesCount[delegator][oldDelegatee]
307              );
308          }
309  
310          // redelegate only if newDelegatee is not empty
311          if (newDelegatee != address(0)) {
312              _incrementDelegation(delegator, newDelegatee, freeVotes(delegator));
313          }
314          emit DelegateChanged(delegator, oldDelegatee, newDelegatee);
315:     }

/// @audit combine into a `struct`: _delegates,_delegatesVotesCount,canContractExceedMaxDelegates,userDelegatedVotes
317      function _incrementDelegation(
318          address delegator,
319          address delegatee,
320          uint256 amount
321      ) internal virtual {
322          // Require freeVotes exceed the delegation size
323          uint256 free = freeVotes(delegator);
324          require(
325              delegatee != address(0) && free >= amount,
326              "ERC20MultiVotes: delegation error"
327          );
328  
329          bool newDelegate = _delegates[delegator].add(delegatee); // idempotent add
330          require(
331              !newDelegate ||
332                  delegateCount(delegator) <= maxDelegates ||
333                  canContractExceedMaxDelegates[delegator],
334              "ERC20MultiVotes: delegation error"
335          );
336  
337          _delegatesVotesCount[delegator][delegatee] += amount;
338          userDelegatedVotes[delegator] += amount;
339  
340          emit Delegation(delegator, delegatee, amount);
341          _writeCheckpoint(delegatee, _add, amount);
342:     }

/// @audit combine into a `struct`: _delegates,_delegatesVotesCount,userDelegatedVotes
344      function _undelegate(
345          address delegator,
346          address delegatee,
347          uint256 amount
348      ) internal virtual {
349          uint256 newDelegates = _delegatesVotesCount[delegator][delegatee] -
350              amount;
351  
352          if (newDelegates == 0) {
353              require(_delegates[delegator].remove(delegatee));
354          }
355  
356          _delegatesVotesCount[delegator][delegatee] = newDelegates;
357          userDelegatedVotes[delegator] -= amount;
358  
359          emit Undelegation(delegator, delegatee, amount);
360          _writeCheckpoint(delegatee, _subtract, amount);
361:     }

/// @audit combine into a `struct`: _delegates,_delegatesVotesCount,userDelegatedVotes
427      function _decrementVotesUntilFree(address user, uint256 votes) internal {
428          uint256 userFreeVotes = freeVotes(user);
429  
430          // early return if already free
431          if (userFreeVotes >= votes) return;
432  
433          // cache total for batch updates
434          uint256 totalFreed;
435  
436          // Loop through all delegates
437          address[] memory delegateList = _delegates[user].values();
438  
439          // Free delegates until through entire list or under votes amount
440          uint256 size = delegateList.length;
441          for (
442              uint256 i = 0;
443              i < size && (userFreeVotes + totalFreed) < votes;
444              i++
445          ) {
446              address delegatee = delegateList[i];
447              uint256 delegateVotes = _delegatesVotesCount[user][delegatee];
448              if (delegateVotes != 0) {
449                  totalFreed += delegateVotes;
450  
451                  require(_delegates[user].remove(delegatee)); // Remove from set. Should never fail.
452  
453                  _delegatesVotesCount[user][delegatee] = 0;
454  
455                  _writeCheckpoint(delegatee, _subtract, delegateVotes);
456                  emit Undelegation(user, delegatee, delegateVotes);
457              }
458          }
459  
460          userDelegatedVotes[user] -= totalFreed;
461:     }

```
*GitHub*: [290](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L290-L315), [317](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L317-L342), [344](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L344-L361), [427](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L427-L461)



### [N&#x2011;20] NatSpec: Non-public state variable declarations should use `@dev` tags
i.e. `@dev` [tags](https://docs.soliditylang.org/en/latest/natspec-format.html#tags). Note that since they're non-public, `@notice` is not the right tag to use.

*There are 34 instances of this issue:*

<details>
<summary>see instances</summary>


```solidity
File: src/core/CoreRef.sol

17:      Core private _core;

```
*GitHub*: [17](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRef.sol#L17-L17)

```solidity
File: src/core/CoreRoles.sol

12:      bytes32 internal constant GOVERNOR = keccak256("GOVERNOR_ROLE");

15:      bytes32 internal constant GUARDIAN = keccak256("GUARDIAN_ROLE");

20:      bytes32 internal constant CREDIT_MINTER = keccak256("CREDIT_MINTER_ROLE");

23       bytes32 internal constant RATE_LIMITED_CREDIT_MINTER =
24:          keccak256("RATE_LIMITED_CREDIT_MINTER_ROLE");

27:      bytes32 internal constant GUILD_MINTER = keccak256("GUILD_MINTER_ROLE");

30       bytes32 internal constant RATE_LIMITED_GUILD_MINTER =
31:          keccak256("RATE_LIMITED_GUILD_MINTER_ROLE");

36:      bytes32 internal constant GAUGE_ADD = keccak256("GAUGE_ADD_ROLE");

39:      bytes32 internal constant GAUGE_REMOVE = keccak256("GAUGE_REMOVE_ROLE");

42       bytes32 internal constant GAUGE_PARAMETERS =
43:          keccak256("GAUGE_PARAMETERS_ROLE");

46       bytes32 internal constant GAUGE_PNL_NOTIFIER =
47:          keccak256("GAUGE_PNL_NOTIFIER_ROLE");

50       bytes32 internal constant GUILD_GOVERNANCE_PARAMETERS =
51:          keccak256("GUILD_GOVERNANCE_PARAMETERS_ROLE");

54       bytes32 internal constant GUILD_SURPLUS_BUFFER_WITHDRAW =
55:          keccak256("GUILD_SURPLUS_BUFFER_WITHDRAW_ROLE");

60       bytes32 internal constant CREDIT_GOVERNANCE_PARAMETERS =
61:          keccak256("CREDIT_GOVERNANCE_PARAMETERS_ROLE");

64       bytes32 internal constant CREDIT_REBASE_PARAMETERS =
65:          keccak256("CREDIT_REBASE_PARAMETERS_ROLE");

71:      bytes32 internal constant TIMELOCK_PROPOSER = keccak256("PROPOSER_ROLE");

74:      bytes32 internal constant TIMELOCK_EXECUTOR = keccak256("EXECUTOR_ROLE");

77:      bytes32 internal constant TIMELOCK_CANCELLER = keccak256("CANCELLER_ROLE");

```
*GitHub*: [12](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRoles.sol#L12-L12), [15](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRoles.sol#L15-L15), [20](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRoles.sol#L20-L20), [23](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRoles.sol#L23-L24), [27](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRoles.sol#L27-L27), [30](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRoles.sol#L30-L31), [36](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRoles.sol#L36-L36), [39](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRoles.sol#L39-L39), [42](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRoles.sol#L42-L43), [46](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRoles.sol#L46-L47), [50](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRoles.sol#L50-L51), [54](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRoles.sol#L54-L55), [60](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRoles.sol#L60-L61), [64](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRoles.sol#L64-L65), [71](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRoles.sol#L71-L71), [74](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRoles.sol#L74-L74), [77](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRoles.sol#L77-L77)

```solidity
File: src/governance/GuildGovernor.sol

25:      uint256 private _quorum;

```
*GitHub*: [25](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildGovernor.sol#L25-L25)

```solidity
File: src/governance/GuildVetoGovernor.sol

27:      uint256 private _quorum;

90:      mapping(uint256 => bytes32) private _timelockIds;

```
*GitHub*: [27](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L27-L27), [90](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L90-L90)

```solidity
File: src/governance/ProfitManager.sol

64:      ProfitSharingConfig internal profitSharingConfig;

94:      uint256 internal _minBorrow = 100e18;

```
*GitHub*: [64](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L64-L64), [94](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L94-L94)

```solidity
File: src/loan/LendingTerm.sol

112:     LendingTermReferences internal refs;

151:     LendingTermParams internal params;

```
*GitHub*: [112](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L112-L112), [151](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L151-L151)

```solidity
File: src/loan/SurplusGuildMinter.sol

86:      mapping(address => mapping(address => UserStake)) internal _stakes;

```
*GitHub*: [86](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L86-L86)

```solidity
File: src/tokens/ERC20Gauges.sol

91:      EnumerableSet.AddressSet internal _deprecatedGauges;

```
*GitHub*: [91](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L91-L91)

```solidity
File: src/tokens/ERC20MultiVotes.sol

49:      mapping(address => Checkpoint[]) private _checkpoints;

203      mapping(address => mapping(address => uint256))
204:         private _delegatesVotesCount;

210:     mapping(address => EnumerableSet.AddressSet) private _delegates;

```
*GitHub*: [49](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L49-L49), [203](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L203-L204), [210](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L210-L210)

```solidity
File: src/tokens/ERC20RebaseDistributor.sol

83:      mapping(address => RebasingState) internal rebasingState;

86:      uint256 internal totalRebasingShares;

108      InterpolatedValue internal __rebasingSharePrice =
109          InterpolatedValue({
110              lastTimestamp: SafeCastLib.safeCastTo32(block.timestamp),
111              lastValue: uint224(START_REBASING_SHARE_PRICE), // safe initial value
112              targetTimestamp: SafeCastLib.safeCastTo32(block.timestamp),
113              targetValue: uint224(START_REBASING_SHARE_PRICE) // safe initial value
114:         });

118      InterpolatedValue internal __unmintedRebaseRewards =
119          InterpolatedValue({
120              lastTimestamp: SafeCastLib.safeCastTo32(block.timestamp),
121              lastValue: 0,
122              targetTimestamp: SafeCastLib.safeCastTo32(block.timestamp),
123              targetValue: 0
124:         });

```
*GitHub*: [83](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L83-L83), [86](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L86-L86), [108](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L108-L114), [118](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L118-L124)

</details>





### [N&#x2011;21] NatSpec: State variable declarations should have descriptions
e.g. `@notice` [tags](https://docs.soliditylang.org/en/latest/natspec-format.html#tags)

*There is one instance of this issue:*

```solidity
File: src/tokens/ERC20MultiVotes.sol

467      bytes32 public constant DELEGATION_TYPEHASH =
468:         keccak256("Delegation(address delegatee,uint256 nonce,uint256 expiry)");

```
*GitHub*: [467](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L467-L468)



### [N&#x2011;22] Outdated package versions
Packages should be updated to the latest major version (if not the very latest version) to ensure the uptake of any important fixes.

*There are 25 instances of this issue:*

<details>
<summary>see instances</summary>


```solidity
File: src/governance/GuildGovernor.sol

5:   import {GovernorSettings} from "@openzeppelin/contracts/governance/extensions/GovernorSettings.sol";

6:   import {GovernorTimelockControl} from "@openzeppelin/contracts/governance/extensions/GovernorTimelockControl.sol";

7:   import {GovernorVotes, IERC165} from "@openzeppelin/contracts/governance/extensions/GovernorVotes.sol";

8:   import {GovernorCountingSimple} from "@openzeppelin/contracts/governance/extensions/GovernorCountingSimple.sol";

9:   import {IVotes} from "@openzeppelin/contracts/governance/utils/IVotes.sol";

10:  import {TimelockController} from "@openzeppelin/contracts/governance/TimelockController.sol";

```
*GitHub*: [5](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildGovernor.sol#L5-L5), [6](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildGovernor.sol#L6-L6), [7](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildGovernor.sol#L7-L7), [8](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildGovernor.sol#L8-L8), [9](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildGovernor.sol#L9-L9), [10](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildGovernor.sol#L10-L10)

```solidity
File: src/governance/GuildVetoGovernor.sol

5:   import {GovernorTimelockControl} from "@openzeppelin/contracts/governance/extensions/GovernorTimelockControl.sol";

6:   import {GovernorVotes, IERC165} from "@openzeppelin/contracts/governance/extensions/GovernorVotes.sol";

7:   import {GovernorCountingSimple} from "@openzeppelin/contracts/governance/extensions/GovernorCountingSimple.sol";

8:   import {IVotes} from "@openzeppelin/contracts/governance/utils/IVotes.sol";

9:   import {TimelockController} from "@openzeppelin/contracts/governance/TimelockController.sol";

```
*GitHub*: [5](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L5-L5), [6](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L6-L6), [7](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L7-L7), [8](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L8-L8), [9](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L9-L9)

```solidity
File: src/governance/LendingTermOnboarding.sol

5:   import {IERC20} from "@openzeppelin/contracts/token/ERC20/IERC20.sol";

6:   import {Strings} from "@openzeppelin/contracts/utils/Strings.sol";

7:   import {AccessControl} from "@openzeppelin/contracts/access/AccessControl.sol";

8:   import {Governor, IGovernor} from "@openzeppelin/contracts/governance/Governor.sol";

```
*GitHub*: [5](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOnboarding.sol#L5-L5), [6](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOnboarding.sol#L6-L6), [7](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOnboarding.sol#L7-L7), [8](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOnboarding.sol#L8-L8)

```solidity
File: src/loan/LendingTerm.sol

5:   import {SafeERC20} from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

6:   import {IERC20Permit} from "@openzeppelin/contracts/token/ERC20/extensions/IERC20Permit.sol";

```
*GitHub*: [5](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L5-L5), [6](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L6-L6)

```solidity
File: src/loan/SimplePSM.sol

5:   import {SafeERC20} from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

```
*GitHub*: [5](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SimplePSM.sol#L5-L5)

```solidity
File: src/tokens/CreditToken.sol

5:   import {ERC20Permit} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Permit.sol";

6:   import {ERC20Burnable} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";

```
*GitHub*: [5](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/CreditToken.sol#L5-L5), [6](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/CreditToken.sol#L6-L6)

```solidity
File: src/tokens/ERC20Gauges.sol

5:   import {EnumerableSet} from "@openzeppelin/contracts/utils/structs/EnumerableSet.sol";

```
*GitHub*: [5](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L5-L5)

```solidity
File: src/tokens/ERC20MultiVotes.sol

7:   import {EnumerableSet} from "@openzeppelin/contracts/utils/structs/EnumerableSet.sol";

```
*GitHub*: [7](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L7-L7)

```solidity
File: src/tokens/GuildToken.sol

5:   import {ERC20Permit} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Permit.sol";

6:   import {ERC20Burnable} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";

7:   import {EnumerableSet} from "@openzeppelin/contracts/utils/structs/EnumerableSet.sol";

```
*GitHub*: [5](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L5-L5), [6](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L6-L6), [7](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L7-L7)

</details>





### [N&#x2011;23] Pausable contract never uses `whenNotPosed` modifier
Consider whether the contract really needs to be `Pausable`

*There is one instance of this issue:*

```solidity
File: src/core/CoreRef.sol

/// @audit Pausable
12   abstract contract CoreRef is Pausable {
13:      /// @notice emitted when the reference to core is updated

```
*GitHub*: [12](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRef.sol#L12-L13)



### [N&#x2011;24] Style guide: Non-`external`/`public` variable names should begin with an underscore
According to the Solidity Style Guide, non-`external`/`public` variable names should begin with an [underscore](https://docs.soliditylang.org/en/latest/style-guide.html#underscore-prefix-for-non-external-functions-and-variables)

*There are 17 instances of this issue:*

```solidity
File: src/core/CoreRoles.sol

12:      bytes32 internal constant GOVERNOR = keccak256("GOVERNOR_ROLE");

15:      bytes32 internal constant GUARDIAN = keccak256("GUARDIAN_ROLE");

20:      bytes32 internal constant CREDIT_MINTER = keccak256("CREDIT_MINTER_ROLE");

23       bytes32 internal constant RATE_LIMITED_CREDIT_MINTER =
24:          keccak256("RATE_LIMITED_CREDIT_MINTER_ROLE");

27:      bytes32 internal constant GUILD_MINTER = keccak256("GUILD_MINTER_ROLE");

30       bytes32 internal constant RATE_LIMITED_GUILD_MINTER =
31:          keccak256("RATE_LIMITED_GUILD_MINTER_ROLE");

36:      bytes32 internal constant GAUGE_ADD = keccak256("GAUGE_ADD_ROLE");

39:      bytes32 internal constant GAUGE_REMOVE = keccak256("GAUGE_REMOVE_ROLE");

42       bytes32 internal constant GAUGE_PARAMETERS =
43:          keccak256("GAUGE_PARAMETERS_ROLE");

46       bytes32 internal constant GAUGE_PNL_NOTIFIER =
47:          keccak256("GAUGE_PNL_NOTIFIER_ROLE");

50       bytes32 internal constant GUILD_GOVERNANCE_PARAMETERS =
51:          keccak256("GUILD_GOVERNANCE_PARAMETERS_ROLE");

54       bytes32 internal constant GUILD_SURPLUS_BUFFER_WITHDRAW =
55:          keccak256("GUILD_SURPLUS_BUFFER_WITHDRAW_ROLE");

60       bytes32 internal constant CREDIT_GOVERNANCE_PARAMETERS =
61:          keccak256("CREDIT_GOVERNANCE_PARAMETERS_ROLE");

64       bytes32 internal constant CREDIT_REBASE_PARAMETERS =
65:          keccak256("CREDIT_REBASE_PARAMETERS_ROLE");

71:      bytes32 internal constant TIMELOCK_PROPOSER = keccak256("PROPOSER_ROLE");

74:      bytes32 internal constant TIMELOCK_EXECUTOR = keccak256("EXECUTOR_ROLE");

77:      bytes32 internal constant TIMELOCK_CANCELLER = keccak256("CANCELLER_ROLE");

```
*GitHub*: [12](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRoles.sol#L12-L12), [15](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRoles.sol#L15-L15), [20](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRoles.sol#L20-L20), [23](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRoles.sol#L23-L24), [27](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRoles.sol#L27-L27), [30](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRoles.sol#L30-L31), [36](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRoles.sol#L36-L36), [39](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRoles.sol#L39-L39), [42](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRoles.sol#L42-L43), [46](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRoles.sol#L46-L47), [50](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRoles.sol#L50-L51), [54](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRoles.sol#L54-L55), [60](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRoles.sol#L60-L61), [64](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRoles.sol#L64-L65), [71](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRoles.sol#L71-L71), [74](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRoles.sol#L74-L74), [77](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRoles.sol#L77-L77)



### [N&#x2011;25] Use of `override` is unnecessary
Starting with Solidity version [0.8.8](https://docs.soliditylang.org/en/v0.8.20/contracts.html#function-overriding), using the `override` keyword when the function solely overrides an interface function, and the function doesn't exist in multiple base contracts, is unnecessary.

*There are 49 instances of this issue:*

<details>
<summary>see instances</summary>


```solidity
File: src/governance/GuildGovernor.sol

57       function quorum(
58           uint256 /* blockNumber*/
59:      ) public view override returns (uint256) {

78       function setVotingDelay(
79           uint256 newVotingDelay
80:      ) public override onlyCoreRole(CoreRoles.GOVERNOR) {

85       function setVotingPeriod(
86           uint256 newVotingPeriod
87:      ) public override onlyCoreRole(CoreRoles.GOVERNOR) {

92       function setProposalThreshold(
93           uint256 newProposalThreshold
94:      ) public override onlyCoreRole(CoreRoles.GOVERNOR) {

123      function _cancel(
124          address[] memory targets,
125          uint256[] memory values,
126          bytes[] memory calldatas,
127          bytes32 descriptionHash
128:     ) internal override(Governor, GovernorTimelockControl) returns (uint256) {

132      function _execute(
133          uint256 proposalId,
134          address[] memory targets,
135          uint256[] memory values,
136          bytes[] memory calldatas,
137          bytes32 descriptionHash
138:     ) internal override(Governor, GovernorTimelockControl) {

142      function _executor()
143          internal
144          view
145          override(Governor, GovernorTimelockControl)
146          returns (address)
147:     {

151      function proposalThreshold()
152          public
153          view
154          override(Governor, GovernorSettings)
155          returns (uint256)
156:     {

160      function state(
161          uint256 proposalId
162      )
163          public
164          view
165          override(Governor, GovernorTimelockControl)
166          returns (ProposalState)
167:     {

171      function supportsInterface(
172          bytes4 interfaceId
173:     ) public view override(Governor, GovernorTimelockControl) returns (bool) {

```
*GitHub*: [57](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildGovernor.sol#L57-L59), [78](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildGovernor.sol#L78-L80), [85](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildGovernor.sol#L85-L87), [92](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildGovernor.sol#L92-L94), [123](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildGovernor.sol#L123-L128), [132](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildGovernor.sol#L132-L138), [142](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildGovernor.sol#L142-L147), [151](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildGovernor.sol#L151-L156), [160](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildGovernor.sol#L160-L167), [171](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildGovernor.sol#L171-L173)

```solidity
File: src/governance/GuildTimelockController.sol

30       function hasRole(
31           bytes32 role,
32           address account
33:      ) public view virtual override returns (bool) {

38:      function _setRoleAdmin(bytes32 role, bytes32 adminRole) internal override {}

41:      function _grantRole(bytes32 role, address account) internal override {}

44:      function _revokeRole(bytes32 role, address account) internal override {}

```
*GitHub*: [30](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildTimelockController.sol#L30-L33), [38](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildTimelockController.sol#L38-L38), [41](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildTimelockController.sol#L41-L41), [44](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildTimelockController.sol#L44-L44)

```solidity
File: src/governance/GuildVetoGovernor.sol

53       function quorum(
54           uint256 /* blockNumber*/
55:      ) public view override returns (uint256) {

130      function COUNTING_MODE()
131          public
132          pure
133          virtual
134          override
135          returns (string memory)
136:     {

143      function hasVoted(
144          uint256 proposalId,
145          address account
146:     ) public view virtual override returns (bool) {

172      function _quorumReached(
173          uint256 proposalId
174:     ) internal view virtual override returns (bool) {

186      function _voteSucceeded(
187          uint256 /* proposalId*/
188:     ) internal pure virtual override returns (bool) {

195      function _countVote(
196          uint256 proposalId,
197          address account,
198          uint8 support,
199          uint256 weight,
200          bytes memory // params
201:     ) internal virtual override {

222:     function votingDelay() public pure override returns (uint256) {

230:     function votingPeriod() public pure override returns (uint256) {

246      function state(
247          uint256 proposalId
248:     ) public view override returns (ProposalState) {

304      function propose(
305          address[] memory /* targets*/,
306          uint256[] memory /* values*/,
307          bytes[] memory /* calldatas*/,
308          string memory /* description*/
309:     ) public pure override returns (uint256) {

```
*GitHub*: [53](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L53-L55), [130](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L130-L136), [143](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L143-L146), [172](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L172-L174), [186](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L186-L188), [195](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L195-L201), [222](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L222-L222), [230](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L230-L230), [246](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L246-L248), [304](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L304-L309)

```solidity
File: src/governance/LendingTermOnboarding.sol

171      function propose(
172          address[] memory /* targets*/,
173          uint256[] memory /* values*/,
174          bytes[] memory /* calldatas*/,
175          string memory /* description*/
176:     ) public pure override(IGovernor, Governor) returns (uint256) {

```
*GitHub*: [171](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOnboarding.sol#L171-L176)

```solidity
File: src/tokens/CreditToken.sol

83       function _mint(
84           address account,
85           uint256 amount
86:      ) internal override(ERC20, ERC20RebaseDistributor) {

90       function _burn(
91           address account,
92           uint256 amount
93:      ) internal override(ERC20, ERC20MultiVotes, ERC20RebaseDistributor) {

98       function balanceOf(
99           address account
100:     ) public view override(ERC20, ERC20RebaseDistributor) returns (uint256) {

104      function totalSupply()
105          public
106          view
107          override(ERC20, ERC20RebaseDistributor)
108          returns (uint256)
109:     {

113      function transfer(
114          address to,
115          uint256 amount
116      )
117          public
118          override(ERC20, ERC20MultiVotes, ERC20RebaseDistributor)
119          returns (bool)
120:     {

125      function transferFrom(
126          address from,
127          address to,
128          uint256 amount
129      )
130          public
131          override(ERC20, ERC20MultiVotes, ERC20RebaseDistributor)
132          returns (bool)
133:     {

```
*GitHub*: [83](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/CreditToken.sol#L83-L86), [90](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/CreditToken.sol#L90-L93), [98](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/CreditToken.sol#L98-L100), [104](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/CreditToken.sol#L104-L109), [113](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/CreditToken.sol#L113-L120), [125](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/CreditToken.sol#L125-L133)

```solidity
File: src/tokens/ERC20Gauges.sol

476:     function _burn(address from, uint256 amount) internal virtual override {

481      function transfer(
482          address to,
483          uint256 amount
484:     ) public virtual override returns (bool) {

489      function transferFrom(
490          address from,
491          address to,
492          uint256 amount
493:     ) public virtual override returns (bool) {

```
*GitHub*: [476](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L476-L476), [481](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L481-L484), [489](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L489-L493)

```solidity
File: src/tokens/ERC20MultiVotes.sol

403:     function _burn(address from, uint256 amount) internal virtual override {

408      function transfer(
409          address to,
410          uint256 amount
411:     ) public virtual override returns (bool) {

416      function transferFrom(
417          address from,
418          address to,
419          uint256 amount
420:     ) public virtual override returns (bool) {

```
*GitHub*: [403](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L403-L403), [408](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L408-L411), [416](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L416-L420)

```solidity
File: src/tokens/ERC20RebaseDistributor.sol

426      function balanceOf(
427          address account
428:     ) public view virtual override returns (uint256) {

444:     function totalSupply() public view virtual override returns (uint256) {

461:     function _burn(address account, uint256 amount) internal virtual override {

509:     function _mint(address account, uint256 amount) internal virtual override {

553      function transfer(
554          address to,
555          uint256 amount
556:     ) public virtual override returns (bool) {

646      function transferFrom(
647          address from,
648          address to,
649          uint256 amount
650:     ) public virtual override returns (bool) {

```
*GitHub*: [426](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L426-L428), [444](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L444-L444), [461](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L461-L461), [509](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L509-L509), [553](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L553-L556), [646](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L646-L650)

```solidity
File: src/tokens/GuildToken.sol

182      function _beforeTokenTransfer(
183          address from,
184          address to,
185          uint256 /* amount*/
186:     ) internal view override {

207      function _decrementGaugeWeight(
208          address user,
209          address gauge,
210          uint256 weight
211:     ) internal override {

242      function _incrementGaugeWeight(
243          address user,
244          address gauge,
245          uint256 weight
246:     ) internal override {

279      function _burn(
280          address from,
281          uint256 amount
282:     ) internal virtual override(ERC20, ERC20Gauges, ERC20MultiVotes) {

288      function transfer(
289          address to,
290          uint256 amount
291      )
292          public
293          virtual
294          override(ERC20, ERC20Gauges, ERC20MultiVotes)
295          returns (bool)
296:     {

302      function transferFrom(
303          address from,
304          address to,
305          uint256 amount
306      )
307          public
308          virtual
309          override(ERC20, ERC20Gauges, ERC20MultiVotes)
310          returns (bool)
311:     {

```
*GitHub*: [182](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L182-L186), [207](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L207-L211), [242](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L242-L246), [279](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L279-L282), [288](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L288-L296), [302](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L302-L311)

</details>
