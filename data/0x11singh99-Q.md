# Quality Assurance Report

| QA Issues | Issues                                         | Instances |
| --------- | ---------------------------------------------- | --------- |
| [L-01](#l-01-missing-address0-check) | Missing address(0) check | 3 |
| [L-02](#l-02-check-value-for-0-before-dividing) | Check value for 0 before dividing | 2 |
| [N-01](#n-01-missing-event-emission-for-critical-state-changes) | Missing event emission for critical state changes | 11 |
| [N-02](#n-02-use-an-efficient-logic-in-setter-functions) | Use an efficient logic in setter functions | 3 |
| [N-03](#n-03-in-solidity-following-a-style-guide-is-important-for-readability-and-maintainability) | In Solidity, following a style guide is important for readability and maintainability | 3 |
| [N-04](#n-04-stick-with-one-way-of-using-curly-brackets-for-single-instruction-if-conditions) | Stick with one way of using curly-brackets for single-instruction if conditions | 21 |
| [N-05](#n-05-unnecessary-type-casting) | Unnecessary Type Casting | 7 |
| [N-06](#n-06-typos) | Typos | 2 |
| [N-07](#n-07-use-already-cached-value) | Use already cached value | 1 |
| [N-08](#n-08-named-return-is-confusing) | Named return is confusing | 3 |


# Low-Severity

## [L-01] Missing address(0) check 

**Note: Missed by bot-report**

_3 instances_

<details><summary>See Instances</summary>

If you don't check whether term is equal to address(0) before setting the created[term] value based on block.timestamp, a transaction calling this function with term as address(0) will proceed and set the created[address(0)] value to the current block.timestamp.
This behavior could have unintended consequences depending on your contract's logic. Storing a timestamp for address(0) might not make sense in certain contexts, and it could potentially lead to confusion or unexpected behavior when retrieving or using this value later in your contract.

```solidity
File : src/governance/LendingTermOnboarding.sol

153:    address term = Clones.clone(implementation);
154:        LendingTerm(term).initialize(
155:            address(core()),
156:            LendingTerm.LendingTermReferences({
157:                profitManager: profitManager,
158:                guildToken: guildToken,
159:                auctionHouse: auctionHouse,
160:                creditMinter: creditMinter,
161:                creditToken: creditToken
162:            }),
163:            params
164:        );
165:        created[term] = block.timestamp; // @audit check `term` for address(0)
166:        emit TermCreated(block.timestamp, implementation, term, params);
167:        return term;
168:    }

```
[153-168](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOnboarding.sol#L153C8-L168C6)

```solidity
File : src/tokens/ERC20Gauges.sol

452:     function _setCanExceedMaxGauges(
453:         address account,
454:         bool canExceedMax
455:     ) internal {
456:         if (canExceedMax) {
457:             require(
458:                 account.code.length != 0,
459:                 "ERC20Gauges: not a smart contract"
460:             );
461:         }
462: 
463:         canExceedMaxGauges[account] = canExceedMax; // @audit check `account` for address(0)
464:
465:         emit CanExceedMaxGaugesUpdate(account, canExceedMax);
466:     }

```
[452-466](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20Gauges.sol#L452C4-L466C6)

```solidity
File : src/tokens/ERC20RebaseDistributor.sol

293:        rebasingState[account] = RebasingState({ // @audit check `account` for address(0)
294:            isRebasing: 1,
295:            nShares: uint248(shares)
296:        });

```
[293-296](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20RebaseDistributor.sol#L293C1-L296C12)
</details> 

## [L-02] Check value for 0 before dividing

Division by zero is undefined in programming languages and can cause errors or unexpected behavior in your code.

**Note: Missed by bot-report**

_2 instances_

<details><summary>See Instances</summary>

Before performing the division operation `(totalBorrowedCredit * toleratedGaugeWeight) / totalWeight`, it's good practice to include a check to ensure `totalWeight` is not zero. If `totalWeight` could be zero in certain scenarios, you might want to add a condition to handle this case to avoid division by zero. 

```diff
File : src/loan/LendingTerm.sol

          uint256 totalWeight = GuildToken(_guildToken).totalTypeWeight(
              gaugeType
          );
          uint256 creditMinterBuffer = RateLimitedMinter(refs.creditMinter)
              .buffer();
          uint256 _hardCap = params.hardCap; // cached SLOAD
          if (gaugeWeight == 0) {
             return 0; // no gauge vote, 0 debt ceiling
          } else if (gaugeWeight == totalWeight) {
              // one gauge, unlimited debt ceiling
              // returns min(hardCap, creditMinterBuffer)
              return
                 _hardCap < creditMinterBuffer ? _hardCap : creditMinterBuffer;
          }
          
         uint256 _issuance = issuance; // cached SLOAD
          uint256 totalBorrowedCredit = ProfitManager(refs.profitManager)
             .totalBorrowedCredit();
          uint256 gaugeWeightTolerance = ProfitManager(refs.profitManager)
              .gaugeWeightTolerance();
         if (totalBorrowedCredit == 0 && gaugeWeight != 0) {
              // first-ever CREDIT mint on a non-zero gauge weight term
             // does not check the relative debt ceilings
              // returns min(hardCap, creditMinterBuffer)
              return
                  _hardCap < creditMinterBuffer ? _hardCap : creditMinterBuffer;
         }
          uint256 toleratedGaugeWeight = (gaugeWeight * gaugeWeightTolerance) /
             1e18;            
-          uint256 debtCeilingBefore = (totalBorrowedCredit *
-              toleratedGaugeWeight) / totalWeight;
+          if (totalWeight != 0) { 
+          uint256 debtCeilingBefore = (totalBorrowedCredit *
+              toleratedGaugeWeight) / totalWeight;
+         
+         }

```
[279-308](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L279C9-L308C49)

```diff
File : src/loan/LendingTerm.sol

        uint256 loanDebt = getLoanDebt(loanId);
        require(debtToRepay < loanDebt, "LendingTerm: full repayment");
-        uint256 percentRepaid = (debtToRepay * 1e18) / loanDebt; // [0, 1e18[
+       if(loanDebt != 0) {
+        uint256 percentRepaid = (debtToRepay * 1e18) / loanDebt; // [0, 1e18[
+      
+       }
```
[508-510](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L508C8-L510C78)
</details> 


# Non-Critical

## [N-01] Missing event emission for critical state changes

**Note: Missed by bot-report**

_11 instances_

<details><summary>See Instances</summary>

```solidity
File : src/core/CoreRef.sol

19:     constructor(address coreAddress) {
20:          _core = Core(coreAddress); // @audit NC - missing event emission
21:     }

```
[19-21](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/core/CoreRef.sol#L19C4-L21C6)

```solidity
File : src/governance/GuildVetoGovernor.sol

315:    function createVeto(bytes32 timelockId) external returns (uint256) {
316:         // Check that the operation is pending in the timelock
317:         uint256 timelockExecutionTime = TimelockController(payable(timelock))
318:            .getTimestamp(timelockId);
319:        require(
320:            timelockExecutionTime > 1,
321:            "GuildVetoGovernor: action must be pending"
322:        );
323:
324:        // Build proposal data
325:        (
326:            address[] memory targets,
327:            uint256[] memory values,
328:            bytes[] memory calldatas,
329:            string memory description
330:        ) = _getVetoCalls(timelockId);
331:
332:        uint256 proposalId = super.propose(
333:            targets,
334:            values,
335:            calldatas,
336:            description
337:        );
338:
339:        // Save mapping between proposalId and timelockId
340:        _timelockIds[proposalId] = timelockId; // @audit NC - missing event emission
341:        
342:        return proposalId;
343:     }

```
[315-343](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildVetoGovernor.sol#L315C5-L343C6)

```solidity
File : src/governance/LendingTermOnboarding.sol

181:    function proposeOnboard(
182:          address term
183:      ) external whenNotPaused returns (uint256 proposalId) {
184:          // Check that the term has been created by this factory
185:         require(created[term] != 0, "LendingTermOnboarding: invalid term");
186:
187:         // Check that the term was not subject to an onboard vote recently
188:         require(
189:            lastProposal[term] + MIN_DELAY_BETWEEN_PROPOSALS < block.timestamp,
190:            "LendingTermOnboarding: recently proposed"
191:        );
192:        lastProposal[term] = block.timestamp; // @audit NC - missing event emission
193:
194:        // Check that the term is not already active
195:        // note that terms that have been offboarded in the past can be re-onboarded
196:        // and won't fail this check. This is intentional, because some terms might be offboarded
197:        // due to specific market conditions, and it might be desirable to re-onboard them
198:        // at a later date.
199:        bool isGauge = GuildToken(guildToken).isGauge(term);
200:        require(!isGauge, "LendingTermOnboarding: active term");
201:
202:        // Generate calldata for the proposal
203:        (
204:            address[] memory targets,
205:            uint256[] memory values,
206:            bytes[] memory calldatas,
207:            string memory description
208:        ) = getOnboardProposeArgs(term);
209:
210:        // propose
211:        return Governor.propose(targets, values, calldatas, description); 
212:    }

```
[181-212](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOnboarding.sol#L181C5-L212C6)

```solidity
File : src/governance/ProfitManager.sol

156:    function initializeReferences(
157:        address _credit,
158:        address _guild,
159:        address _psm
160:    ) external onlyCoreRole(CoreRoles.GOVERNOR) {
161:        assert(
162:            credit == address(0) && guild == address(0) && psm == address(0)
163:        );
164:        credit = _credit;
165:        guild = _guild;
166:        psm = _psm;
167:    } // @audit NC - missing event emission

```
[156-167](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/ProfitManager.sol#L156C1-L167C6)

```solidity
File : src/loan/LendingTerm.sol

160:     function initialize(
161:         address _core,
162:         LendingTermReferences calldata _refs,
163:         LendingTermParams calldata _params
164:     ) external {
165:         // can initialize only once
166:         assert(address(core()) == address(0));
167:     assert(_core != address(0));
168:
169:         // initialize storage
170:         _setCore(_core);
171:         refs = _refs;
172:         params = _params;
173:     } // @audit NC - missing event emission

```
[160-173](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L160C1-L173C6)

```solidity
File : src/loan/SurplusGuildMinter.sol

88:    constructor(
89:        address _core,
90:        address _profitManager,
91:        address _credit,
92:        address _guild,
93:        address _rlgm,
94:        uint256 _mintRatio,
95:        uint256 _rewardRatio
96:    ) CoreRef(_core) {
97:        profitManager = _profitManager;
98:        credit = _credit;
99:        guild = _guild;
100:       rlgm = _rlgm;
101:       mintRatio = _mintRatio;
102:       rewardRatio = _rewardRatio;
103:   } // @audit NC - missing event emission

```
[88-103](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/SurplusGuildMinter.sol#L88C1-L103C6)

```solidity
File : src/tokens/ERC20Gauges.sol

249:   function _incrementUserAndGlobalWeights(
250:         address user,
251:         uint256 weight
252:     ) internal returns (uint256 newUserWeight) {
253:         newUserWeight = getUserWeight[user] + weight;
254:         // Ensure under weight
255:         require(newUserWeight <= balanceOf(user), "ERC20Gauges: overweight");
256: 
257:         // Update gauge state
258:         getUserWeight[user] = newUserWeight;
259:
260:         totalWeight += weight;
261:     } // @audit NC - missing event emission

301:    function decrementGauge(
302:         address gauge,
303:         uint256 weight
304:     ) public virtual returns (uint256 newUserWeight) {
305:         // All operations will revert on underflow, protecting against bad inputs
306:         _decrementGaugeWeight(msg.sender, gauge, weight);
307:         if (!_deprecatedGauges.contains(gauge)) {
308:             totalTypeWeight[gaugeType[gauge]] -= weight;
309:             totalWeight -= weight;
310:         }
311:         return getUserWeight[msg.sender];
312:     } // @audit NC - missing event emission

340:     function decrementGauges(
341:         address[] calldata gaugeList,
342:         uint256[] calldata weights
343:     ) public virtual returns (uint256 newUserWeight) {
344:         uint256 size = gaugeList.length;
345:         require(weights.length == size, "ERC20Gauges: size mismatch");
346:
347:         // store total in summary for batch update on user/global state
348:         uint256 weightsSum;
349:
350:         // Update gauge specific state
351:         // All operations will revert on underflow, protecting against bad inputs
352:         for (uint256 i = 0; i < size; ) {
353:             address gauge = gaugeList[i];
354:             uint256 weight = weights[i];
355:
356:             _decrementGaugeWeight(msg.sender, gauge, weight);
357:             if (!_deprecatedGauges.contains(gauge)) {
358:                 totalTypeWeight[gaugeType[gauge]] -= weight;
359:                 weightsSum += weight;
360:             }
361:             unchecked {
362:                 ++i;
363:             }
364:         }
365:         totalWeight -= weightsSum;
366:         return getUserWeight[msg.sender];
367:     } // @audit NC - missing event emission

```
[249-261](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20Gauges.sol#L249C4-L261C6), [301-312](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20Gauges.sol#L301C1-L312C6), [340-367](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20Gauges.sol#L340C5-L367C6)

```solidity
File : src/tokens/ERC20Gauges.sol

500:    function _decrementWeightUntilFree(address user, uint256 weight) internal {
501:        uint256 userFreeWeight = balanceOf(user) - getUserWeight[user];
502:
503:        // early return if already free
504:        if (userFreeWeight >= weight) return;
505:
506:        // cache totals for batch updates
507:        uint256 userFreed;
508:        uint256 totalFreed;
509:
510:        // Loop through all user gauges, live and deprecated
511:        address[] memory gaugeList = _userGauges[user].values();
512:
513:        // Free gauges until through entire list or under weight
514:        uint256 size = gaugeList.length;
515:        for (
516:            uint256 i = 0;
517:            i < size && (userFreeWeight + userFreed) < weight;
518:
519:        ) {
520:            address gauge = gaugeList[i];
521:            uint256 userGaugeWeight = getUserGaugeWeight[user][gauge];
522:            if (userGaugeWeight != 0) {
523:                userFreed += userGaugeWeight;
524:                _decrementGaugeWeight(user, gauge, userGaugeWeight);
525:
526:                // If the gauge is live (not deprecated), include its weight in the total to remove
527:                if (!_deprecatedGauges.contains(gauge)) {
528:                    totalTypeWeight[gaugeType[gauge]] -= userGaugeWeight;
529:                    totalFreed += userGaugeWeight;
530:                }
531:
532:                unchecked {
533:                    ++i;
534:                }
535:            }
536:        }
537:
538:        totalWeight -= totalFreed;
539:    } // @audit NC - missing event emission
540:   }

```
[500-540](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20Gauges.sol#L500C4-L540C2)

```solidity
File : src/utils/RateLimitedV2.sol

40:    constructor(
41:        uint256 _maxRateLimitPerSecond,
42:        uint128 _rateLimitPerSecond,
43:        uint128 _bufferCap
44:    ) {
45:        lastBufferUsedTime = block.timestamp.safeCastTo32();
46:
47:        _setBufferCap(_bufferCap);
48:        bufferStored = _bufferCap;
49:
50:        require(
51:            _rateLimitPerSecond <= _maxRateLimitPerSecond,
52:            "RateLimited: rateLimitPerSecond too high"
53:        );
54:        _setRateLimitPerSecond(_rateLimitPerSecond);
55:
56:        MAX_RATE_LIMIT_PER_SECOND = _maxRateLimitPerSecond;
57:    } // @audit NC - missing event emission

```
[40-57](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/utils/RateLimitedV2.sol#L40C1-L57C6)
</details> 

## [N-02] Use an efficient logic in setter functions

When intending to emit both the old and new values, there isn't a need to cache the old value that will only be used once. Simply emit both values before assigning a new value to the state variable.

_3 instances_

<details><summary>See Instances</summary>

```solidity
File : src/core/CoreRef.sol

49:     function _setCore(address newCore) internal {
50:          address oldCore = address(_core);
51:          _core = Core(newCore);
52:
53:          emit CoreUpdate(oldCore, newCore);
54:     }

```
[49-54](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/core/CoreRef.sol#L49C5-L54C6)

```diff
File : src/core/CoreRef.sol

      function _setCore(address newCore) internal {
-          address oldCore = address(_core);
+          emit CoreUpdate(address(_core), newCore);
           _core = Core(newCore);

-          emit CoreUpdate(oldCore, newCore);
      }

```

```solidity
File : src/tokens/ERC20Gauges.sol

444:    function _setMaxGauges(uint256 newMax) internal {
445:        uint256 oldMax = maxGauges;
446:        maxGauges = newMax;
447:
448:        emit MaxGaugesUpdate(oldMax, newMax);
449:    }

```
[444-449](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20Gauges.sol#L444C1-L449C6)

```diff
File : src/tokens/ERC20Gauges.sol

    function _setMaxGauges(uint256 newMax) internal {
-        uint256 oldMax = maxGauges;
+        emit MaxGaugesUpdate(maxGauges, newMax);
         maxGauges = newMax;

-        emit MaxGaugesUpdate(oldMax, newMax);
    }

```

```solidity
File : src/tokens/ERC20MultiVotes.sol

147:    function _setMaxDelegates(uint256 newMax) internal {
148:        uint256 oldMax = maxDelegates;
149:        maxDelegates = newMax;
150:
151:        emit MaxDelegatesUpdate(oldMax, newMax);
152:     }

```
[147-152](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20MultiVotes.sol#L147C1-L152C6)

```diff
File : src/tokens/ERC20MultiVotes.sol

    function _setMaxDelegates(uint256 newMax) internal {
-        uint256 oldMax = maxDelegates;
+        emit MaxDelegatesUpdate(maxDelegates, newMax);
         maxDelegates = newMax;

-        emit MaxDelegatesUpdate(oldMax, newMax);
    }

```
</details> 

## [N-03] In Solidity, following a style guide is important for readability and maintainability

**Note : Missed by bot-report**

_3 instances_

<details><summary>See Instances</summary>

struct `call` should be declare starting of the contract.
```solidity
File : src/core/CoreRef.sol

74:    struct Call {
75:          /// @notice target address to call
76:          address target;
77:          /// @notice amount of eth to send with the call
78:          uint256 value;
79:          /// @notice payload to send to target
80:          bytes callData;
81:     }

```
[74-81](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/core/CoreRef.sol#L74C4-L81C6)

mapping `canExceedMaxGauges` should be declare starting of the contract
```solidity
File : src/tokens/ERC20Gauges.sol

393:    mapping(address => bool) public canExceedMaxGauges;

```
[393](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20Gauges.sol#L393C4-L393C56)

event `TransfersEnabled` should be declare starting of the contract
```solidity
File : src/tokens/GuildToken.sol

172:   event TransfersEnabled(uint256 block, uint256 timestamp);

```
[172](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/GuildToken.sol#L172C5-L172C62)
</details> 

## [N-04] Stick with one way of using curly-brackets for single-instruction if conditions

When if condition wants to execute a single instruction, that instruction does not need to be inside {}. 
It's a good coding practice to stick to one way of if-syntax.

_21 instances_

<details><summary>See Instances</summary>

E.g., those two syntax are valid:
```
if (CONDITION) INSTRUCTION;

if (CONDITION) {
  INSTRUCTION;

}
```

```solidity
File : src/governance/GuildVetoGovernor.sol

258:        if (status == ProposalState.Executed) {
259:            return ProposalState.Executed;
260:        }

263:        if (status == ProposalState.Pending) {
264:            return ProposalState.Pending;
265         }

274:        if (timelockOperationTimestamp == 0) {
275:            return ProposalState.Canceled;
276:        }

278:        if (timelockOperationTimestamp == 1) {
279:            return ProposalState.Defeated;
280:        }

```
[258-260](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildVetoGovernor.sol#L258C1-L260C10), [263-265](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildVetoGovernor.sol#L263C1-L265C10), [274-276](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildVetoGovernor.sol#L274C8-L276C10), [278-280](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildVetoGovernor.sol#L278C9-L280C10)

```solidity
File : src/governance/ProfitManager.sol

472:        if (_gaugeProfitIndex == 0) {
473:             _gaugeProfitIndex = 1e18;
474:        }
475:        if (_userGaugeProfitIndex == 0) {
476:            _userGaugeProfitIndex = 1e18;
477:        }

```
[472-477](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/ProfitManager.sol#L472C12-L477C14)

```solidity
File : src/loan/LendingTerm.sol

204:        if (borrowTime == 0) {
205:            return 0;
206:        }
207:
208:        if (loan.closeTime != 0) {
209:            return 0;
210:        }
211:
212:        if (loan.callTime != 0) {
213:            return loan.callDebt;
214:        }

225:        if (_openingFee != 0) {
226:            loanDebt += (borrowAmount * _openingFee) / 1e18;
227:        }

298:        if (totalBorrowedCredit == 0 && gaugeWeight != 0) {
299:            // first-ever CREDIT mint on a non-zero gauge weight term
300:            // does not check the relative debt ceilings
301:            // returns min(hardCap, creditMinterBuffer)
302:            return
303:                _hardCap < creditMinterBuffer ? _hardCap : creditMinterBuffer;
304:        }

309:        if (_issuance >= debtCeilingBefore) {
310:            return debtCeilingBefore; // no more borrows allowed
311:        }

313:        if (toleratedGaugeWeight >= totalWeight) {
314:            // if the gauge weight is above 100% when we include tolerance,
315:            // the gauge relative debt ceilings are not constraining.
316:            return
317:                _hardCap < creditMinterBuffer ? _hardCap : creditMinterBuffer;
318:        }

324:         if (creditMinterBuffer < _debtCeiling) {
325:            return creditMinterBuffer;
326:         }
327:         if (_hardCap < _debtCeiling) {
328:            return _hardCap;
329:         }

413:         if (params.maxDelayBetweenPartialRepay != 0) {
414:            lastPartialRepay[loanId] = block.timestamp;
415:         }

```
[204-214](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L204C8-L214C10), [225-227](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L225C1-L227C10), [298-304](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L298C1-L304C10), [309-311](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L309C1-L311C10), [313-318](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L313C8-L318C10), [324-329](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L324C8-L329C10), [413-415](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L413C8-L415C10)

```solidity
File : src/loan/SurplusGuildMinter.sol

229:         if (lastGaugeLoss > uint256(userStake.lastGaugeLoss)) {
230:             slashed = true;
231:         }

262:         if (creditReward != 0) {
263:             CreditToken(credit).transfer(user, creditReward);
264:         }

             if (updateState) {
288:            _stakes[user][term] = userStake;
289:         }

```
[229-231](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/SurplusGuildMinter.sol#L229C1-L231C10), [262-264](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/SurplusGuildMinter.sol#L262C1-L264C14), [287-289](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/SurplusGuildMinter.sol#L287C1-L289C10)

```solidity
File : src/tokens/ERC20RebaseDistributor.sol

729:        if (
730:            rebasingStateFrom.isRebasing == 1 || rebasingStateTo.isRebasing == 1
731:        ) {
732:            updateTotalRebasingShares(_rebasingSharePrice, sharesDelta);
733:        }

```
[729-733](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20RebaseDistributor.sol#L729C1-L733C10)

```solidity
File : src/utils/RateLimitedV2.sol

117:        if (newBuffer == _bufferCap) {
118:            /// save an SSTORE + some stack operations if buffer cannot be increased.
119:            /// last buffer used time doesn't need to be updated as buffer cannot
120:            /// increase past the buffer cap
121:            return;
122:        }

```
[117-122](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/utils/RateLimitedV2.sol#L117C1-L122C10)
</details> 

## [N-05] Unnecessary Type Casting

Avoiding unnecessary type casting improves code readability and can make the codebase more maintainable by reducing unnecessary complexity.

_7 instances_

<details><summary>See Instances</summary>

Here `surplusBufferSplit`, `guildSplit` and `otherSplit` are type of `uint256`. 
```solidity
File : src/governance/ProfitManager.sol

212:    profitSharingConfig = ProfitSharingConfig({
213:            surplusBufferSplit: uint32(surplusBufferSplit / 1e9),
214:            guildSplit: uint32(guildSplit / 1e9),
215:            otherSplit: uint32(otherSplit / 1e9),
216:            otherRecipient: otherRecipient
217:        });

```
[212-217](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/ProfitManager.sol#L212C8-L217C12)

In this function `_setRateLimitPerSecond`  `rateLimitPerSecond` is type of `uint128`, `oldRateLimitPerSecond` is type of `uint256` and event `RateLimitPerSecondUpdate(uint256, uint256)`. Here `uint128` is assigning in `uint256` and `RateLimitPerSecondUpdate(uint256, uint128)` is emitting.
```solidity
File : src/utils/RateLimitedV2.sol

138:     function _setRateLimitPerSecond(uint128 newRateLimitPerSecond) internal {
139:        uint256 oldRateLimitPerSecond = rateLimitPerSecond;
140:        rateLimitPerSecond = newRateLimitPerSecond;
141:
142:        emit RateLimitPerSecondUpdate(
143:            oldRateLimitPerSecond,
144:            newRateLimitPerSecond
145:        );
146:     }

```
[138-146](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/utils/RateLimitedV2.sol#L138C1-L146C6)

## Recommendation

```diff
File : src/utils/RateLimitedV2.sol

      function _setRateLimitPerSecond(uint128 newRateLimitPerSecond) internal {
-        uint256 oldRateLimitPerSecond = rateLimitPerSecond;
+        emit RateLimitPerSecondUpdate(
+            uint256(rateLimitPerSecond),
+            uint256(newRateLimitPerSecond)
+        );
         rateLimitPerSecond = newRateLimitPerSecond;

-        emit RateLimitPerSecondUpdate(
-            oldRateLimitPerSecond,
-            newRateLimitPerSecond
-        );
    }

```

In this function `_setBufferCap`  `bufferCap` is type of `uint128`, `oldBufferCap` is type of `uint256` and event `BufferCapUpdate(uint256, uint256)`. Here `uint128` is assigning in `uint256` and `BufferCapUpdate(uint256, uint128)` is emitting. 
```solidity
File : src/utils/RateLimitedV2.sol

149:    function _setBufferCap(uint128 newBufferCap) internal {
150:        _updateBufferStored(newBufferCap);
151:
152:        uint256 oldBufferCap = bufferCap;
153:        bufferCap = newBufferCap;
154:
155:        emit BufferCapUpdate(oldBufferCap, newBufferCap);
156:    }

```
[149-156](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/utils/RateLimitedV2.sol#L149C1-L156C6)

## Recommendation

```diff
File : src/utils/RateLimitedV2.sol

    function _setBufferCap(uint128 newBufferCap) internal {
        _updateBufferStored(newBufferCap);

-        uint256 oldBufferCap = bufferCap;
+        emit BufferCapUpdate(uint256(bufferCap), uint256(newBufferCap));
         bufferCap = newBufferCap;

-        emit BufferCapUpdate(oldBufferCap, newBufferCap);
    }

```
</details> 

## [N-06] Typos 

**Note: Missed by bot-report**

_2 instances_

```diff
File : src/loan/AuctionHouse.sol

-      /// @notice emitted when au action starts 
+      /// @notice emitted when auction starts 

-      /// @notice emitted when au anction ends 
+      /// @notice emitted when auction ends 

```
[11](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/AuctionHouse.sol#L11C5-L11C46), [19](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/AuctionHouse.sol#L19C5-L19C45)

## [N-07] Use already cached value 

**Note: Missed by bot-report**

_1 instance_

```diff
File : src/loan/LendingTerm.sol

         uint256 borrowTime = loan.borrowTime;
-        require(loan.borrowTime != 0, "LendingTerm: loan not found");
+        require(borrowTime != 0, "LendingTerm: loan not found");

```
[642-643](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L642C4-L643C70)

## [N-08] Named return is confusing 

**Note: Missed by bot-report**

_3 instances_

```solidity
File : src/loan/SurplusGuildMinter.sol

216:    function getRewards(
217:        address user,
218:        address term
219:    )
220:        public
221:        returns (
222:            uint256 lastGaugeLoss, // GuildToken.lastGaugeLoss(term)
223:            UserStake memory userStake, // stake state after execution of getRewards()
224:            bool slashed // true if the user has been slashed
225:        )

```
[216-225](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/SurplusGuildMinter.sol#L216C1-L225C10)