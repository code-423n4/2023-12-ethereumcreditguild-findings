**Note:** I've filtered out the specific instances found by the winning bot

## Summary

### Gas Optimizations

| |Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
| [[G&#x2011;01](#g01-do-while-is-cheaper-than-for-loops-when-the-initial-check-can-be-skipped)] | `do`-`while` is cheaper than `for`-loops when the initial check can be skipped | 6 |  - |
| [[G&#x2011;02](#g02-argument-validation-should-be-at-the-top-of-the-functionblock)] | Argument validation should be at the top of the function/block | 1 |  - |
| [[G&#x2011;03](#g03-assembly-use-scratch-space-for-building-calldata)] | Assembly: Use scratch space for building calldata | 67 |  14740 |
| [[G&#x2011;04](#g04-avoid-transferring-amounts-of-zero-in-order-to-save-gas)] | Avoid transferring amounts of zero in order to save gas | 2 |  200 |
| [[G&#x2011;05](#g05-combine-mappings-referenced-in-the-same-function-by-the-same-key)] | Combine `mapping`s referenced in the same function by the same key | 19 |  798 |
| [[G&#x2011;06](#g06-internal-functions-only-called-once-can-be-inlined-to-save-gas)] | `internal` functions only called once can be inlined to save gas | 10 |  200 |
| [[G&#x2011;07](#g07-unchecked---can-be-used-on-the-division-of-two-uints-in-order-to-save-gas)] | `unchecked {}`  can be used on the division of two `uint`s in order to save gas | 3 |  60 |
| [[G&#x2011;08](#g08-private-functions-only-called-once-can-be-inlined-to-save-gas)] | `private` functions only called once can be inlined to save gas | 1 |  20 |
| [[G&#x2011;09](#g09-assembly-use-scratch-space-when-building-emitted-events-with-two-data-arguments)] | Assembly: Use scratch space when building emitted events with two data arguments | 21 |  315 |
| [[G&#x2011;10](#g10-x--y-costs-more-gas-than-x--x--y-for-basic-typed-state-variables)] | `x += y` costs more gas than `x = x + y` for basic-typed state variables | 1 |  10 |
| [[G&#x2011;11](#g11-consider-using-soladys-token-contracts-to-save-gas)] | Consider using `solady`'s token contracts to save gas | 10 |  - |
| [[G&#x2011;12](#g12-consider-using-soladys-fixedpointmathlib)] | Consider using solady's `FixedPointMathLib` | 42 |  - |
| [[G&#x2011;13](#g13-duplicated-requirerevert-checks-should-be-refactored-to-a-modifier-or-function-to-save-deployment-gas)] | Duplicated `require()`/`revert()` checks should be refactored to a modifier or function to save deployment gas | 16 |  - |
| [[G&#x2011;14](#g14-initializers-can-be-marked-payable)] | Initializers can be marked `payable` | 1 |  - |
| [[G&#x2011;15](#g15-mappings-are-cheaper-to-use-than-storage-arrays)] | Mappings are cheaper to use than storage arrays | 1 |  2100 |
| [[G&#x2011;16](#g16-multiple-addressid-mappings-can-be-combined-into-a-single-mapping-of-an-addressid-to-a-struct)] | Multiple `address`/ID mappings can be combined into a single `mapping` of an `address`/ID to a `struct` | 7 |  - |
| [[G&#x2011;17](#g17-multiple-accesses-of-a-memorycalldata-array-should-use-a-local-variable-cache)] | Multiple accesses of a `memory`/`calldata` array should use a local variable cache | 3 |  - |
| [[G&#x2011;18](#g18-multiple-accesses-of-a-storage-array-should-use-a-local-variable-cache)] | Multiple accesses of a storage array should use a local variable cache | 1 |  42 |
| [[G&#x2011;19](#g19-reduce-deployment-costs-by-tweaking-contracts-metadata)] | Reduce deployment costs by tweaking contracts' metadata | 13 |  - |
| [[G&#x2011;20](#g20-state-variables-only-set-in-the-constructor-should-be-declared-immutable)] | State variables only set in the constructor should be declared `immutable` | 2 |  4194 |
| [[G&#x2011;21](#g21-events-should-be-emitted-outside-of-loops)] | Events should be emitted outside of loops | 12 |  4500 |
| [[G&#x2011;22](#g22-emitting-constants-wastes-gas)] | Emitting constants wastes gas | 6 |  48 |
| [[G&#x2011;23](#g23-stack-variable-is-only-used-once)] | Stack variable is only used once | 71 |  213 |
| [[G&#x2011;24](#g24-splitting-require-statements-that-use--saves-gas)] | Splitting `require()` statements that use `&&` saves gas | 1 |  3 |
| [[G&#x2011;25](#g25-string-literals-passed-to-abiencodeabiencodepacked-should-not-be-split-by-commas)] | String literals passed to `abi.encode()`/`abi.encodePacked()` should not be split by commas | 1 |  - |
| [[G&#x2011;26](#g26-update-openzeppelin-dependency-to-save-gas)] | Update OpenZeppelin dependency to save gas | 39 |  - |
| [[G&#x2011;27](#g27-use-internal-functions-inside-modifiers-to-save-gas)] | Use internal functions inside modifiers to save gas | 1 |  - |
| [[G&#x2011;28](#g28-use-short-circuit-evaluation-to-avoid-external-calls)] | Use short-circuit evaluation to avoid external calls | 1 |  - |
| [[G&#x2011;29](#g29-use-short-circuit-evaluation-to-avoid-reading-state-variables)] | Use short-circuit evaluation to avoid reading state variables | 4 |  - |
| [[G&#x2011;30](#g30-using-private-rather-than-public-saves-gas)] | Using `private` rather than `public`, saves gas | 66 |  - |

Total: 429 instances over 30 issues with **27443 gas** saved

Gas totals are estimates based on data from the Ethereum Yellowpaper. The estimates use the lower bounds of ranges and count two iterations of each `for`-loop. All values above are runtime, not deployment, values; deployment values are listed in the individual issue descriptions. The table above as well as its gas numbers do not include any of the excluded findings.





## Gas Optimizations


### [G&#x2011;01] `do`-`while` is cheaper than `for`-loops when the initial check can be skipped
`for (uint256 i; i < len; ++i){ ... }` -> `do { ...; ++i } while (i < len);`

*There are 6 instances of this issue:*

```solidity
File: src/core/CoreRef.sol

96:          for (uint256 i = 0; i < calls.length; i++) {

```
*GitHub*: [96](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRef.sol#L96-L96)

```solidity
File: src/governance/ProfitManager.sol

443:         for (uint256 i = 0; i < gauges.length; ) {

467:         for (uint256 i = 0; i < gauges.length; ) {

```
*GitHub*: [443](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L443-L443), [467](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L467-L467)

```solidity
File: src/loan/LendingTerm.sol

685:         for (uint256 i = 0; i < loanIds.length; i++) {

```
*GitHub*: [685](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L685-L685)

```solidity
File: src/tokens/ERC20Gauges.sol

280:         for (uint256 i = 0; i < size; ) {

352:         for (uint256 i = 0; i < size; ) {

```
*GitHub*: [280](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L280-L280), [352](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L352-L352)



### [G&#x2011;02] Argument validation should be at the top of the function/block
Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (**2100 gas***) in a function that may ultimately revert in the unhappy case.

*There is one instance of this issue:*

```solidity
File: src/loan/LendingTerm.sol

167:         assert(_core != address(0));

```
*GitHub*: [167](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L167-L167)



### [G&#x2011;03] Assembly: Use scratch space for building calldata
If an external call's calldata can fit into two or fewer words, use the scratch space to build the calldata, rather than allowing Solidity to do a memory expansion.

*There are 67 instances of this issue:*

<details>
<summary>see instances</summary>


```solidity
File: src/core/CoreRef.sol

25:          require(_core.hasRole(role, msg.sender), "UNAUTHORIZED");

```
*GitHub*: [25](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRef.sol#L25-L25)

```solidity
File: src/governance/GuildVetoGovernor.sol

269          uint256 timelockOperationTimestamp = TimelockController(
270              payable(timelock)
271:         ).getTimestamp(queueid);

317          uint256 timelockExecutionTime = TimelockController(payable(timelock))
318:             .getTimestamp(timelockId);

```
*GitHub*: [269](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L269-L271), [317](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L317-L318)

```solidity
File: src/governance/LendingTermOffboarding.sol

99           require(
100              GuildToken(guildToken).isGauge(term),
101              "LendingTermOffboarding: not an active term"
102:         );

126          uint256 userWeight = GuildToken(guildToken).getPastVotes(
127              msg.sender,
128              snapshotBlock
129:         );

159:         GuildToken(guildToken).removeGauge(term);

181          require(
182              GuildToken(guildToken).isDeprecatedGauge(term),
183              "LendingTermOffboarding: re-onboarded"
184:         );

```
*GitHub*: [99](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOffboarding.sol#L99-L102), [126](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOffboarding.sol#L126-L129), [159](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOffboarding.sol#L159-L159), [181](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOffboarding.sol#L181-L184)

```solidity
File: src/governance/LendingTermOnboarding.sol

199:         bool isGauge = GuildToken(guildToken).isGauge(term);

```
*GitHub*: [199](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOnboarding.sol#L199-L199)

```solidity
File: src/governance/ProfitManager.sol

273:         CreditToken(credit).transfer(to, amount);

285:         CreditToken(credit).transfer(to, amount);

305:             GuildToken(guild).notifyGaugeLoss(gauge);

322:                 CreditToken(_credit).burn(loss);

327:                 CreditToken(_credit).burn(_surplusBuffer);

371                  CreditToken(_credit).transfer(
372                      _profitSharingConfig.otherRecipient,
373                      amountForOther
374:                 );

379:                 CreditToken(_credit).distribute(amountForCredit);

388                  uint256 _gaugeWeight = uint256(
389                      GuildToken(guild).getGaugeWeight(gauge)
390:                 );

413          uint256 _userGaugeWeight = uint256(
414              GuildToken(guild).getUserGaugeWeight(user, gauge)
415:         );

434:             CreditToken(credit).transfer(user, creditEarned);

442:         address[] memory gauges = GuildToken(guild).userGauges(user);

464:         gauges = GuildToken(_guild).userGauges(user);

480                  uint256 _userGaugeWeight = uint256(
481                      GuildToken(_guild).getUserGaugeWeight(user, gauge)
482:                 );

```
*GitHub*: [273](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L273-L273), [285](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L285-L285), [305](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L305-L305), [322](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L322-L322), [327](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L327-L327), [371](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L371-L374), [379](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L379-L379), [388](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L388-L390), [413](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L413-L415), [434](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L434-L434), [442](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L442-L442), [464](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L464-L464), [480](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L480-L482)

```solidity
File: src/loan/LendingTerm.sol

274          uint256 gaugeWeight = GuildToken(_guildToken).getGaugeWeight(
275              address(this)
276:         );

278:         uint256 gaugeType = GuildToken(_guildToken).gaugeType(address(this));

279          uint256 totalWeight = GuildToken(_guildToken).totalTypeWeight(
280              gaugeType
281:         );

383          uint256 _debtCeiling = (GuildToken(refs.guildToken)
384              .calculateGaugeAllocation(
385                  address(this),
386                  totalBorrowedCredit + borrowAmount
387:             ) * gaugeWeightTolerance) / 1e18;

418:         RateLimitedMinter(refs.creditMinter).mint(borrower, borrowAmount);

546          CreditToken(refs.creditToken).transfer(
547              refs.profitManager,
548              interestRepaid
549:         );

550          ProfitManager(refs.profitManager).notifyPnL(
551              address(this),
552              int256(interestRepaid)
553:         );

554:         CreditToken(refs.creditToken).burn(principalRepaid);

555:         RateLimitedMinter(refs.creditMinter).replenishBuffer(principalRepaid);

597              CreditToken(refs.creditToken).transfer(
598                  refs.profitManager,
599                  interest
600:             );

603              ProfitManager(refs.profitManager).notifyPnL(
604                  address(this),
605                  int256(interest)
606:             );

610:         CreditToken(refs.creditToken).burn(principal);

611:         RateLimitedMinter(refs.creditMinter).replenishBuffer(principal);

653              GuildToken(refs.guildToken).isDeprecatedGauge(address(this)) ||
654:                 partialRepayDelayPassed(loanId),

715:         ProfitManager(refs.profitManager).notifyPnL(address(this), pnl);

784:             CreditToken(refs.creditToken).burn(principal);

785:             RateLimitedMinter(refs.creditMinter).replenishBuffer(principal);

792                  CreditToken(refs.creditToken).transfer(
793                      refs.profitManager,
794                      interest
795:                 );

797:             ProfitManager(refs.profitManager).notifyPnL(address(this), pnl);

```
*GitHub*: [274](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L274-L276), [278](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L278-L278), [279](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L279-L281), [383](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L383-L387), [418](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L418-L418), [546](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L546-L549), [550](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L550-L553), [554](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L554-L554), [555](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L555-L555), [597](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L597-L600), [603](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L603-L606), [610](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L610-L610), [611](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L611-L611), [653](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L653-L654), [715](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L715-L715), [784](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L784-L784), [785](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L785-L785), [792](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L792-L795), [797](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L797-L797)

```solidity
File: src/loan/SimplePSM.sol

110:         CreditToken(credit).mint(to, amountOut);

121:             !CreditToken(credit).isRebasing(msg.sender),

127:         CreditToken(credit).mint(msg.sender, amountOut);

128:         CreditToken(credit).forceEnterRebase(msg.sender);

140:         CreditToken(credit).burnFrom(msg.sender, amountIn);

```
*GitHub*: [110](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SimplePSM.sol#L110-L110), [121](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SimplePSM.sol#L121-L121), [127](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SimplePSM.sol#L127-L127), [128](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SimplePSM.sol#L128-L128), [140](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SimplePSM.sol#L140-L140)

```solidity
File: src/loan/SurplusGuildMinter.sol

129:         CreditToken(credit).approve(address(profitManager), amount);

130:         ProfitManager(profitManager).donateToTermSurplusBuffer(term, amount);

135:         RateLimitedMinter(rlgm).mint(address(this), guildAmount);

136:         GuildToken(guild).incrementGauge(term, guildAmount);

142              profitIndex: SafeCastLib.safeCastTo160(
143                  ProfitManager(profitManager).userGaugeProfitIndex(
144                      address(this),
145                      term
146                  )
147:             ),

206:         GuildToken(guild).decrementGauge(term, guildAmount);

207:         RateLimitedMinter(rlgm).replenishBuffer(guildAmount);

208:         GuildToken(guild).burn(guildAmount);

228:         lastGaugeLoss = GuildToken(guild).lastGaugeLoss(term);

239:         ProfitManager(profitManager).claimRewards(address(this)); // this will update profit indexes

240          uint256 _profitIndex = ProfitManager(profitManager)
241:             .userGaugeProfitIndex(address(this), term);

259:                 RateLimitedMinter(rlgm).mint(user, guildReward);

263:                 CreditToken(credit).transfer(user, creditReward);

305:             RateLimitedMinter(rlgm).mint(address(this), guildAmount);

306:             GuildToken(guild).incrementGauge(term, guildAmount);

310:             GuildToken(guild).decrementGauge(term, guildAmount);

311:             RateLimitedMinter(rlgm).replenishBuffer(guildAmount);

312:             GuildToken(guild).burn(guildAmount);

```
*GitHub*: [129](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L129-L129), [130](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L130-L130), [135](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L135-L135), [136](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L136-L136), [142](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L142-L147), [206](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L206-L206), [207](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L207-L207), [208](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L208-L208), [228](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L228-L228), [239](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L239-L239), [240](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L240-L241), [259](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L259-L259), [263](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L263-L263), [305](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L305-L305), [306](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L306-L306), [310](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L310-L310), [311](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L311-L311), [312](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L312-L312)

```solidity
File: src/rate-limits/RateLimitedMinter.sol

54:          IERC20Mintable(token).mint(to, amount); /// interactions

```
*GitHub*: [54](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/rate-limits/RateLimitedMinter.sol#L54-L54)

```solidity
File: src/tokens/GuildToken.sol

220:         ProfitManager(profitManager).claimGaugeRewards(user, gauge);

226:             uint256 debtCeilingAfterDecrement = LendingTerm(gauge).debtCeiling(-int256(weight));

258:         ProfitManager(profitManager).claimGaugeRewards(user, gauge);

```
*GitHub*: [220](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L220-L220), [226](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L226-L226), [258](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L258-L258)

</details>





### [G&#x2011;04] Avoid transferring amounts of zero in order to save gas
Skipping the external call when nothing will be transferred, will save at least **100 gas**

*There are 2 instances of this issue:*

```solidity
File: src/loan/LendingTerm.sol

539          CreditToken(refs.creditToken).transferFrom(
540              repayer,
541              address(this),
542              debtToRepay
543:         );

```
*GitHub*: [539](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L539-L543)

```solidity
File: src/loan/SurplusGuildMinter.sol

128:         CreditToken(credit).transferFrom(msg.sender, address(this), amount);

```
*GitHub*: [128](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L128-L128)



### [G&#x2011;05] Combine `mapping`s referenced in the same function by the same key
Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot (i.e. runtime gas savings). Even if the values can't be packed, if both fields are accessed in the same function (as is the case for these instances), combining them can save **~42 gas per access** due to [not having to recalculate the key's keccak256 hash](https://gist.github.com/IllIllI000/639032d73e35d7e968ff58d8784bc445) (Gkeccak256 - 30 gas) and that calculation's associated stack operations.

*There are 19 instances of this issue:*

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
File: src/governance/ProfitManager.sol

/// @audit combine into a `struct`: gaugeProfitIndex,termSurplusBuffer
292      function notifyPnL(
293          address gauge,
294          int256 amount
295      ) external onlyCoreRole(CoreRoles.GAUGE_PNL_NOTIFIER) {
296          uint256 _surplusBuffer = surplusBuffer;
297          uint256 _termSurplusBuffer = termSurplusBuffer[gauge];
298          address _credit = credit;
299  
300          // handling loss
301          if (amount < 0) {
302              uint256 loss = uint256(-amount);
303  
304              // save gauge loss
305              GuildToken(guild).notifyGaugeLoss(gauge);
306  
307              // deplete the term surplus buffer, if any, and
308              // donate its content to the general surplus buffer
309              if (_termSurplusBuffer != 0) {
310                  termSurplusBuffer[gauge] = 0;
311                  emit TermSurplusBufferUpdate(block.timestamp, gauge, 0);
312                  _surplusBuffer += _termSurplusBuffer;
313              }
314  
315              if (loss < _surplusBuffer) {
316                  // deplete the surplus buffer
317                  surplusBuffer = _surplusBuffer - loss;
318                  emit SurplusBufferUpdate(
319                      block.timestamp,
320                      _surplusBuffer - loss
321                  );
322                  CreditToken(_credit).burn(loss);
323              } else {
324                  // empty the surplus buffer
325                  loss -= _surplusBuffer;
326                  surplusBuffer = 0;
327                  CreditToken(_credit).burn(_surplusBuffer);
328                  emit SurplusBufferUpdate(block.timestamp, 0);
329  
330                  // update the CREDIT multiplier
331                  uint256 creditTotalSupply = CreditToken(_credit).totalSupply();
332                  uint256 newCreditMultiplier = (creditMultiplier *
333                      (creditTotalSupply - loss)) / creditTotalSupply;
334                  creditMultiplier = newCreditMultiplier;
335                  emit CreditMultiplierUpdate(
336                      block.timestamp,
337                      newCreditMultiplier
338                  );
339              }
340          }
341          // handling profit
342          else if (amount > 0) {
343              ProfitSharingConfig
344                  memory _profitSharingConfig = profitSharingConfig;
345  
346              uint256 amountForSurplusBuffer = (uint256(amount) *
347                  uint256(_profitSharingConfig.surplusBufferSplit)) / 1e9;
348  
349              uint256 amountForGuild = (uint256(amount) *
350                  uint256(_profitSharingConfig.guildSplit)) / 1e9;
351  
352              uint256 amountForOther = (uint256(amount) *
353                  uint256(_profitSharingConfig.otherSplit)) / 1e9;
354  
355              uint256 amountForCredit = uint256(amount) -
356                  amountForSurplusBuffer -
357                  amountForGuild -
358                  amountForOther;
359  
360              // distribute to surplus buffer
361              if (amountForSurplusBuffer != 0) {
362                  surplusBuffer = _surplusBuffer + amountForSurplusBuffer;
363                  emit SurplusBufferUpdate(
364                      block.timestamp,
365                      _surplusBuffer + amountForSurplusBuffer
366                  );
367              }
368  
369              // distribute to other
370              if (amountForOther != 0) {
371                  CreditToken(_credit).transfer(
372                      _profitSharingConfig.otherRecipient,
373                      amountForOther
374                  );
375              }
376  
377              // distribute to lenders
378              if (amountForCredit != 0) {
379                  CreditToken(_credit).distribute(amountForCredit);
380              }
381  
382              // distribute to the guild
383              if (amountForGuild != 0) {
384                  // update the gauge profit index
385                  // if the gauge has 0 weight, does not update the profit index, this is unnecessary
386                  // because the profit index is used to reattribute profit to users voting for the gauge,
387                  // and if the weigth is 0, there are no users voting for the gauge.
388                  uint256 _gaugeWeight = uint256(
389                      GuildToken(guild).getGaugeWeight(gauge)
390                  );
391                  if (_gaugeWeight != 0) {
392                      uint256 _gaugeProfitIndex = gaugeProfitIndex[gauge];
393                      if (_gaugeProfitIndex == 0) {
394                          _gaugeProfitIndex = 1e18;
395                      }
396                      gaugeProfitIndex[gauge] =
397                          _gaugeProfitIndex +
398                          (amountForGuild * 1e18) /
399                          _gaugeWeight;
400                  }
401              }
402          }
403  
404          emit GaugePnL(gauge, block.timestamp, amount);
405:     }

```
*GitHub*: [292](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L292-L405)

```solidity
File: src/loan/LendingTerm.sol

/// @audit combine into a `struct`: lastPartialRepay,loans
237      function partialRepayDelayPassed(
238          bytes32 loanId
239      ) public view returns (bool) {
240          // if no periodic partial repays are expected, always return false
241          if (params.maxDelayBetweenPartialRepay == 0) return false;
242  
243          // if loan doesn't exist, return false
244          if (loans[loanId].borrowTime == 0) return false;
245  
246          // if loan is closed, return false
247          if (loans[loanId].closeTime != 0) return false;
248  
249          // return true if delay is passed
250          return
251              lastPartialRepay[loanId] <
252              block.timestamp - params.maxDelayBetweenPartialRepay;
253:     }

/// @audit combine into a `struct`: lastPartialRepay,loans
339      function _borrow(
340          address borrower,
341          uint256 borrowAmount,
342          uint256 collateralAmount
343      ) internal returns (bytes32 loanId) {
344          require(borrowAmount != 0, "LendingTerm: cannot borrow 0");
345          require(collateralAmount != 0, "LendingTerm: cannot stake 0");
346  
347          loanId = keccak256(
348              abi.encode(borrower, address(this), block.timestamp)
349          );
350  
351          // check that the loan doesn't already exist
352          require(loans[loanId].borrowTime == 0, "LendingTerm: loan exists");
353  
354          // check that enough collateral is provided
355          uint256 creditMultiplier = ProfitManager(refs.profitManager)
356              .creditMultiplier();
357          uint256 maxBorrow = (collateralAmount *
358              params.maxDebtPerCollateralToken) / creditMultiplier;
359          require(
360              borrowAmount <= maxBorrow,
361              "LendingTerm: not enough collateral"
362          );
363  
364          // check that enough CREDIT is borrowed
365          require(
366              borrowAmount >= ProfitManager(refs.profitManager).minBorrow(),
367              "LendingTerm: borrow amount too low"
368          );
369  
370          // check the hardcap
371          uint256 _issuance = issuance;
372          uint256 _postBorrowIssuance = _issuance + borrowAmount;
373          require(
374              _postBorrowIssuance <= params.hardCap,
375              "LendingTerm: hardcap reached"
376          );
377  
378          // check the debt ceiling
379          uint256 totalBorrowedCredit = ProfitManager(refs.profitManager)
380              .totalBorrowedCredit();
381          uint256 gaugeWeightTolerance = ProfitManager(refs.profitManager)
382              .gaugeWeightTolerance();
383          uint256 _debtCeiling = (GuildToken(refs.guildToken)
384              .calculateGaugeAllocation(
385                  address(this),
386                  totalBorrowedCredit + borrowAmount
387              ) * gaugeWeightTolerance) / 1e18;
388          if (totalBorrowedCredit == 0) {
389              // if the lending term is deprecated, `calculateGaugeAllocation` will return 0, and the borrow
390              // should revert because the debt ceiling is reached (no borrows should be allowed anymore).
391              // first borrow in the system does not check proportions of issuance, just that the term is not deprecated.
392              require(_debtCeiling != 0, "LendingTerm: debt ceiling reached");
393          } else {
394              require(
395                  _postBorrowIssuance <= _debtCeiling,
396                  "LendingTerm: debt ceiling reached"
397              );
398          }
399  
400          // save loan in state
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
411          });
412          issuance = _postBorrowIssuance;
413          if (params.maxDelayBetweenPartialRepay != 0) {
414              lastPartialRepay[loanId] = block.timestamp;
415          }
416  
417          // mint debt to the borrower
418          RateLimitedMinter(refs.creditMinter).mint(borrower, borrowAmount);
419  
420          // pull the collateral from the borrower
421          IERC20(params.collateralToken).safeTransferFrom(
422              borrower,
423              address(this),
424              collateralAmount
425          );
426  
427          // emit event
428          emit LoanOpen(
429              block.timestamp,
430              loanId,
431              borrower,
432              collateralAmount,
433              borrowAmount
434          );
435:     }

/// @audit combine into a `struct`: lastPartialRepay,loans
490      function _partialRepay(
491          address repayer,
492          bytes32 loanId,
493          uint256 debtToRepay
494      ) internal {
495          Loan storage loan = loans[loanId];
496  
497          // check the loan is open
498          uint256 borrowTime = loan.borrowTime;
499          require(borrowTime != 0, "LendingTerm: loan not found");
500          require(
501              borrowTime < block.timestamp,
502              "LendingTerm: loan opened in same block"
503          );
504          require(loan.closeTime == 0, "LendingTerm: loan closed");
505          require(loan.callTime == 0, "LendingTerm: loan called");
506  
507          // compute partial repayment
508          uint256 loanDebt = getLoanDebt(loanId);
509          require(debtToRepay < loanDebt, "LendingTerm: full repayment");
510          uint256 percentRepaid = (debtToRepay * 1e18) / loanDebt; // [0, 1e18[
511          uint256 borrowAmount = loan.borrowAmount;
512          uint256 creditMultiplier = ProfitManager(refs.profitManager)
513              .creditMultiplier();
514          uint256 principal = (borrowAmount * loan.borrowCreditMultiplier) /
515              creditMultiplier;
516          uint256 principalRepaid = (principal * percentRepaid) / 1e18;
517          uint256 interestRepaid = debtToRepay - principalRepaid;
518          uint256 issuanceDecrease = (borrowAmount * percentRepaid) / 1e18;
519          require(
520              principalRepaid != 0 && interestRepaid != 0,
521              "LendingTerm: repay too small"
522          );
523          require(
524              debtToRepay >= (loanDebt * params.minPartialRepayPercent) / 1e18,
525              "LendingTerm: repay below min"
526          );
527          require(
528              borrowAmount - issuanceDecrease >
529                  ProfitManager(refs.profitManager).minBorrow(),
530              "LendingTerm: below min borrow"
531          );
532  
533          // update loan in state
534          loans[loanId].borrowAmount -= issuanceDecrease;
535          lastPartialRepay[loanId] = block.timestamp;
536          issuance -= issuanceDecrease;
537  
538          // pull the debt from the borrower
539          CreditToken(refs.creditToken).transferFrom(
540              repayer,
541              address(this),
542              debtToRepay
543          );
544  
545          // forward profit portion to the ProfitManager, burn the rest
546          CreditToken(refs.creditToken).transfer(
547              refs.profitManager,
548              interestRepaid
549          );
550          ProfitManager(refs.profitManager).notifyPnL(
551              address(this),
552              int256(interestRepaid)
553          );
554          CreditToken(refs.creditToken).burn(principalRepaid);
555          RateLimitedMinter(refs.creditMinter).replenishBuffer(principalRepaid);
556  
557          // emit event
558          emit LoanPartialRepay(block.timestamp, loanId, repayer, debtToRepay);
559:     }

```
*GitHub*: [237](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L237-L253), [339](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L339-L435), [490](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L490-L559)

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

```solidity
File: src/tokens/GuildToken.sol

/// @audit combine into a `struct`: gaugeType,lastGaugeLoss,lastGaugeLossApplied
133      function applyGaugeLoss(address gauge, address who) external {
134          // check preconditions
135          uint256 _lastGaugeLoss = lastGaugeLoss[gauge];
136          uint256 _lastGaugeLossApplied = lastGaugeLossApplied[gauge][who];
137          require(
138              _lastGaugeLoss != 0 && _lastGaugeLossApplied < _lastGaugeLoss,
139              "GuildToken: no loss to apply"
140          );
141  
142          // read user weight allocated to the lossy gauge
143          uint256 _userGaugeWeight = getUserGaugeWeight[who][gauge];
144  
145          // remove gauge weight allocation
146          lastGaugeLossApplied[gauge][who] = block.timestamp;
147          _decrementGaugeWeight(who, gauge, _userGaugeWeight);
148          if (!_deprecatedGauges.contains(gauge)) {
149              totalTypeWeight[gaugeType[gauge]] -= _userGaugeWeight;
150              totalWeight -= _userGaugeWeight;
151          }
152  
153          // apply loss
154          _burn(who, uint256(_userGaugeWeight));
155          emit GaugeLossApply(
156              gauge,
157              who,
158              uint256(_userGaugeWeight),
159              block.timestamp
160          );
161:     }

/// @audit combine into a `struct`: lastGaugeLoss,lastGaugeLossApplied
207      function _decrementGaugeWeight(
208          address user,
209          address gauge,
210          uint256 weight
211      ) internal override {
212          uint256 _lastGaugeLoss = lastGaugeLoss[gauge];
213          uint256 _lastGaugeLossApplied = lastGaugeLossApplied[gauge][user];
214          require(
215              _lastGaugeLossApplied >= _lastGaugeLoss,
216              "GuildToken: pending loss"
217          );
218  
219          // update the user profit index and claim rewards
220          ProfitManager(profitManager).claimGaugeRewards(user, gauge);
221  
222          // check if gauge is currently using its allocated debt ceiling.
223          // To decrement gauge weight, guild holders might have to call loans if the debt ceiling is used.
224          uint256 issuance = LendingTerm(gauge).issuance();
225          if (issuance != 0) {
226              uint256 debtCeilingAfterDecrement = LendingTerm(gauge).debtCeiling(-int256(weight));
227              require(
228                  issuance <= debtCeilingAfterDecrement,
229                  "GuildToken: debt ceiling used"
230              );
231          }
232  
233          super._decrementGaugeWeight(user, gauge, weight);
234:     }

/// @audit combine into a `struct`: lastGaugeLoss,lastGaugeLossApplied
242      function _incrementGaugeWeight(
243          address user,
244          address gauge,
245          uint256 weight
246      ) internal override {
247          uint256 _lastGaugeLoss = lastGaugeLoss[gauge];
248          uint256 _lastGaugeLossApplied = lastGaugeLossApplied[gauge][user];
249          if (getUserGaugeWeight[user][gauge] == 0) {
250              lastGaugeLossApplied[gauge][user] = block.timestamp;
251          } else {
252              require(
253                  _lastGaugeLossApplied >= _lastGaugeLoss,
254                  "GuildToken: pending loss"
255              );
256          }
257  
258          ProfitManager(profitManager).claimGaugeRewards(user, gauge);
259  
260          super._incrementGaugeWeight(user, gauge, weight);
261:     }

```
*GitHub*: [133](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L133-L161), [207](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L207-L234), [242](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L242-L261)



### [G&#x2011;06] `internal` functions only called once can be inlined to save gas
Not inlining costs **20 to 40 gas** because of two extra `JUMP` instructions and additional stack operations needed for function calls. The inliner can do it only for 'simple' cases:
> Now to get back to the point why we require the routine to be simple: As soon as you do more complicated things like for example branching, calling external contracts, the Common Subexpression Eliminator cannot re-construct the code anymore or does not do full symbolic evaluation of the expressions. 

https://soliditylang.org/blog/2021/03/02/saving-gas-with-simple-inliner/

Therefore, the instances below contain branching or use op-codes with side-effects

*There are 10 instances of this issue:*

```solidity
File: src/loan/LendingTerm.sol

339      function _borrow(
340          address borrower,
341          uint256 borrowAmount,
342          uint256 collateralAmount
343:     ) internal returns (bytes32 loanId) {

448      function _addCollateral(
449          address borrower,
450          bytes32 loanId,
451          uint256 collateralToAdd
452:     ) internal {

490      function _partialRepay(
491          address repayer,
492          bytes32 loanId,
493          uint256 debtToRepay
494:     ) internal {

567:     function _repay(address repayer, bytes32 loanId) internal {

```
*GitHub*: [339](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L339-L343), [448](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L448-L452), [490](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L490-L494), [567](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L567-L567)

```solidity
File: src/tokens/ERC20Gauges.sol

395      function _addGauge(
396          uint256 _type,
397          address gauge
398:     ) internal returns (uint256 weight) {

425:     function _removeGauge(address gauge) internal {

444:     function _setMaxGauges(uint256 newMax) internal {

452      function _setCanExceedMaxGauges(
453          address account,
454          bool canExceedMax
455:     ) internal {

```
*GitHub*: [395](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L395-L398), [425](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L425-L425), [444](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L444-L444), [452](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L452-L455)

```solidity
File: src/utils/RateLimitedV2.sol

93:      function _depleteBuffer(uint256 amount) internal {

111:     function _replenishBuffer(uint256 amount) internal {

```
*GitHub*: [93](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/utils/RateLimitedV2.sol#L93-L93), [111](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/utils/RateLimitedV2.sol#L111-L111)



### [G&#x2011;07] `unchecked {}`  can be used on the division of two `uint`s in order to save gas
The division cannot overflow, since both the numerator and the denominator are non-negative

*There are 3 instances of this issue:*

```solidity
File: src/governance/ProfitManager.sol

483:                 creditEarned[i] = (_userGaugeWeight * deltaIndex) / 1e18;

```
*GitHub*: [483](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L483-L483)

```solidity
File: src/tokens/ERC20RebaseDistributor.sol

213              uint256 percentChange = (sharesAfter * START_REBASING_SHARE_PRICE) /
214:                 sharesBefore;

216                  (delta * START_REBASING_SHARE_PRICE) /
217:                 percentChange;

```
*GitHub*: [213](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L213-L214), [216](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L216-L217)



### [G&#x2011;08] `private` functions only called once can be inlined to save gas
Not inlining costs **20 to 40 gas** because of two extra `JUMP` instructions and additional stack operations needed for function calls. The inliner can do it only for 'simple' cases:
> Now to get back to the point why we require the routine to be simple: As soon as you do more complicated things like for example branching, calling external contracts, the Common Subexpression Eliminator cannot re-construct the code anymore or does not do full symbolic evaluation of the expressions. 

https://soliditylang.org/blog/2021/03/02/saving-gas-with-simple-inliner/

Therefore, the instances below contain branching or use op-codes with side-effects

*There is one instance of this issue:*

```solidity
File: src/tokens/ERC20MultiVotes.sol

103      function _checkpointsLookup(
104          Checkpoint[] storage ckpts,
105          uint256 blockNumber
106:     ) private view returns (uint256) {

```
*GitHub*: [103](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L103-L106)



### [G&#x2011;09] Assembly: Use scratch space when building emitted events with two data arguments
Using the [scratch space](https://gist.github.com/IllIllI000/87c4f03139fa03780fa548b8e4b02b5b) for more than one, but at most two words worth of data (non-indexed arguments) will save gas over needing Solidity's abi memory expansion used for emitting normally.

*There are 21 instances of this issue:*

<details>
<summary>see instances</summary>


```solidity
File: src/governance/GuildGovernor.sol

69:          emit QuorumUpdated(_quorum, newQuorum);

```
*GitHub*: [69](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildGovernor.sol#L69-L69)

```solidity
File: src/governance/GuildVetoGovernor.sol

72:          emit QuorumUpdated(_quorum, newQuorum);

100:         emit TimelockChange(timelock, newTimelock);

```
*GitHub*: [72](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L72-L72), [100](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L100-L100)

```solidity
File: src/governance/LendingTermOffboarding.sol

81:          emit QuorumUpdated(quorum, _quorum);

106          emit OffboardSupport(
107              block.timestamp,
108              term,
109              block.number,
110              address(0),
111              1
112:         );

141          emit OffboardSupport(
142              block.timestamp,
143              term,
144              snapshotBlock,
145              msg.sender,
146              userWeight
147:         );

```
*GitHub*: [81](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOffboarding.sol#L81-L81), [106](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOffboarding.sol#L106-L112), [141](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOffboarding.sol#L141-L147)

```solidity
File: src/loan/LendingTerm.sol

428          emit LoanOpen(
429              block.timestamp,
430              loanId,
431              borrower,
432              collateralAmount,
433              borrowAmount
434:         );

```
*GitHub*: [428](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L428-L434)

```solidity
File: src/loan/SimplePSM.sol

111:         emit Mint(block.timestamp, to, amountIn, amountOut);

129:         emit Mint(block.timestamp, msg.sender, amountIn, amountOut);

143:         emit Redeem(block.timestamp, to, amountIn, amountOut);

```
*GitHub*: [111](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SimplePSM.sol#L111-L111), [129](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SimplePSM.sol#L129-L129), [143](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SimplePSM.sol#L143-L143)

```solidity
File: src/tokens/ERC20Gauges.sol

448:         emit MaxGaugesUpdate(oldMax, newMax);

```
*GitHub*: [448](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L448-L448)

```solidity
File: src/tokens/ERC20MultiVotes.sol

151:         emit MaxDelegatesUpdate(oldMax, newMax);

384:         emit DelegateVotesChanged(delegatee, oldWeight, newWeight);

```
*GitHub*: [151](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L151-L151), [384](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L384-L384)

```solidity
File: src/tokens/ERC20RebaseDistributor.sol

353          emit RebaseDistribution(
354              msg.sender,
355              block.timestamp,
356              amount,
357              _rebasingSupply
358:         );

```
*GitHub*: [353](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L353-L358)

```solidity
File: src/tokens/GuildToken.sol

155          emit GaugeLossApply(
156              gauge,
157              who,
158              uint256(_userGaugeWeight),
159              block.timestamp
160:         );

177:         emit TransfersEnabled(block.number, block.timestamp);

199:         emit ProfitManagerUpdated(block.timestamp, _newProfitManager);

```
*GitHub*: [155](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L155-L160), [177](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L177-L177), [199](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L199-L199)

```solidity
File: src/utils/RateLimitedV2.sol

106:         emit BufferUsed(amount, bufferStored);

134:         emit BufferReplenished(amount, bufferStored);

142          emit RateLimitPerSecondUpdate(
143              oldRateLimitPerSecond,
144              newRateLimitPerSecond
145:         );

155:         emit BufferCapUpdate(oldBufferCap, newBufferCap);

```
*GitHub*: [106](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/utils/RateLimitedV2.sol#L106-L106), [134](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/utils/RateLimitedV2.sol#L134-L134), [142](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/utils/RateLimitedV2.sol#L142-L145), [155](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/utils/RateLimitedV2.sol#L155-L155)

</details>





### [G&#x2011;10] `x += y` costs more gas than `x = x + y` for basic-typed state variables
Using the addition operator instead of plus-equals saves **[10 gas](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)** because of extra `PUSH`es and `POP`s associated with the manipulation of the state variable when using `+=` for basic-typed state variables

*There is one instance of this issue:*

```solidity
File: src/governance/GuildVetoGovernor.sol

211:             proposalvote.againstVotes += weight;

```
*GitHub*: [211](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L211-L211)



### [G&#x2011;11] Consider using `solady`'s token contracts to save gas
They're written using heavily-optimized assembly, and will your users a lot of gas

*There are 10 instances of this issue:*

```solidity
File: src/tokens/CreditToken.sol

/// @audit ERC20Burnable
/// @audit ERC20
/// @audit ERC20Permit
18   contract CreditToken is
19       CoreRef,
20       ERC20Burnable,
21       ERC20MultiVotes,
22       ERC20RebaseDistributor
23:  {

```
*GitHub*: [18](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/CreditToken.sol#L18-L23), [18](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/CreditToken.sol#L18-L23), [18](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/CreditToken.sol#L18-L23)

```solidity
File: src/tokens/ERC20Gauges.sol

/// @audit ERC20
58:  abstract contract ERC20Gauges is ERC20 {

```
*GitHub*: [58](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L58-L58)

```solidity
File: src/tokens/ERC20MultiVotes.sol

/// @audit ERC20
/// @audit ERC20Permit
35:  abstract contract ERC20MultiVotes is ERC20Permit {

```
*GitHub*: [35](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L35-L35), [35](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L35-L35)

```solidity
File: src/tokens/ERC20RebaseDistributor.sol

/// @audit ERC20
42   abstract contract ERC20RebaseDistributor is ERC20 {
43       /*///////////////////////////////////////////////////////////////
44                               EVENTS
45       ///////////////////////////////////////////////////////////////*/
46   
47:      /// @notice Emitted when an `account` enters rebasing.

```
*GitHub*: [42](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L42-L47)

```solidity
File: src/tokens/GuildToken.sol

/// @audit ERC20Burnable
/// @audit ERC20Permit
/// @audit ERC20
37:  contract GuildToken is CoreRef, ERC20Burnable, ERC20Gauges, ERC20MultiVotes {

```
*GitHub*: [37](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L37-L37), [37](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L37-L37), [37](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L37-L37)



### [G&#x2011;12] Consider using solady's `FixedPointMathLib`
Saves gas, and works to avoid unnecessary [overflows](https://github.com/Vectorized/solady/blob/6cce088e69d6e46671f2f622318102bd5db77a65/src/utils/FixedPointMathLib.sol#L267).

*There are 42 instances of this issue:*

<details>
<summary>see instances</summary>


```solidity
File: src/governance/ProfitManager.sol

152:         return (_minBorrow * 1e18) / creditMultiplier;

332                  uint256 newCreditMultiplier = (creditMultiplier *
333:                     (creditTotalSupply - loss)) / creditTotalSupply;

346              uint256 amountForSurplusBuffer = (uint256(amount) *
347:                 uint256(_profitSharingConfig.surplusBufferSplit)) / 1e9;

349              uint256 amountForGuild = (uint256(amount) *
350:                 uint256(_profitSharingConfig.guildSplit)) / 1e9;

352              uint256 amountForOther = (uint256(amount) *
353:                 uint256(_profitSharingConfig.otherSplit)) / 1e9;

398                          (amountForGuild * 1e18) /
399:                         _gaugeWeight;

429:             creditEarned = (_userGaugeWeight * deltaIndex) / 1e18;

483:                 creditEarned[i] = (_userGaugeWeight * deltaIndex) / 1e18;

```
*GitHub*: [152](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L152-L152), [332](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L332-L333), [346](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L346-L347), [349](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L349-L350), [352](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L352-L353), [398](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L398-L399), [429](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L429-L429), [483](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L483-L483)

```solidity
File: src/loan/AuctionHouse.sol

141:             collateralReceived = (_collateralAmount * elapsed) / midPoint;

152:             creditAsked = _callDebt - (_callDebt * elapsed) / PHASE_2_DURATION;

```
*GitHub*: [141](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/AuctionHouse.sol#L141-L141), [152](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/AuctionHouse.sol#L152-L152)

```solidity
File: src/loan/LendingTerm.sol

218          uint256 interest = (borrowAmount *
219              params.interestRate *
220              (block.timestamp - borrowTime)) /
221:             YEAR /

226:             loanDebt += (borrowAmount * _openingFee) / 1e18;

230:         loanDebt = (loanDebt * loan.borrowCreditMultiplier) / creditMultiplier;

305          uint256 toleratedGaugeWeight = (gaugeWeight * gaugeWeightTolerance) /
306:             1e18;

307          uint256 debtCeilingBefore = (totalBorrowedCredit *
308:             toleratedGaugeWeight) / totalWeight;

320          uint256 maxBorrow = (remainingDebtCeiling * totalWeight) /
321:             otherGaugesWeight;

357          uint256 maxBorrow = (collateralAmount *
358:             params.maxDebtPerCollateralToken) / creditMultiplier;

383          uint256 _debtCeiling = (GuildToken(refs.guildToken)
384              .calculateGaugeAllocation(
385                  address(this),
386                  totalBorrowedCredit + borrowAmount
387:             ) * gaugeWeightTolerance) / 1e18;

510:         uint256 percentRepaid = (debtToRepay * 1e18) / loanDebt; // [0, 1e18[

514          uint256 principal = (borrowAmount * loan.borrowCreditMultiplier) /
515:             creditMultiplier;

516:         uint256 principalRepaid = (principal * percentRepaid) / 1e18;

518:         uint256 issuanceDecrease = (borrowAmount * percentRepaid) / 1e18;

524:             debtToRepay >= (loanDebt * params.minPartialRepayPercent) / 1e18,

585          uint256 principal = (borrowAmount * loan.borrowCreditMultiplier) /
586:             creditMultiplier;

712          uint256 principal = (borrowAmount *
713:             loans[loanId].borrowCreditMultiplier) / creditMultiplier;

754          uint256 principal = (borrowAmount *
755:             loans[loanId].borrowCreditMultiplier) / creditMultiplier;

```
*GitHub*: [218](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L218-L221), [226](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L226-L226), [230](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L230-L230), [305](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L305-L306), [307](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L307-L308), [320](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L320-L321), [357](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L357-L358), [383](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L383-L387), [510](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L510-L510), [514](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L514-L515), [516](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L516-L516), [518](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L518-L518), [524](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L524-L524), [585](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L585-L586), [712](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L712-L713), [754](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L754-L755)

```solidity
File: src/loan/SimplePSM.sol

83:          return (amountIn * decimalCorrection * 1e18) / creditMultiplier;

92:          return (amountIn * creditMultiplier) / 1e18 / decimalCorrection;

```
*GitHub*: [83](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SimplePSM.sol#L83-L83), [92](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SimplePSM.sol#L92-L92)

```solidity
File: src/loan/SurplusGuildMinter.sol

134:         uint256 guildAmount = (_mintRatio * amount) / 1e18;

175          uint256 userMintRatio = (uint256(userStake.guild) * 1e18) /
176:             userStake.credit; /// upcast guild to prevent overflow

177:         uint256 guildAmount = (userMintRatio * amount) / 1e18;

250              uint256 creditReward = (uint256(userStake.guild) * deltaIndex) /
251:                 1e18;

252:             uint256 guildReward = (creditReward * rewardRatio) / 1e18;

302:         uint256 guildAfter = (mintRatio * uint256(userStake.credit)) / 1e18;

```
*GitHub*: [134](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L134-L134), [175](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L175-L176), [177](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L177-L177), [250](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L250-L251), [252](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L252-L252), [302](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L302-L302)

```solidity
File: src/tokens/ERC20Gauges.sol

192:         return (quantity * weight) / total;

```
*GitHub*: [192](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L192-L192)

```solidity
File: src/tokens/ERC20MultiVotes.sol

124:         return (a & b) + (a ^ b) / 2;

```
*GitHub*: [124](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L124-L124)

```solidity
File: src/tokens/ERC20RebaseDistributor.sol

150                  (delta * elapsed) /
151:                 (targetTimestamp - lastTimestamp);

213              uint256 percentChange = (sharesAfter * START_REBASING_SHARE_PRICE) /
214:                 sharesBefore;

216                  (delta * START_REBASING_SHARE_PRICE) /
217:                 percentChange;

256:         return (balance * START_REBASING_SHARE_PRICE) / sharePrice;

266          uint256 rebasedBalance = (shares * sharePrice) /
267:             START_REBASING_SHARE_PRICE +

365              uint256 newTargetSharePrice = (amount *
366                  START_REBASING_SHARE_PRICE +
367                  __rebasingSharePrice.targetValue *
368:                 _totalRebasingShares) / _totalRebasingShares;

```
*GitHub*: [150](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L150-L151), [213](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L213-L214), [216](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L216-L217), [256](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L256-L256), [266](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L266-L267), [365](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L365-L368)

</details>





### [G&#x2011;13] Duplicated `require()`/`revert()` checks should be refactored to a modifier or function to save deployment gas
This will cost more runtime gas, but will reduce deployment gas when the function is (optionally called via a modifier) called more than once as is the case for the examples below. Most projects do not make this trade-off, but it's available nonetheless.

*There are 16 instances of this issue:*

```solidity
File: src/governance/LendingTermOffboarding.sol

176:         require(canOffboard[term], "LendingTermOffboarding: quorum not met");

```
*GitHub*: [176](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOffboarding.sol#L176-L176)

```solidity
File: src/loan/LendingTerm.sol

504:         require(loan.closeTime == 0, "LendingTerm: loan closed");

505:         require(loan.callTime == 0, "LendingTerm: loan called");

572:         require(borrowTime != 0, "LendingTerm: loan not found");

573          require(
574              borrowTime < block.timestamp,
575              "LendingTerm: loan opened in same block"
576:         );

577:         require(loan.closeTime == 0, "LendingTerm: loan closed");

578:         require(loan.callTime == 0, "LendingTerm: loan called");

643:         require(loan.borrowTime != 0, "LendingTerm: loan not found");

646:         require(loan.closeTime == 0, "LendingTerm: loan closed");

649:         require(loan.callTime == 0, "LendingTerm: loan called");

659          require(
660              borrowTime < block.timestamp,
661              "LendingTerm: loan opened in same block"
662:         );

699:         require(loan.borrowTime != 0, "LendingTerm: loan not found");

702:         require(loan.closeTime == 0, "LendingTerm: loan closed");

```
*GitHub*: [504](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L504-L504), [505](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L505-L505), [572](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L572-L572), [573](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L573-L576), [577](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L577-L577), [578](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L578-L578), [643](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L643-L643), [646](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L646-L646), [649](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L649-L649), [659](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L659-L662), [699](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L699-L699), [702](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L702-L702)

```solidity
File: src/tokens/ERC20Gauges.sol

285:             require(isGauge(gauge), "ERC20Gauges: invalid gauge");

345:         require(weights.length == size, "ERC20Gauges: size mismatch");

```
*GitHub*: [285](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L285-L285), [345](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L345-L345)

```solidity
File: src/tokens/GuildToken.sol

252              require(
253                  _lastGaugeLossApplied >= _lastGaugeLoss,
254                  "GuildToken: pending loss"
255:             );

```
*GitHub*: [252](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L252-L255)



### [G&#x2011;14] Initializers can be marked `payable`
Payable functions cost less gas to execute, since the compiler does not have to add extra checks to ensure that a payment wasn't provided. An initializer can safely be marked as payable, since only the deployer would be able to pass funds, and the project itself would not pass any funds.

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



### [G&#x2011;15] Mappings are cheaper to use than storage arrays
When using storage arrays, solidity adds an internal lookup of the array's length (a Gcoldsload **2100 gas**) to ensure you don't read past the array's end. You can avoid this lookup by using a `mapping` and storing the number of entries in a separate storage variable. In cases where you have sentinel values (e.g. 'zero' means invalid), you can avoid length checks

*There is one instance of this issue:*

```solidity
File: src/tokens/ERC20MultiVotes.sol

49:      mapping(address => Checkpoint[]) private _checkpoints;

```
*GitHub*: [49](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L49-L49)



### [G&#x2011;16] Multiple `address`/ID mappings can be combined into a single `mapping` of an `address`/ID to a `struct`
Saves a storage slot for each of the removed mappings (i.e. this finding is not about runtime savings). The instances below refer to both mappings using the same key in the same function, so the mappings are related.

*There are 7 instances of this issue:*

```solidity
File: src/governance/LendingTermOnboarding.sol

/// @audit combine into a `struct`: created,lastProposal
23   contract LendingTermOnboarding is GuildGovernor {
24:      /// @notice minimum delay between proposals of onboarding of a given term

```
*GitHub*: [23](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOnboarding.sol#L23-L24)

```solidity
File: src/tokens/ERC20Gauges.sol

/// @audit combine into a `struct`: _userGauges,getUserGaugeWeight,getUserWeight
/// @audit combine into a `struct`: gaugeType,getGaugeWeight
/// @audit combine into a `struct`: _userGauges,canExceedMaxGauges,getUserGaugeWeight
58:  abstract contract ERC20Gauges is ERC20 {

```
*GitHub*: [58](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L58-L58), [58](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L58-L58), [58](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L58-L58)

```solidity
File: src/tokens/ERC20MultiVotes.sol

/// @audit combine into a `struct`: _delegates,_delegatesVotesCount
/// @audit combine into a `struct`: _delegates,_delegatesVotesCount,userDelegatedVotes
/// @audit combine into a `struct`: _delegates,_delegatesVotesCount,canContractExceedMaxDelegates,userDelegatedVotes
35:  abstract contract ERC20MultiVotes is ERC20Permit {

```
*GitHub*: [35](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L35-L35), [35](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L35-L35), [35](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L35-L35)



### [G&#x2011;17] Multiple accesses of a `memory`/`calldata` array should use a local variable cache
The instances below point to the second+ access of a value inside a `memory`/`calldata` array, within a function. Caching avoids recalculating the array offsets into `memory`/`calldata`

*There are 3 instances of this issue:*

```solidity
File: src/core/CoreRef.sol

/// @audit calls...[]
98:              uint256 value = calls[i].value;

/// @audit calls...[]
99:              bytes calldata callData = calls[i].callData;

```
*GitHub*: [98](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRef.sol#L98-L98), [99](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRef.sol#L99-L99)

```solidity
File: src/governance/ProfitManager.sol

/// @audit creditEarned...[]
484:                 totalCreditEarned += creditEarned[i];

```
*GitHub*: [484](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L484-L484)



### [G&#x2011;18] Multiple accesses of a storage array should use a local variable cache
The instances below point to the second+ access of a value inside a storage array, within a function. Caching an array index value in a local `storage` or `calldata` variable when the value is accessed [multiple times](https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0), saves **~42 gas per access** due to not having to recalculate the array's keccak256 hash (`Gkeccak256` - **30 gas**) and that calculation's associated stack operations.

*There is one instance of this issue:*

```solidity
File: src/tokens/ERC20MultiVotes.sol

/// @audit _checkpoints...[]
82:          return pos == 0 ? 0 : _checkpoints[account][pos - 1].votes;

```
*GitHub*: [82](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L82-L82)



### [G&#x2011;19] Reduce deployment costs by tweaking contracts' metadata
See [this](https://www.rareskills.io/post/solidity-metadata) link, at its bottom, for full details

*There are 13 instances of this issue:*

<details>
<summary>see instances</summary>


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





### [G&#x2011;20] State variables only set in the constructor should be declared `immutable`
Avoids a Gsset (**20000 gas**) in the constructor, and replaces the first access in each transaction (Gcoldsload - **2100 gas**) and each access thereafter (Gwarmacces - **100 gas**) with a `PUSH32` (**3 gas**). 

While `string`s are not value types, and therefore cannot be `immutable`/`constant` if not hard-coded outside of the constructor, the same behavior can be achieved by making the current contract `abstract` with `virtual` functions for the `string` accessors, and having a child contract override the functions with the hard-coded implementation-specific values.

*There are 2 instances of this issue:*

```solidity
File: src/governance/LendingTermOffboarding.sol

64:      uint256 public nOffboardingsInProgress;

```
*GitHub*: [64](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOffboarding.sol#L64-L64)

```solidity
File: src/loan/AuctionHouse.sol

54:      uint256 public nAuctionsInProgress;

```
*GitHub*: [54](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/AuctionHouse.sol#L54-L54)



### [G&#x2011;21] Events should be emitted outside of loops
Emitting an event has an overhead of **375 gas**, which will be incurred on every iteration of the loop. It is cheaper to `emit` only [once](https://github.com/ethereum/EIPs/blob/adad5968fd6de29902174e0cb51c8fc3dceb9ab5/EIPS/eip-1155.md?plain=1#L68) after the loop has finished.

*There are 12 instances of this issue:*

```solidity
File: src/governance/ProfitManager.sol

/// @audit  claimGaugeRewards(), claimRewards()
/// @audit  claimGaugeRewards(), _incrementGaugeWeight(), incrementGauges()
/// @audit  claimGaugeRewards(), _decrementGaugeWeight(), decrementGauges()
/// @audit  claimGaugeRewards(), _decrementGaugeWeight(), _decrementWeightUntilFree()
433:             emit ClaimRewards(block.timestamp, user, gauge, creditEarned);

```
*GitHub*: [433](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L433-L433), [433](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L433-L433), [433](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L433-L433), [433](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L433-L433)

```solidity
File: src/loan/AuctionHouse.sol

/// @audit  startAuction(), _call(), callMany()
106          emit AuctionStart(
107              block.timestamp,
108              loanId,
109              LendingTerm(msg.sender).collateralToken(),
110              loan.collateralAmount,
111              callDebt
112:         );

```
*GitHub*: [106](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/AuctionHouse.sol#L106-L112)

```solidity
File: src/loan/LendingTerm.sol

/// @audit  _call(), callMany()
674:         emit LoanCall(block.timestamp, loanId);

```
*GitHub*: [674](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L674-L674)

```solidity
File: src/tokens/ERC20Gauges.sol

/// @audit  _incrementGaugeWeight(), incrementGauges()
/// @audit  _incrementGaugeWeight(), _incrementGaugeWeight(), incrementGauges()
246:         emit IncrementGaugeWeight(user, gauge, weight);

/// @audit  _decrementGaugeWeight(), decrementGauges()
/// @audit  _decrementGaugeWeight(), _decrementWeightUntilFree()
/// @audit  _decrementGaugeWeight(), _decrementGaugeWeight(), decrementGauges()
/// @audit  _decrementGaugeWeight(), _decrementGaugeWeight(), _decrementWeightUntilFree()
331:         emit DecrementGaugeWeight(user, gauge, weight);

```
*GitHub*: [246](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L246-L246), [246](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L246-L246), [331](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L331-L331), [331](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L331-L331), [331](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L331-L331), [331](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L331-L331)



### [G&#x2011;22] Emitting constants wastes gas
Every event parameter costs `Glogdata` (**8 gas**) per byte. You can avoid this extra cost, in cases where you're emitting a constant, by creating a second version of the event, which doesn't have the parameter (and have users look to the contract's variables for its value instead), and using the new event in the cases shown below.

*There are 6 instances of this issue:*

```solidity
File: src/governance/LendingTermOffboarding.sol

106          emit OffboardSupport(
107              block.timestamp,
108              term,
109              block.number,
110              address(0),
111              1
112:         );

```
*GitHub*: [106](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOffboarding.sol#L106-L112)

```solidity
File: src/governance/ProfitManager.sol

107:         emit MinBorrowUpdate(block.timestamp, 100e18);

311:                 emit TermSurplusBufferUpdate(block.timestamp, gauge, 0);

328:                 emit SurplusBufferUpdate(block.timestamp, 0);

```
*GitHub*: [107](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L107-L107), [311](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L311-L311), [328](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L328-L328)

```solidity
File: src/loan/AuctionHouse.sol

223          emit AuctionEnd(
224              block.timestamp,
225              loanId,
226              LendingTerm(_lendingTerm).collateralToken(),
227              0, // collateralSold
228              0 // debtRecovered
229:         );

```
*GitHub*: [223](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/AuctionHouse.sol#L223-L229)

```solidity
File: src/loan/LendingTerm.sol

721:         emit LoanClose(block.timestamp, loanId, LoanCloseType.Forgive, 0);

```
*GitHub*: [721](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L721-L721)



### [G&#x2011;23] Stack variable is only used once
If the variable is only accessed once, it's cheaper to use the assigned value directly that one time, and save the **3 gas** the extra stack assignment would spend

*There are 71 instances of this issue:*

<details>
<summary>see instances</summary>


```solidity
File: src/core/CoreRef.sol

50:          address oldCore = address(_core);

```
*GitHub*: [50](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRef.sol#L50-L50)

```solidity
File: src/governance/GuildVetoGovernor.sol

317          uint256 timelockExecutionTime = TimelockController(payable(timelock))
318:             .getTimestamp(timelockId);

325          (
326              address[] memory targets,
327              uint256[] memory values,
328              bytes[] memory calldatas,
329              string memory description
330:         ) = _getVetoCalls(timelockId);

351          (
352              address[] memory targets,
353              uint256[] memory values,
354              bytes[] memory calldatas,
355              string memory description
356:         ) = _getVetoCalls(timelockId);

```
*GitHub*: [317](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L317-L318), [325](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L325-L330), [351](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L351-L356)

```solidity
File: src/governance/LendingTermOnboarding.sol

114          (bool success, bytes memory returned) = params.collateralToken.call(
115              abi.encodeWithSelector(IERC20.totalSupply.selector)
116:         );

199:         bool isGauge = GuildToken(guildToken).isGauge(term);

203          (
204              address[] memory targets,
205              uint256[] memory values,
206              bytes[] memory calldatas,
207              string memory description
208:         ) = getOnboardProposeArgs(term);

```
*GitHub*: [114](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOnboarding.sol#L114-L116), [199](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOnboarding.sol#L199-L199), [203](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOnboarding.sol#L203-L208)

```solidity
File: src/loan/AuctionHouse.sol

139:             uint256 elapsed = block.timestamp - _startTime; // [0, midPoint[

149:             uint256 PHASE_2_DURATION = auctionDuration - midPoint;

150:             uint256 elapsed = block.timestamp - _startTime - midPoint; // [0, PHASE_2_DURATION[

205:         (, uint256 creditAsked) = getBidDetail(loanId);

```
*GitHub*: [139](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/AuctionHouse.sol#L139-L139), [149](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/AuctionHouse.sol#L149-L149), [150](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/AuctionHouse.sol#L150-L150), [205](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/AuctionHouse.sol#L205-L205)

```solidity
File: src/loan/LendingTerm.sol

218          uint256 interest = (borrowAmount *
219              params.interestRate *
220              (block.timestamp - borrowTime)) /
221              YEAR /
222:             1e18;

228          uint256 creditMultiplier = ProfitManager(refs.profitManager)
229:             .creditMultiplier();

278:         uint256 gaugeType = GuildToken(_guildToken).gaugeType(address(this));

296          uint256 gaugeWeightTolerance = ProfitManager(refs.profitManager)
297:             .gaugeWeightTolerance();

312:         uint256 remainingDebtCeiling = debtCeilingBefore - _issuance; // always >0

319:         uint256 otherGaugesWeight = totalWeight - toleratedGaugeWeight; // always >0

320          uint256 maxBorrow = (remainingDebtCeiling * totalWeight) /
321:             otherGaugesWeight;

357          uint256 maxBorrow = (collateralAmount *
358:             params.maxDebtPerCollateralToken) / creditMultiplier;

381          uint256 gaugeWeightTolerance = ProfitManager(refs.profitManager)
382:             .gaugeWeightTolerance();

512          uint256 creditMultiplier = ProfitManager(refs.profitManager)
513:             .creditMultiplier();

514          uint256 principal = (borrowAmount * loan.borrowCreditMultiplier) /
515:             creditMultiplier;

583          uint256 creditMultiplier = ProfitManager(refs.profitManager)
584:             .creditMultiplier();

642:         uint256 borrowTime = loan.borrowTime;

709          uint256 creditMultiplier = ProfitManager(refs.profitManager)
710:             .creditMultiplier();

711:         uint256 borrowAmount = loans[loanId].borrowAmount;

712          uint256 principal = (borrowAmount *
713:             loans[loanId].borrowCreditMultiplier) / creditMultiplier;

714:         int256 pnl = -int256(principal);

751          uint256 creditMultiplier = ProfitManager(refs.profitManager)
752:             .creditMultiplier();

```
*GitHub*: [218](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L218-L222), [228](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L228-L229), [278](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L278-L278), [296](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L296-L297), [312](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L312-L312), [319](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L319-L319), [320](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L320-L321), [357](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L357-L358), [381](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L381-L382), [512](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L512-L513), [514](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L514-L515), [583](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L583-L584), [642](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L642-L642), [709](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L709-L710), [711](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L711-L711), [712](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L712-L713), [714](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L714-L714), [751](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L751-L752)

```solidity
File: src/loan/SimplePSM.sol

75:          uint256 decimals = uint256(ERC20(_pegToken).decimals());

81           uint256 creditMultiplier = ProfitManager(profitManager)
82:              .creditMultiplier();

90           uint256 creditMultiplier = ProfitManager(profitManager)
91:              .creditMultiplier();

```
*GitHub*: [75](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SimplePSM.sol#L75-L75), [81](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SimplePSM.sol#L81-L82), [90](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SimplePSM.sol#L90-L91)

```solidity
File: src/loan/SurplusGuildMinter.sol

160          (, UserStake memory userStake, bool slashed) = getRewards(
161              msg.sender,
162              term
163:         );

175          uint256 userMintRatio = (uint256(userStake.guild) * 1e18) /
176:             userStake.credit; /// upcast guild to prevent overflow

295:         (, UserStake memory userStake, bool slashed) = getRewards(user, term);

```
*GitHub*: [160](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L160-L163), [175](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L175-L176), [295](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L295-L295)

```solidity
File: src/tokens/ERC20Gauges.sol

235:         bool added = _userGauges[user].add(gauge); // idempotent add

400:         bool previouslyDeprecated = _deprecatedGauges.remove(gauge);

```
*GitHub*: [235](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L235-L235), [400](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L400-L400)

```solidity
File: src/tokens/ERC20MultiVotes.sol

323:         uint256 free = freeVotes(delegator);

329:         bool newDelegate = _delegates[delegator].add(delegatee); // idempotent add

```
*GitHub*: [323](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L323-L323), [329](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L329-L329)

```solidity
File: src/tokens/ERC20RebaseDistributor.sol

146:             uint256 elapsed = block.timestamp - lastTimestamp;

147:             uint256 delta = targetValue - lastValue;

213              uint256 percentChange = (sharesAfter * START_REBASING_SHARE_PRICE) /
214:                 sharesBefore;

215              uint256 targetNewSharePrice = currentRebasingSharePrice +
216                  (delta * START_REBASING_SHARE_PRICE) /
217:                 percentChange;

230:         uint256 _unmintedRebaseRewards = interpolatedValue(val);

290:         uint256 balance = ERC20.balanceOf(account);

316          uint256 rebasedBalance = _shares2balance(
317              shares,
318              currentRebasingSharePrice,
319              0,
320              rawBalance
321:         );

365              uint256 newTargetSharePrice = (amount *
366                  START_REBASING_SHARE_PRICE +
367                  __rebasingSharePrice.targetValue *
368:                 _totalRebasingShares) / _totalRebasingShares;

377:             uint256 _unmintedRebaseRewards = unmintedRebaseRewards();

417:         uint256 _unmintedRebaseRewards = interpolatedValue(val);

470              uint256 rebasedBalance = _shares2balance(
471                  _rebasingState.nShares,
472                  _rebasingSharePrice,
473                  0,
474                  balanceBefore
475:             );

490:             uint256 balanceAfter = balanceBefore - amount;

495:             uint256 sharesBurnt = _rebasingState.nShares - sharesAfter;

531:             uint256 sharesReceived = sharesAfter - _rebasingState.nShares;

567:             uint256 shares = uint256(rebasingStateFrom.nShares);

568              uint256 rebasedBalance = _shares2balance(
569                  shares,
570                  _rebasingSharePrice,
571                  0,
572                  fromBalanceBefore
573:             );

584:         bool success = ERC20.transfer(to, amount);

589:             uint256 fromBalanceAfter = fromBalanceBefore - amount;

594:             uint256 sharesSpent = rebasingStateFrom.nShares - fromSharesAfter;

618:             uint256 sharesReceived = toSharesAfter - rebasingStateTo.nShares;

661:             uint256 shares = uint256(rebasingStateFrom.nShares);

662              uint256 rebasedBalance = _shares2balance(
663                  shares,
664                  _rebasingSharePrice,
665                  0,
666                  fromBalanceBefore
667:             );

678:         bool success = ERC20.transferFrom(from, to, amount);

683:             uint256 fromBalanceAfter = fromBalanceBefore - amount;

688:             uint256 sharesSpent = rebasingStateFrom.nShares - fromSharesAfter;

712:             uint256 sharesReceived = toSharesAfter - rebasingStateTo.nShares;

```
*GitHub*: [146](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L146-L146), [147](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L147-L147), [213](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L213-L214), [215](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L215-L217), [230](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L230-L230), [290](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L290-L290), [316](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L316-L321), [365](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L365-L368), [377](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L377-L377), [417](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L417-L417), [470](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L470-L475), [490](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L490-L490), [495](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L495-L495), [531](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L531-L531), [567](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L567-L567), [568](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L568-L573), [584](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L584-L584), [589](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L589-L589), [594](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L594-L594), [618](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L618-L618), [661](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L661-L661), [662](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L662-L667), [678](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L678-L678), [683](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L683-L683), [688](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L688-L688), [712](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L712-L712)

```solidity
File: src/tokens/GuildToken.sol

226:             uint256 debtCeilingAfterDecrement = LendingTerm(gauge).debtCeiling(-int256(weight));

```
*GitHub*: [226](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L226-L226)

```solidity
File: src/utils/RateLimitedV2.sol

84:          uint256 elapsed = block.timestamp.safeCastTo32() - lastBufferUsedTime;

99:          uint32 blockTimestamp = block.timestamp.safeCastTo32();

100:         uint224 newBufferStored = (newBuffer - amount).safeCastTo224();

124:         uint32 blockTimestamp = block.timestamp.safeCastTo32();

126          uint224 newBufferStored = Math
127              .min(newBuffer + amount, _bufferCap)
128:             .safeCastTo224();

```
*GitHub*: [84](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/utils/RateLimitedV2.sol#L84-L84), [99](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/utils/RateLimitedV2.sol#L99-L99), [100](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/utils/RateLimitedV2.sol#L100-L100), [124](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/utils/RateLimitedV2.sol#L124-L124), [126](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/utils/RateLimitedV2.sol#L126-L128)

</details>





### [G&#x2011;24] Splitting `require()` statements that use `&&` saves gas
See [this issue](https://github.com/code-423n4/2022-01-xdefi-findings/issues/128) which describes the fact that there is a larger deployment gas cost, but with enough runtime calls, the change ends up being cheaper by **3 gas**

*There is one instance of this issue:*

```solidity
File: src/governance/ProfitManager.sol

161          assert(
162              credit == address(0) && guild == address(0) && psm == address(0)
163:         );

```
*GitHub*: [161](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L161-L163)



### [G&#x2011;25] String literals passed to `abi.encode()`/`abi.encodePacked()` should not be split by commas
String literals can be split into multiple parts and still be considered as a single string literal. Adding commas between each chunk makes it no longer a single string, and instead multiple strings. EACH new comma costs [***21 gas***](https://gist.github.com/IllIllI000/837d1b36c16c9bfe1010f9f775a09bbf) due to stack operations and separate `MSTORE`s

*There is one instance of this issue:*

```solidity
File: src/governance/LendingTermOnboarding.sol

/// @audit 1 commas
230           description = string.concat(
231               "[",
232               Strings.toString(block.number),
233               "]",
234               " Enable term ",
235               Strings.toHexString(term)
236:          );

```
*GitHub*: [230](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOnboarding.sol#L230-L236)



### [G&#x2011;26] Update OpenZeppelin dependency to save gas
Every release contains new gas optimizations. Use the latest version to take advantage of this

*There are 39 instances of this issue:*

<details>
<summary>see instances</summary>


```solidity
File: src/core/Core.sol

5:   import {AccessControlEnumerable} from "@openzeppelin/contracts/access/AccessControlEnumerable.sol";

```
*GitHub*: [5](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/Core.sol#L5-L5)

```solidity
File: src/core/CoreRef.sol

7:   import {Pausable} from "@openzeppelin/contracts/security/Pausable.sol";

```
*GitHub*: [7](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRef.sol#L7-L7)

```solidity
File: src/governance/GuildGovernor.sol

4:   import {Governor, IGovernor} from "@openzeppelin/contracts/governance/Governor.sol";

5:   import {GovernorSettings} from "@openzeppelin/contracts/governance/extensions/GovernorSettings.sol";

6:   import {GovernorTimelockControl} from "@openzeppelin/contracts/governance/extensions/GovernorTimelockControl.sol";

7:   import {GovernorVotes, IERC165} from "@openzeppelin/contracts/governance/extensions/GovernorVotes.sol";

8:   import {GovernorCountingSimple} from "@openzeppelin/contracts/governance/extensions/GovernorCountingSimple.sol";

9:   import {IVotes} from "@openzeppelin/contracts/governance/utils/IVotes.sol";

10:  import {TimelockController} from "@openzeppelin/contracts/governance/TimelockController.sol";

```
*GitHub*: [4](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildGovernor.sol#L4-L4), [5](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildGovernor.sol#L5-L5), [6](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildGovernor.sol#L6-L6), [7](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildGovernor.sol#L7-L7), [8](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildGovernor.sol#L8-L8), [9](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildGovernor.sol#L9-L9), [10](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildGovernor.sol#L10-L10)

```solidity
File: src/governance/GuildTimelockController.sol

6:   import {TimelockController} from "@openzeppelin/contracts/governance/TimelockController.sol";

```
*GitHub*: [6](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildTimelockController.sol#L6-L6)

```solidity
File: src/governance/GuildVetoGovernor.sol

4:   import {Governor, IGovernor} from "@openzeppelin/contracts/governance/Governor.sol";

5:   import {GovernorTimelockControl} from "@openzeppelin/contracts/governance/extensions/GovernorTimelockControl.sol";

6:   import {GovernorVotes, IERC165} from "@openzeppelin/contracts/governance/extensions/GovernorVotes.sol";

7:   import {GovernorCountingSimple} from "@openzeppelin/contracts/governance/extensions/GovernorCountingSimple.sol";

8:   import {IVotes} from "@openzeppelin/contracts/governance/utils/IVotes.sol";

9:   import {TimelockController} from "@openzeppelin/contracts/governance/TimelockController.sol";

```
*GitHub*: [4](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L4-L4), [5](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L5-L5), [6](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L6-L6), [7](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L7-L7), [8](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L8-L8), [9](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L9-L9)

```solidity
File: src/governance/LendingTermOnboarding.sol

4:   import {Clones} from "@openzeppelin/contracts/proxy/Clones.sol";

5:   import {IERC20} from "@openzeppelin/contracts/token/ERC20/IERC20.sol";

6:   import {Strings} from "@openzeppelin/contracts/utils/Strings.sol";

7:   import {AccessControl} from "@openzeppelin/contracts/access/AccessControl.sol";

8:   import {Governor, IGovernor} from "@openzeppelin/contracts/governance/Governor.sol";

```
*GitHub*: [4](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOnboarding.sol#L4-L4), [5](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOnboarding.sol#L5-L5), [6](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOnboarding.sol#L6-L6), [7](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOnboarding.sol#L7-L7), [8](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOnboarding.sol#L8-L8)

```solidity
File: src/loan/LendingTerm.sol

4:   import {IERC20} from "@openzeppelin/contracts/token/ERC20/IERC20.sol";

5:   import {SafeERC20} from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

6:   import {IERC20Permit} from "@openzeppelin/contracts/token/ERC20/extensions/IERC20Permit.sol";

```
*GitHub*: [4](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L4-L4), [5](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L5-L5), [6](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L6-L6)

```solidity
File: src/loan/SimplePSM.sol

4:   import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

5:   import {SafeERC20} from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

```
*GitHub*: [4](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SimplePSM.sol#L4-L4), [5](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SimplePSM.sol#L5-L5)

```solidity
File: src/tokens/CreditToken.sol

4:   import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

5:   import {ERC20Permit} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Permit.sol";

6:   import {ERC20Burnable} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";

```
*GitHub*: [4](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/CreditToken.sol#L4-L4), [5](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/CreditToken.sol#L5-L5), [6](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/CreditToken.sol#L6-L6)

```solidity
File: src/tokens/ERC20Gauges.sol

4:   import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

5:   import {EnumerableSet} from "@openzeppelin/contracts/utils/structs/EnumerableSet.sol";

```
*GitHub*: [4](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L4-L4), [5](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L5-L5)

```solidity
File: src/tokens/ERC20MultiVotes.sol

6:   import {ERC20Permit} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Permit.sol";

7:   import {EnumerableSet} from "@openzeppelin/contracts/utils/structs/EnumerableSet.sol";

```
*GitHub*: [6](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L6-L6), [7](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L7-L7)

```solidity
File: src/tokens/ERC20RebaseDistributor.sol

4:   import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

```
*GitHub*: [4](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20RebaseDistributor.sol#L4-L4)

```solidity
File: src/tokens/GuildToken.sol

4:   import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

5:   import {ERC20Permit} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Permit.sol";

6:   import {ERC20Burnable} from "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";

7:   import {EnumerableSet} from "@openzeppelin/contracts/utils/structs/EnumerableSet.sol";

```
*GitHub*: [4](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L4-L4), [5](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L5-L5), [6](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L6-L6), [7](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L7-L7)

```solidity
File: src/utils/RateLimitedV2.sol

4:   import {Math} from "@openzeppelin/contracts/utils/math/Math.sol";

```
*GitHub*: [4](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/utils/RateLimitedV2.sol#L4-L4)

</details>





### [G&#x2011;27] Use internal functions inside modifiers to save gas
This will cost more runtime gas, but will reduce deployment gas when the modifier is called more than once as is the case for the examples below. Most projects do not make this trade-off, but it's available nonetheless.

*There is one instance of this issue:*

```solidity
File: src/core/CoreRef.sol

24       modifier onlyCoreRole(bytes32 role) {
25           require(_core.hasRole(role, msg.sender), "UNAUTHORIZED");
26           _;
27:      }

```
*GitHub*: [24](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/CoreRef.sol#L24-L27)



### [G&#x2011;28] Use short-circuit evaluation to avoid external calls
By evaluating expressions involving constants, literals, and local variables before ones involving external calls, you can avoid unnecessarily executing the calls in the unhappy path.

*There is one instance of this issue:*

```solidity
File: src/loan/LendingTerm.sol

/// @audit move the expression involving `isDeprecatedGauge()` to the right
653              GuildToken(refs.guildToken).isDeprecatedGauge(address(this)) ||
654:                 partialRepayDelayPassed(loanId),

```
*GitHub*: [653](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L653-L654)



### [G&#x2011;29] Use short-circuit evaluation to avoid reading state variables
By evaluating expressions involving constants, literals, and local variables before ones involving storage variables, you can avoid unnecessarily reading the storage variables in the unhappy path.

*There are 4 instances of this issue:*

```solidity
File: src/loan/LendingTerm.sol

/// @audit move the expression involving `refs` to the right
653              GuildToken(refs.guildToken).isDeprecatedGauge(address(this)) ||
654:                 partialRepayDelayPassed(loanId),

/// @audit move the expression involving `loans` to the right
745              collateralOut == loans[loanId].collateralAmount ||
746:                 collateralOut == 0,

```
*GitHub*: [653](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L653-L654), [745](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L745-L746)

```solidity
File: src/tokens/GuildToken.sol

/// @audit move the expression involving `transferable` to the right
/// @audit move the expression involving `transferable` to the right
188:             transferable || from == address(0) || to == address(0),

```
*GitHub*: [188](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L188-L188), [188](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L188-L188)



### [G&#x2011;30] Using `private` rather than `public`, saves gas
For constants, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

*There are 66 instances of this issue:*

<details>
<summary>see instances</summary>


```solidity
File: src/governance/GuildVetoGovernor.sol

86:      address public timelock;

```
*GitHub*: [86](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/GuildVetoGovernor.sol#L86-L86)

```solidity
File: src/governance/LendingTermOffboarding.sol

39:      uint256 public quorum;

42:      address public immutable guildToken;

45:      address public immutable psm;

49:      mapping(uint256 => mapping(address => uint256)) public polls;

53       mapping(address => mapping(uint256 => mapping(address => uint256)))
54:          public userPollVotes;

58:      mapping(address => uint256) public lastPollBlock;

61:      mapping(address => bool) public canOffboard;

64:      uint256 public nOffboardingsInProgress;

```
*GitHub*: [39](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOffboarding.sol#L39-L39), [42](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOffboarding.sol#L42-L42), [45](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOffboarding.sol#L45-L45), [49](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOffboarding.sol#L49-L49), [53](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOffboarding.sol#L53-L54), [58](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOffboarding.sol#L58-L58), [61](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOffboarding.sol#L61-L61), [64](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOffboarding.sol#L64-L64)

```solidity
File: src/governance/LendingTermOnboarding.sol

27:      mapping(address => uint256) public lastProposal;

30:      mapping(address => bool) public implementations;

32:      address public immutable guildToken;

34:      uint256 public immutable gaugeType;

38:      mapping(address => uint256) public created;

41:      address public immutable profitManager;

43:      address public immutable auctionHouse;

45:      address public immutable creditMinter;

47:      address public immutable creditToken;

```
*GitHub*: [27](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOnboarding.sol#L27-L27), [30](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOnboarding.sol#L30-L30), [32](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOnboarding.sol#L32-L32), [34](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOnboarding.sol#L34-L34), [38](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOnboarding.sol#L38-L38), [41](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOnboarding.sol#L41-L41), [43](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOnboarding.sol#L43-L43), [45](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOnboarding.sol#L45-L45), [47](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/LendingTermOnboarding.sol#L47-L47)

```solidity
File: src/governance/ProfitManager.sol

32:      address public guild;

35:      address public credit;

38:      address public psm;

41:      mapping(address => uint256) public gaugeProfitIndex;

44:      mapping(address => mapping(address => uint256)) public userGaugeProfitIndex;

70:      uint256 public surplusBuffer;

77:      mapping(address => uint256) public termSurplusBuffer;

87:      uint256 public creditMultiplier = 1e18;

104:     uint256 public gaugeWeightTolerance = 1.2e18; // 120%

```
*GitHub*: [32](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L32-L32), [35](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L35-L35), [38](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L38-L38), [41](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L41-L41), [44](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L44-L44), [70](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L70-L70), [77](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L77-L77), [87](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L87-L87), [104](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/governance/ProfitManager.sol#L104-L104)

```solidity
File: src/loan/AuctionHouse.sol

31:      uint256 public immutable midPoint;

38:      uint256 public immutable auctionDuration;

54:      uint256 public nAuctionsInProgress;

```
*GitHub*: [31](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/AuctionHouse.sol#L31-L31), [38](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/AuctionHouse.sol#L38-L38), [54](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/AuctionHouse.sol#L54-L54)

```solidity
File: src/loan/LendingTerm.sol

73:      mapping(bytes32 => uint256) public lastPartialRepay;

95:      uint256 public issuance;

```
*GitHub*: [73](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L73-L73), [95](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/LendingTerm.sol#L95-L95)

```solidity
File: src/loan/SimplePSM.sol

29:      address public immutable profitManager;

32:      address public immutable credit;

35:      address public immutable pegToken;

39:      uint256 public pegTokenBalance;

43:      uint256 public immutable decimalCorrection;

46:      bool public redemptionsPaused;

```
*GitHub*: [29](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SimplePSM.sol#L29-L29), [32](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SimplePSM.sol#L32-L32), [35](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SimplePSM.sol#L35-L35), [39](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SimplePSM.sol#L39-L39), [43](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SimplePSM.sol#L43-L43), [46](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SimplePSM.sol#L46-L46)

```solidity
File: src/loan/SurplusGuildMinter.sol

56:      address public immutable profitManager;

59:      address public immutable credit;

62:      address public immutable guild;

65:      address public immutable rlgm;

70:      uint256 public mintRatio;

75:      uint256 public rewardRatio;

```
*GitHub*: [56](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L56-L56), [59](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L59-L59), [62](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L62-L62), [65](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L65-L65), [70](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L70-L70), [75](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SurplusGuildMinter.sol#L75-L75)

```solidity
File: src/rate-limits/RateLimitedMinter.sol

19:      address public immutable token;

22:      bytes32 public immutable role;

```
*GitHub*: [19](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/rate-limits/RateLimitedMinter.sol#L19-L19), [22](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/rate-limits/RateLimitedMinter.sol#L22-L22)

```solidity
File: src/tokens/ERC20Gauges.sol

66:      mapping(address => mapping(address => uint256)) public getUserGaugeWeight;

70:      mapping(address => uint256) public getUserWeight;

74:      mapping(address => uint256) public getGaugeWeight;

77:      uint256 public totalWeight;

81:      mapping(uint256 => uint256) public totalTypeWeight;

84:      mapping(address => uint256) public gaugeType;

390:     uint256 public maxGauges;

393:     mapping(address => bool) public canExceedMaxGauges;

```
*GitHub*: [66](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L66-L66), [70](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L70-L70), [74](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L74-L74), [77](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L77-L77), [81](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L81-L81), [84](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L84-L84), [390](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L390-L390), [393](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20Gauges.sol#L393-L393)

```solidity
File: src/tokens/ERC20MultiVotes.sol

141:     uint256 public maxDelegates;

144:     mapping(address => bool) public canContractExceedMaxDelegates;

207:     mapping(address => uint256) public userDelegatedVotes;

```
*GitHub*: [141](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L141-L141), [144](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L144-L144), [207](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/ERC20MultiVotes.sol#L207-L207)

```solidity
File: src/tokens/GuildToken.sol

41:      address public profitManager;

117:     mapping(address => uint256) public lastGaugeLoss;

120:     mapping(address => mapping(address => uint256)) public lastGaugeLossApplied;

169:     bool public transferable; // default = false

```
*GitHub*: [41](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L41-L41), [117](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L117-L117), [120](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L120-L120), [169](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/tokens/GuildToken.sol#L169-L169)

```solidity
File: src/utils/RateLimitedV2.sol

18:      uint256 public immutable MAX_RATE_LIMIT_PER_SECOND;

23:      uint128 public rateLimitPerSecond;

26:      uint128 public bufferCap;

31:      uint32 public lastBufferUsedTime;

34:      uint224 public bufferStored;

```
*GitHub*: [18](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/utils/RateLimitedV2.sol#L18-L18), [23](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/utils/RateLimitedV2.sol#L23-L23), [26](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/utils/RateLimitedV2.sol#L26-L26), [31](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/utils/RateLimitedV2.sol#L31-L31), [34](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/utils/RateLimitedV2.sol#L34-L34)

</details>


