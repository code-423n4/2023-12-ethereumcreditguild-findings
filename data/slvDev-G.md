## Gas Findings

Initial gas report was saved with `forge snapshot`.

Total gas saved per issue represent `Overall gas change` from `forge snapshot --diff`.

More details from `forge snapshot --diff` can be found in Gas Findings Details.


|    | Issue | Instances | Total Gas Saved |
|----|-------|:---------:|:---------:|
| [G-01] | `do`-`while` is cheaper than `for`-loops when the initial check can be skipped | 4 | 50034 |
| [G-02] | Using `require()` instead of `assert()` safes gas | 5 | 16370 |
| [G-03] | Use `assembly` to write mutable storage values | 103 | 430 (test one case) |
| [G-04] | Optimize Unsigned Integer Comparison With Zero | 1 | 815 |
| [G-05] | Reduce deployment costs by tweaking contracts' metadata | 20 | 212000 |
| [G-06] | Optimize `require/revert` Statements with Assembly | 119 | 35700 |


## Gas Findings Details

### [G-01] `do`-`while` is cheaper than `for`-loops when the initial check can be skipped

Using `do-while` loops instead of `for` loops can be more gas-efficient. 
Even if you add an `if` condition to account for the case where the loop doesn't execute at all, a `do-while` loop can still be cheaper in terms of gas.

It's critical to make shure that initial check can be skipped, otherwise the loop will be executed at least once, which can lead to unexpected results.

Example:
```solidity
/// 774 gas cost
function forLoop() public pure {
    for (uint256 i; i < 10;) {
        unchecked {
            ++i;
        }
    }
}
/// 519 gas cost
function doWhileLoop() public pure {
    uint256 i;
    do {
        unchecked {
            ++i;
        }
    } while (i < 10);
}
```

`forge snapshot --diff` output:
```bash
    testStake() (gas: -29 (-0.005%)) 
    testGetPendingRewards() (gas: -100 (-0.006%)) 
    testProfitDistribution() (gas: -252 (-0.013%)) 
    testUpdateMintRatio() (gas: -72 (-0.014%)) 
    testCallManySuccess() (gas: -103 (-0.016%)) 
    testUnstakeWithLoss() (gas: -128 (-0.018%)) 
    testUnstakeWithoutLoss() (gas: -186 (-0.027%)) 
    testEmergencyActionReverting() (gas: -25 (-0.106%)) 
    testEmergencyActionSucceedsGovernor(uint256) (gas: -96 (-0.128%)) 
    testEmergencyActionSucceedsGovernorSendsEth(uint128) (gas: -96 (-0.148%)) 
    testBorrowWithOpeningFeeSuccess() (gas: -9224 (-0.207%)) 
    testInitialize() (gas: -9224 (-0.249%)) 
    testEmergencyActionSucceedsGovernorSendEth(uint128) (gas: -96 (-0.350%)) 
    testInitializeReferences() (gas: -9429 (-0.389%)) 
    testMultipleUsers() (gas: -20974 (-0.812%)) 
    Overall gas change: -50034 (-0.047%)
```

<details>
<summary><i>4 issue instances in 3 files:</i></summary>

```solidity
File: src/core/CoreRef.sol

96: for (uint256 i = 0; i < calls.length; i++) {
            address payable target = payable(calls[i].target);
            uint256 value = calls[i].value;
            bytes calldata callData = calls[i].callData;

            (bool success, bytes memory returned) = target.call{value: value}(
                callData
            );
            require(success, "CoreRef: underlying call reverted");
            returnData[i] = returned;
        }
```
[96](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/core/CoreRef.sol#L96)

```solidity
File: src/governance/ProfitManager.sol

443: for (uint256 i = 0; i < gauges.length; ) {
            creditEarned += claimGaugeRewards(user, gauges[i]);
            unchecked {
                ++i;
            }
        }
467: for (uint256 i = 0; i < gauges.length; ) {
            address gauge = gauges[i];
            uint256 _gaugeProfitIndex = gaugeProfitIndex[gauge];
            uint256 _userGaugeProfitIndex = userGaugeProfitIndex[user][gauge];

            if (_gaugeProfitIndex == 0) {
                _gaugeProfitIndex = 1e18;
            }
            if (_userGaugeProfitIndex == 0) {
                _userGaugeProfitIndex = 1e18;
            }
            uint256 deltaIndex = _gaugeProfitIndex - _userGaugeProfitIndex;
            if (deltaIndex != 0) {
                uint256 _userGaugeWeight = uint256(
                    GuildToken(_guild).getUserGaugeWeight(user, gauge)
                );
                creditEarned[i] = (_userGaugeWeight * deltaIndex) / 1e18;
                totalCreditEarned += creditEarned[i];
            }

            unchecked {
                ++i;
            }
        }
```
[443](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/ProfitManager.sol#L443) | [467](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/ProfitManager.sol#L467)

```solidity
File: src/loan/LendingTerm.sol

685: for (uint256 i = 0; i < loanIds.length; i++) {
            _call(msg.sender, loanIds[i], _auctionHouse);
        }
```
[685](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/loan/LendingTerm.sol#L685)
</details>

### [G-02] Using `require()` instead of `assert()` safes gas

Using `require()` saves 33 gas per call.
```solidity
    function test() public payable {
        // require(false); // 133 gas
        // assert(false); // 166 gas 
    }
```

`forge snapshot --diff` output:
```bash
    testBorrowWithOpeningFeeSuccess() (gas: -5616 (-0.126%)) 
    testInitialize() (gas: -5700 (-0.154%)) 
    testInitializeReferences() (gas: -5054 (-0.208%)) 
    Overall gas change: -16370 (-0.015%)
```

<details>
<summary><i>5 issue instances in 4 files:</i></summary>

```solidity
File: src/governance/ProfitManager.sol

161: assert(
            credit == address(0) && guild == address(0) && psm == address(0)
        )
```
[161](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/ProfitManager.sol#L161)

```solidity
File: src/governance/GuildVetoGovernor.sol

255: assert(queueid != bytes32(0))
```
[255](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/GuildVetoGovernor.sol#L255)

```solidity
File: src/loan/AuctionHouse.sol

131: assert(block.timestamp >= _startTime)
```
[131](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/loan/AuctionHouse.sol#L131)

```solidity
File: src/loan/LendingTerm.sol

166: assert(address(core()) == address(0))
167: assert(_core != address(0))
```
[166](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/loan/LendingTerm.sol#L166) | [167](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/loan/LendingTerm.sol#L167)
</details>

### [G-03] Use `assembly` to write mutable storage values

Writing to storage using `assembly` is more gas efficient.

Change one case for test:
```diff
    function donateToSurplusBuffer(uint256 amount) external {
        CreditToken(credit).transferFrom(msg.sender, address(this), amount);
        uint256 newSurplusBuffer = surplusBuffer + amount;
-       surplusBuffer = newSurplusBuffer;
+       assembly {
+           sstore(surplusBuffer.slot, newSurplusBuffer)
+       }
        emit SurplusBufferUpdate(block.timestamp, newSurplusBuffer);
    }
```

`forge snapshot --diff` output:
```bash
    testDepleteTermSurplusBuffer() (gas: -5 (-0.001%)) 
    testDepleteSurplusBuffer() (gas: -6 (-0.002%)) 
    testWithdrawFromSurplusBuffer() (gas: -6 (-0.002%)) 
    testDonateToSurplusBuffer() (gas: -6 (-0.004%)) 
    testInitializeReferences() (gas: -407 (-0.017%)) 
    Overall gas change: -430 (-0.000%)
```

<details>
<summary><i>103 issue instances in 15 files:</i></summary>

```solidity
File: src/core/CoreRef.sol

20: _core = Core(coreAddress)
51: _core = Core(newCore)
```
[20](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/core/CoreRef.sol#L20) | [51](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/core/CoreRef.sol#L51)

```solidity
File: src/governance/GuildGovernor.sol

70: _quorum = newQuorum
```
[70](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/GuildGovernor.sol#L70)

```solidity
File: src/governance/LendingTermOffboarding.sol

74: quorum = _quorum
82: quorum = _quorum
104: polls[block.number][term] = 1
105: lastPollBlock[term] = block.number
136: userPollVotes[msg.sender][snapshotBlock][term] = userWeight
137: polls[snapshotBlock][term] = _weight + userWeight
139: canOffboard[term] = true
197: canOffboard[term] = false
```
[74](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/LendingTermOffboarding.sol#L74) | [82](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/LendingTermOffboarding.sol#L82) | [104](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/LendingTermOffboarding.sol#L104) | [105](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/LendingTermOffboarding.sol#L105) | [136](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/LendingTermOffboarding.sol#L136) | [137](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/LendingTermOffboarding.sol#L137) | [139](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/LendingTermOffboarding.sol#L139) | [197](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/LendingTermOffboarding.sol#L197)

```solidity
File: src/governance/LendingTermOnboarding.sol

96: implementations[implementation] = allowed
165: created[term] = block.timestamp
192: lastProposal[term] = block.timestamp
```
[96](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/LendingTermOnboarding.sol#L96) | [165](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/LendingTermOnboarding.sol#L165) | [192](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/LendingTermOnboarding.sol#L192)

```solidity
File: src/governance/ProfitManager.sol

164: credit = _credit
165: guild = _guild
166: psm = _psm
182: _minBorrow = newValue
190: gaugeWeightTolerance = newValue
212: profitSharingConfig = ProfitSharingConfig({
            surplusBufferSplit: uint32(surplusBufferSplit / 1e9),
            guildSplit: uint32(guildSplit / 1e9),
            otherSplit: uint32(otherSplit / 1e9),
            otherRecipient: otherRecipient
        })
254: surplusBuffer = newSurplusBuffer
262: termSurplusBuffer[term] = newSurplusBuffer
272: surplusBuffer = newSurplusBuffer
284: termSurplusBuffer[term] = newSurplusBuffer
310: termSurplusBuffer[gauge] = 0
317: surplusBuffer = _surplusBuffer - loss
326: surplusBuffer = 0
334: creditMultiplier = newCreditMultiplier
362: surplusBuffer = _surplusBuffer + amountForSurplusBuffer
396: gaugeProfitIndex[gauge] =
                        _gaugeProfitIndex +
                        (amountForGuild * 1e18) /
                        _gaugeWeight
430: userGaugeProfitIndex[user][gauge] = _gaugeProfitIndex
```
[164](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/ProfitManager.sol#L164) | [165](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/ProfitManager.sol#L165) | [166](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/ProfitManager.sol#L166) | [182](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/ProfitManager.sol#L182) | [190](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/ProfitManager.sol#L190) | [212](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/ProfitManager.sol#L212) | [254](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/ProfitManager.sol#L254) | [262](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/ProfitManager.sol#L262) | [272](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/ProfitManager.sol#L272) | [284](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/ProfitManager.sol#L284) | [310](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/ProfitManager.sol#L310) | [317](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/ProfitManager.sol#L317) | [326](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/ProfitManager.sol#L326) | [334](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/ProfitManager.sol#L334) | [362](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/ProfitManager.sol#L362) | [396](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/ProfitManager.sol#L396) | [430](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/ProfitManager.sol#L430)

```solidity
File: src/governance/GuildVetoGovernor.sol

73: _quorum = newQuorum
101: timelock = newTimelock
340: _timelockIds[proposalId] = timelockId
```
[73](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/GuildVetoGovernor.sol#L73) | [101](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/GuildVetoGovernor.sol#L101) | [340](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/GuildVetoGovernor.sol#L340)

```solidity
File: src/tokens/ERC20Gauges.sol

258: getUserWeight[user] = newUserWeight
321: getUserGaugeWeight[user][gauge] = oldWeight - weight
409: gaugeType[gauge] = _type
446: maxGauges = newMax
463: canExceedMaxGauges[account] = canExceedMax
```
[258](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/tokens/ERC20Gauges.sol#L258) | [321](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/tokens/ERC20Gauges.sol#L321) | [409](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/tokens/ERC20Gauges.sol#L409) | [446](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/tokens/ERC20Gauges.sol#L446) | [463](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/tokens/ERC20Gauges.sol#L463)

```solidity
File: src/tokens/ERC20MultiVotes.sol

149: maxDelegates = newMax
164: canContractExceedMaxDelegates[account] = canExceedMax
356: _delegatesVotesCount[delegator][delegatee] = newDelegates
453: _delegatesVotesCount[user][delegatee] = 0
```
[149](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/tokens/ERC20MultiVotes.sol#L149) | [164](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/tokens/ERC20MultiVotes.sol#L164) | [356](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/tokens/ERC20MultiVotes.sol#L356) | [453](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/tokens/ERC20MultiVotes.sol#L453)

```solidity
File: src/tokens/ERC20RebaseDistributor.sol

186: totalRebasingShares = sharesAfter
190: __rebasingSharePrice = InterpolatedValue({
                lastTimestamp: SafeCastLib.safeCastTo32(block.timestamp), // now
                lastValue: uint224(START_REBASING_SHARE_PRICE), // safe initial value
                targetTimestamp: SafeCastLib.safeCastTo32(block.timestamp), // now
                targetValue: uint224(START_REBASING_SHARE_PRICE) // safe initial value
            })
196: __unmintedRebaseRewards = InterpolatedValue({
                lastTimestamp: SafeCastLib.safeCastTo32(block.timestamp),
                lastValue: 0,
                targetTimestamp: SafeCastLib.safeCastTo32(block.timestamp),
                targetValue: 0
            })
218: __rebasingSharePrice = InterpolatedValue({
                lastTimestamp: SafeCastLib.safeCastTo32(block.timestamp), // now
                lastValue: SafeCastLib.safeCastTo224(currentRebasingSharePrice), // current value
                targetTimestamp: val.targetTimestamp, // unchanged
                targetValue: SafeCastLib.safeCastTo224(targetNewSharePrice) // adjusted target
            })
231: __unmintedRebaseRewards = InterpolatedValue({
            lastTimestamp: SafeCastLib.safeCastTo32(block.timestamp), // now
            lastValue: SafeCastLib.safeCastTo224(
                _unmintedRebaseRewards - amount
            ), // adjusted current
            targetTimestamp: val.targetTimestamp, // unchanged
            targetValue: val.targetValue - SafeCastLib.safeCastTo224(amount) // adjusted target
        })
293: rebasingState[account] = RebasingState({
            isRebasing: 1,
            nShares: uint248(shares)
        })
329: rebasingState[account] = RebasingState({isRebasing: 0, nShares: 0})
369: __rebasingSharePrice = InterpolatedValue({
                lastTimestamp: SafeCastLib.safeCastTo32(block.timestamp),
                lastValue: SafeCastLib.safeCastTo224(_rebasingSharePrice),
                targetTimestamp: SafeCastLib.safeCastTo32(endTimestamp),
                targetValue: SafeCastLib.safeCastTo224(newTargetSharePrice)
            })
378: __unmintedRebaseRewards = InterpolatedValue({
                lastTimestamp: SafeCastLib.safeCastTo32(block.timestamp),
                lastValue: SafeCastLib.safeCastTo224(_unmintedRebaseRewards),
                targetTimestamp: SafeCastLib.safeCastTo32(endTimestamp),
                targetValue: __unmintedRebaseRewards.targetValue +
                    SafeCastLib.safeCastTo224(amount)
            })
496: rebasingState[account] = RebasingState({
                isRebasing: 1,
                nShares: uint248(sharesAfter)
            })
532: rebasingState[account] = RebasingState({
                isRebasing: 1,
                nShares: uint248(sharesAfter)
            })
596: rebasingState[msg.sender] = RebasingState({
                isRebasing: 1,
                nShares: uint248(fromSharesAfter)
            })
620: rebasingState[to] = RebasingState({
                isRebasing: 1,
                nShares: uint248(toSharesAfter)
            })
690: rebasingState[from] = RebasingState({
                isRebasing: 1,
                nShares: uint248(fromSharesAfter)
            })
714: rebasingState[to] = RebasingState({
                isRebasing: 1,
                nShares: uint248(toSharesAfter)
            })
```
[186](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/tokens/ERC20RebaseDistributor.sol#L186) | [190](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/tokens/ERC20RebaseDistributor.sol#L190) | [196](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/tokens/ERC20RebaseDistributor.sol#L196) | [218](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/tokens/ERC20RebaseDistributor.sol#L218) | [231](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/tokens/ERC20RebaseDistributor.sol#L231) | [293](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/tokens/ERC20RebaseDistributor.sol#L293) | [329](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/tokens/ERC20RebaseDistributor.sol#L329) | [369](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/tokens/ERC20RebaseDistributor.sol#L369) | [378](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/tokens/ERC20RebaseDistributor.sol#L378) | [496](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/tokens/ERC20RebaseDistributor.sol#L496) | [532](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/tokens/ERC20RebaseDistributor.sol#L532) | [596](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/tokens/ERC20RebaseDistributor.sol#L596) | [620](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/tokens/ERC20RebaseDistributor.sol#L620) | [690](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/tokens/ERC20RebaseDistributor.sol#L690) | [714](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/tokens/ERC20RebaseDistributor.sol#L714)

```solidity
File: src/tokens/GuildToken.sol

51: profitManager = _profitManager
127: lastGaugeLoss[gauge] = block.timestamp
146: lastGaugeLossApplied[gauge][who] = block.timestamp
176: transferable = true
198: profitManager = _newProfitManager
250: lastGaugeLossApplied[gauge][user] = block.timestamp
```
[51](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/tokens/GuildToken.sol#L51) | [127](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/tokens/GuildToken.sol#L127) | [146](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/tokens/GuildToken.sol#L146) | [176](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/tokens/GuildToken.sol#L176) | [198](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/tokens/GuildToken.sol#L198) | [250](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/tokens/GuildToken.sol#L250)

```solidity
File: src/loan/AuctionHouse.sol

96: auctions[loanId] = Auction({
            startTime: block.timestamp,
            endTime: 0,
            lendingTerm: msg.sender,
            collateralAmount: loan.collateralAmount,
            callDebt: callDebt
        })
175: auctions[loanId].endTime = block.timestamp
209: auctions[loanId].endTime = block.timestamp
```
[96](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/loan/AuctionHouse.sol#L96) | [175](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/loan/AuctionHouse.sol#L175) | [209](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/loan/AuctionHouse.sol#L209)

```solidity
File: src/loan/LendingTerm.sol

171: refs = _refs
172: params = _params
401: loans[loanId] = Loan({
            borrower: borrower,
            borrowTime: block.timestamp,
            borrowAmount: borrowAmount,
            borrowCreditMultiplier: creditMultiplier,
            collateralAmount: collateralAmount,
            caller: address(0),
            callTime: 0,
            callDebt: 0,
            closeTime: 0
        })
412: issuance = _postBorrowIssuance
414: lastPartialRepay[loanId] = block.timestamp
535: lastPartialRepay[loanId] = block.timestamp
666: loans[loanId].callTime = block.timestamp
667: loans[loanId].callDebt = loanDebt
668: loans[loanId].caller = caller
705: loans[loanId].closeTime = block.timestamp
718: params.hardCap = 0
771: loans[loanId].closeTime = block.timestamp
841: refs.auctionHouse = _newValue
849: params.hardCap = _newValue
```
[171](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/loan/LendingTerm.sol#L171) | [172](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/loan/LendingTerm.sol#L172) | [401](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/loan/LendingTerm.sol#L401) | [412](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/loan/LendingTerm.sol#L412) | [414](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/loan/LendingTerm.sol#L414) | [535](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/loan/LendingTerm.sol#L535) | [666](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/loan/LendingTerm.sol#L666) | [667](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/loan/LendingTerm.sol#L667) | [668](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/loan/LendingTerm.sol#L668) | [705](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/loan/LendingTerm.sol#L705) | [718](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/loan/LendingTerm.sol#L718) | [771](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/loan/LendingTerm.sol#L771) | [841](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/loan/LendingTerm.sol#L841) | [849](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/loan/LendingTerm.sol#L849)

```solidity
File: src/loan/SimplePSM.sol

152: redemptionsPaused = paused
```
[152](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/loan/SimplePSM.sol#L152)

```solidity
File: src/loan/SurplusGuildMinter.sol

101: mintRatio = _mintRatio
102: rewardRatio = _rewardRatio
151: _stakes[msg.sender][term] = userStake
196: _stakes[msg.sender][term] = userStake
288: _stakes[user][term] = userStake
307: _stakes[user][term].guild = SafeCastLib.safeCastTo128(guildAfter)
313: _stakes[user][term].guild = SafeCastLib.safeCastTo128(guildAfter)
322: mintRatio = _mintRatio
331: rewardRatio = _rewardRatio
```
[101](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/loan/SurplusGuildMinter.sol#L101) | [102](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/loan/SurplusGuildMinter.sol#L102) | [151](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/loan/SurplusGuildMinter.sol#L151) | [196](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/loan/SurplusGuildMinter.sol#L196) | [288](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/loan/SurplusGuildMinter.sol#L288) | [307](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/loan/SurplusGuildMinter.sol#L307) | [313](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/loan/SurplusGuildMinter.sol#L313) | [322](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/loan/SurplusGuildMinter.sol#L322) | [331](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/loan/SurplusGuildMinter.sol#L331)

```solidity
File: src/utils/RateLimitedV2.sol

45: lastBufferUsedTime = block.timestamp.safeCastTo32()
48: bufferStored = _bufferCap
103: lastBufferUsedTime = blockTimestamp
104: bufferStored = newBufferStored
131: lastBufferUsedTime = blockTimestamp
132: bufferStored = newBufferStored
140: rateLimitPerSecond = newRateLimitPerSecond
153: bufferCap = newBufferCap
163: bufferStored = uint224(newBufferCap)
164: lastBufferUsedTime = newBlockTimestamp
166: bufferStored = newBufferStored
167: lastBufferUsedTime = newBlockTimestamp
```
[45](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/utils/RateLimitedV2.sol#L45) | [48](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/utils/RateLimitedV2.sol#L48) | [103](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/utils/RateLimitedV2.sol#L103) | [104](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/utils/RateLimitedV2.sol#L104) | [131](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/utils/RateLimitedV2.sol#L131) | [132](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/utils/RateLimitedV2.sol#L132) | [140](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/utils/RateLimitedV2.sol#L140) | [153](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/utils/RateLimitedV2.sol#L153) | [163](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/utils/RateLimitedV2.sol#L163) | [164](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/utils/RateLimitedV2.sol#L164) | [166](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/utils/RateLimitedV2.sol#L166) | [167](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/utils/RateLimitedV2.sol#L167)
</details>

### [G-04] Optimize Unsigned Integer Comparison With Zero

```diff
        }
        // handling profit
-       else if (amount > 0) {
+       else if (amount != 0) {
            ProfitSharingConfig
                memory _profitSharingConfig = profitSharingConfig;
```

`forge snapshot --diff` output:
```bash
    testGetPendingRewards() (gas: -6 (-0.000%)) 
    testMultipleUsers() (gas: -12 (-0.000%)) 
    testUnstakeWithLoss() (gas: -4 (-0.001%)) 
    testCleanup() (gas: -4 (-0.001%)) 
    testFlowBorrowCallOnBidGoodDebt() (gas: -6 (-0.001%)) 
    testProfitAccountingCallBidAfterMarkUp() (gas: -6 (-0.001%)) 
    testUnstakeWithoutLoss() (gas: -5 (-0.001%)) 
    testBidSuccessFuzz(uint256) (gas: -6 (-0.001%)) 
    testAddCollateralFailures() (gas: -4 (-0.001%)) 
    testBidFailAuctionConcluded() (gas: -6 (-0.001%)) 
    testGetBidDetailFailAuctionConcluded() (gas: -6 (-0.001%)) 
    testRepaySuccess(uint256) (gas: -4 (-0.001%)) 
    testOffboard() (gas: -6 (-0.001%)) 
    testCanBorrowMoreAfterCreditLoseValue() (gas: -5 (-0.001%)) 
    testCallFailLoanClosed() (gas: -5 (-0.001%)) 
    testActiveLoansAreMarkedUpWhenCreditLoseValue() (gas: -5 (-0.001%)) 
    testProfitAccountingPartialRepayAfterMarkUp() (gas: -6 (-0.001%)) 
    testCannotCleanupAfterReonboard() (gas: -6 (-0.001%)) 
    testProfitAccountingRepayAfterMarkUp() (gas: -6 (-0.001%)) 
    testFlowBorrowRepay() (gas: -5 (-0.001%)) 
    testRepayFailAlreadyClosed1() (gas: -5 (-0.001%)) 
    testMinBorrowAfterCreditLoseValue() (gas: -6 (-0.001%)) 
    testRepayFailRoleRevoked() (gas: -6 (-0.001%)) 
    testPartialRepaySuccess() (gas: -6 (-0.001%)) 
    testDepleteSurplusBuffer() (gas: -6 (-0.002%)) 
    testPartialRepayReverts() (gas: -10 (-0.002%)) 
    testProfitDistribution() (gas: -36 (-0.002%)) 
    testCreditMultiplier() (gas: -6 (-0.002%)) 
    testNotifyPnLLastGaugeLoss() (gas: -6 (-0.002%)) 
    testInitializeReferences() (gas: -615 (-0.025%)) 
    Overall gas change: -815 (-0.001%)
```

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/governance/ProfitManager.sol

342: else if (amount > 0) {
```
[342](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/ProfitManager.sol#L342)
</details>

### [G-05] Reduce deployment costs by tweaking contracts' metadata

The Solidity compiler appends 53 bytes of metadata to the smart contract code which translates to an extra 10,600 gas (200 per bytecode) + the calldata cost (16 gas per non-zero bytes, 4 gas per zero-byte).
This translates to up to 848 additional gas in calldata cost.
One way to reduce this cost is by optimizing the IPFS hash that gets appended to the smart contract code.

Why is this important?
- The metadata adds an extra 53 bytes, resulting in an additional 10,600 gas cost for deployment.
- It also incurs up to 848 additional gas in calldata cost.

Options to Reduce Gas:
1. Use the `--no-cbor-metadata` compiler option to exclude metadata, but this might affect contract verification.
2. Mine for code comments that lead to an IPFS hash with more zeros, reducing calldata costs.

<details>
<summary><i>20 issue instances in 20 files:</i></summary>

```solidity
File: All files

Consider optimizing the IPFS hash during deployment.
```
</details>


### [G-06] Optimize `require/revert` Statements with Assembly

Using inline assembly for revert statements in Solidity can offer gas optimizations.
The typical `require` or `revert` statements in Solidity perform additional memory operations and type checks which can be avoided by using low-level assembly code.

In certain contracts, particularly those that might revert often or are otherwise sensitive to gas costs, using assembly to handle reversion can offer meaningful savings.
These savings primarily come from avoiding memory expansion costs and extra type checks that the Solidity compiler performs.

Example:
```solidity
/// calling restrictedAction(2) with a non-owner address: 24042
function restrictedAction(uint256 num)  external {
    require(owner == msg.sender, "caller is not owner");
    specialNumber = num;
}

// calling restrictedAction(2) with a non-owner address: 23734
function restrictedAction(uint256 num)  external {
    assembly {
        if sub(caller(), sload(owner.slot)) {
            mstore(0x00, 0x20) // store offset to where length of revert message is stored
            mstore(0x20, 0x13) // store length (19)
            mstore(0x40, 0x63616c6c6572206973206e6f74206f776e657200000000000000000000000000) // store hex representation of message
            revert(0x00, 0x60) // revert with data
        }
    }
    specialNumber = num;
}
```

<details>
<summary><i>119 issue instances in 15 files:</i></summary>

```solidity
File: src/core/CoreRef.sol

25: require(_core.hasRole(role, msg.sender), "UNAUTHORIZED");
104: require(success, "CoreRef: underlying call reverted");
```
[25](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/core/CoreRef.sol#L25) | [104](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/core/CoreRef.sol#L104)

```solidity
File: src/governance/LendingTermOffboarding.sol

90: require(
            polls[block.number][term] == 0,
            "LendingTermOffboarding: poll exists"
        );
94: require(
            block.number > lastPollBlock[term] + POLL_DURATION_BLOCKS,
            "LendingTermOffboarding: poll active"
        );
99: require(
            GuildToken(guildToken).isGauge(term),
            "LendingTermOffboarding: not an active term"
        );
120: require(
            block.number <= snapshotBlock + POLL_DURATION_BLOCKS,
            "LendingTermOffboarding: poll expired"
        );
125: require(_weight != 0, "LendingTermOffboarding: poll not found");
130: require(userWeight != 0, "LendingTermOffboarding: zero weight");
131: require(
            userPollVotes[msg.sender][snapshotBlock][term] == 0,
            "LendingTermOffboarding: already voted"
        );
154: require(canOffboard[term], "LendingTermOffboarding: quorum not met");
176: require(canOffboard[term], "LendingTermOffboarding: quorum not met");
177: require(
            LendingTerm(term).issuance() == 0,
            "LendingTermOffboarding: not all loans closed"
        );
181: require(
            GuildToken(guildToken).isDeprecatedGauge(term),
            "LendingTermOffboarding: re-onboarded"
        );
```
[90](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOffboarding.sol#L90) | [94](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOffboarding.sol#L94) | [99](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOffboarding.sol#L99) | [120](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOffboarding.sol#L120) | [125](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOffboarding.sol#L125) | [130](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOffboarding.sol#L130) | [131](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOffboarding.sol#L131) | [154](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOffboarding.sol#L154) | [176](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOffboarding.sol#L176) | [177](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOffboarding.sol#L177) | [181](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOffboarding.sol#L181)

```solidity
File: src/governance/LendingTermOnboarding.sol

109: require(
            implementations[implementation],
            "LendingTermOnboarding: invalid implementation"
        );
117: require(
            success && returned.length == 32,
            "LendingTermOnboarding: invalid collateralToken"
        );
122: require(
            params.maxDebtPerCollateralToken != 0, // must be able to mint non-zero debt
            "LendingTermOnboarding: invalid maxDebtPerCollateralToken"
        );
127: require(
            params.interestRate < 1e18, // interest rate [0, 100[% APR
            "LendingTermOnboarding: invalid interestRate"
        );
132: require(
            // 31557601 comes from the constant LendingTerm.YEAR() + 1
            params.maxDelayBetweenPartialRepay < 31557601, // periodic payment every [0, 1 year]
            "LendingTermOnboarding: invalid maxDelayBetweenPartialRepay"
        );
138: require(
            params.minPartialRepayPercent < 1e18, // periodic payment sizes [0, 100[%
            "LendingTermOnboarding: invalid minPartialRepayPercent"
        );
143: require(
            params.openingFee <= 0.1e18, // open fee expected [0, 10]%
            "LendingTermOnboarding: invalid openingFee"
        );
148: require(
            params.hardCap != 0, // non-zero hardcap
            "LendingTermOnboarding: invalid hardCap"
        );
177: revert("LendingTermOnboarding: cannot propose arbitrary actions");
185: require(created[term] != 0, "LendingTermOnboarding: invalid term");
188: require(
            lastProposal[term] + MIN_DELAY_BETWEEN_PROPOSALS < block.timestamp,
            "LendingTermOnboarding: recently proposed"
        );
200: require(!isGauge, "LendingTermOnboarding: active term");
```
[109](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOnboarding.sol#L109) | [117](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOnboarding.sol#L117) | [122](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOnboarding.sol#L122) | [127](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOnboarding.sol#L127) | [132](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOnboarding.sol#L132) | [138](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOnboarding.sol#L138) | [143](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOnboarding.sol#L143) | [148](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOnboarding.sol#L148) | [177](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOnboarding.sol#L177) | [185](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOnboarding.sol#L185) | [188](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOnboarding.sol#L188) | [200](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOnboarding.sol#L200)

```solidity
File: src/governance/ProfitManager.sol

203: require(otherSplit == 0, "GuildToken: invalid config");
205: require(otherSplit != 0, "GuildToken: invalid config");
207: require(
            surplusBufferSplit + otherSplit + guildSplit + creditSplit == 1e18,
            "GuildToken: invalid config"
        );
```
[203](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/ProfitManager.sol#L203) | [205](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/ProfitManager.sol#L205) | [207](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/ProfitManager.sol#L207)

```solidity
File: src/governance/GuildVetoGovernor.sol

204: require(
            !proposalvote.hasVoted[account],
            "GuildVetoGovernor: vote already cast"
        );
213: revert("GuildVetoGovernor: can only vote against in veto proposals");
310: revert("GuildVetoGovernor: cannot propose arbitrary actions");
319: require(
            timelockExecutionTime > 1,
            "GuildVetoGovernor: action must be pending"
        );
```
[204](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildVetoGovernor.sol#L204) | [213](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildVetoGovernor.sol#L213) | [310](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildVetoGovernor.sol#L310) | [319](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildVetoGovernor.sol#L319)

```solidity
File: src/tokens/ERC20Gauges.sol

223: require(isGauge(gauge), "ERC20Gauges: invalid gauge");
237: require(canExceedMaxGauges[user], "ERC20Gauges: exceed max gauges");
255: require(newUserWeight <= balanceOf(user), "ERC20Gauges: overweight");
274: require(weights.length == size, "ERC20Gauges: size mismatch");
285: require(isGauge(gauge), "ERC20Gauges: invalid gauge");
324: require(_userGauges[user].remove(gauge));
345: require(weights.length == size, "ERC20Gauges: size mismatch");
402: require(
            gauge != address(0) && (newAdd || previouslyDeprecated),
            "ERC20Gauges: invalid gauge"
        );
412: require(gaugeType[gauge] == _type, "ERC20Gauges: invalid type");
427: require(
            _gauges.contains(gauge) && _deprecatedGauges.add(gauge),
            "ERC20Gauges: invalid gauge"
        );
457: require(
                account.code.length != 0,
                "ERC20Gauges: not a smart contract"
            );
```
[223](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20Gauges.sol#L223) | [237](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20Gauges.sol#L237) | [255](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20Gauges.sol#L255) | [274](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20Gauges.sol#L274) | [285](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20Gauges.sol#L285) | [324](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20Gauges.sol#L324) | [345](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20Gauges.sol#L345) | [402](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20Gauges.sol#L402) | [412](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20Gauges.sol#L412) | [427](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20Gauges.sol#L427) | [457](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20Gauges.sol#L457)

```solidity
File: src/tokens/ERC20MultiVotes.sol

95: require(
            blockNumber < block.number,
            "ERC20MultiVotes: not a past block"
        );
159: require(
            !canExceedMax || account.code.length != 0,
            "ERC20MultiVotes: not a smart contract"
        ); // can only approve contracts
297: require(count < 2, "ERC20MultiVotes: delegation error");
324: require(
            delegatee != address(0) && free >= amount,
            "ERC20MultiVotes: delegation error"
        );
330: require(
            !newDelegate ||
                delegateCount(delegator) <= maxDelegates ||
                canContractExceedMaxDelegates[delegator],
            "ERC20MultiVotes: delegation error"
        );
353: require(_delegates[delegator].remove(delegatee));
451: require(_delegates[user].remove(delegatee)); // Remove from set. Should never fail.
479: require(
            block.timestamp <= expiry,
            "ERC20MultiVotes: signature expired"
        );
502: require(nonce == _useNonce(signer), "ERC20MultiVotes: invalid nonce");
503: require(signer != address(0));
```
[95](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20MultiVotes.sol#L95) | [159](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20MultiVotes.sol#L159) | [297](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20MultiVotes.sol#L297) | [324](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20MultiVotes.sol#L324) | [330](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20MultiVotes.sol#L330) | [353](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20MultiVotes.sol#L353) | [451](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20MultiVotes.sol#L451) | [479](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20MultiVotes.sol#L479) | [502](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20MultiVotes.sol#L502) | [503](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20MultiVotes.sol#L503)

```solidity
File: src/tokens/ERC20RebaseDistributor.sol

282: require(
            rebasingState[msg.sender].isRebasing == 0,
            "ERC20RebaseDistributor: already rebasing"
        );
304: require(
            rebasingState[msg.sender].isRebasing == 1,
            "ERC20RebaseDistributor: not rebasing"
        );
339: require(amount != 0, "ERC20RebaseDistributor: cannot distribute zero");
```
[282](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20RebaseDistributor.sol#L282) | [304](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20RebaseDistributor.sol#L304) | [339](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20RebaseDistributor.sol#L339)

```solidity
File: src/tokens/CreditToken.sol

61: require(
            rebasingState[account].isRebasing == 0,
            "CreditToken: already rebasing"
        );
72: require(
            rebasingState[account].isRebasing == 1,
            "CreditToken: not rebasing"
        );
```
[61](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/CreditToken.sol#L61) | [72](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/CreditToken.sol#L72)

```solidity
File: src/tokens/GuildToken.sol

124: require(msg.sender == profitManager, "UNAUTHORIZED");
137: require(
            _lastGaugeLoss != 0 && _lastGaugeLossApplied < _lastGaugeLoss,
            "GuildToken: no loss to apply"
        );
187: require(
            transferable || from == address(0) || to == address(0),
            "GuildToken: transfers disabled"
        );
214: require(
            _lastGaugeLossApplied >= _lastGaugeLoss,
            "GuildToken: pending loss"
        );
227: require(
                issuance <= debtCeilingAfterDecrement,
                "GuildToken: debt ceiling used"
            );
252: require(
                _lastGaugeLossApplied >= _lastGaugeLoss,
                "GuildToken: pending loss"
            );
```
[124](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/GuildToken.sol#L124) | [137](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/GuildToken.sol#L137) | [187](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/GuildToken.sol#L187) | [214](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/GuildToken.sol#L214) | [227](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/GuildToken.sol#L227) | [252](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/GuildToken.sol#L252)

```solidity
File: src/loan/AuctionHouse.sol

61: require(_midPoint < _auctionDuration, "AuctionHouse: invalid params");
77: require(
            core().hasRole(CoreRoles.GAUGE_PNL_NOTIFIER, msg.sender),
            "AuctionHouse: invalid caller"
        );
84: require(
            loan.callTime == block.timestamp,
            "AuctionHouse: loan previously called"
        );
90: require(
            auctions[loanId].startTime == 0,
            "AuctionHouse: auction exists"
        );
123: require(_startTime != 0, "AuctionHouse: invalid auction");
126: require(auctions[loanId].endTime == 0, "AuctionHouse: auction ended");
172: require(creditAsked != 0, "AuctionHouse: cannot bid 0");
206: require(creditAsked == 0, "AuctionHouse: ongoing auction");
```
[61](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/AuctionHouse.sol#L61) | [77](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/AuctionHouse.sol#L77) | [84](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/AuctionHouse.sol#L84) | [90](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/AuctionHouse.sol#L90) | [123](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/AuctionHouse.sol#L123) | [126](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/AuctionHouse.sol#L126) | [172](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/AuctionHouse.sol#L172) | [206](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/AuctionHouse.sol#L206)

```solidity
File: src/loan/LendingTerm.sol

344: require(borrowAmount != 0, "LendingTerm: cannot borrow 0");
345: require(collateralAmount != 0, "LendingTerm: cannot stake 0");
352: require(loans[loanId].borrowTime == 0, "LendingTerm: loan exists");
359: require(
            borrowAmount <= maxBorrow,
            "LendingTerm: not enough collateral"
        );
365: require(
            borrowAmount >= ProfitManager(refs.profitManager).minBorrow(),
            "LendingTerm: borrow amount too low"
        );
373: require(
            _postBorrowIssuance <= params.hardCap,
            "LendingTerm: hardcap reached"
        );
392: require(_debtCeiling != 0, "LendingTerm: debt ceiling reached");
394: require(
                _postBorrowIssuance <= _debtCeiling,
                "LendingTerm: debt ceiling reached"
            );
453: require(collateralToAdd != 0, "LendingTerm: cannot add 0");
458: require(loan.borrowTime != 0, "LendingTerm: loan not found");
459: require(loan.closeTime == 0, "LendingTerm: loan closed");
460: require(loan.callTime == 0, "LendingTerm: loan called");
499: require(borrowTime != 0, "LendingTerm: loan not found");
500: require(
            borrowTime < block.timestamp,
            "LendingTerm: loan opened in same block"
        );
504: require(loan.closeTime == 0, "LendingTerm: loan closed");
505: require(loan.callTime == 0, "LendingTerm: loan called");
509: require(debtToRepay < loanDebt, "LendingTerm: full repayment");
519: require(
            principalRepaid != 0 && interestRepaid != 0,
            "LendingTerm: repay too small"
        );
523: require(
            debtToRepay >= (loanDebt * params.minPartialRepayPercent) / 1e18,
            "LendingTerm: repay below min"
        );
527: require(
            borrowAmount - issuanceDecrease >
                ProfitManager(refs.profitManager).minBorrow(),
            "LendingTerm: below min borrow"
        );
572: require(borrowTime != 0, "LendingTerm: loan not found");
573: require(
            borrowTime < block.timestamp,
            "LendingTerm: loan opened in same block"
        );
577: require(loan.closeTime == 0, "LendingTerm: loan closed");
578: require(loan.callTime == 0, "LendingTerm: loan called");
643: require(loan.borrowTime != 0, "LendingTerm: loan not found");
646: require(loan.closeTime == 0, "LendingTerm: loan closed");
649: require(loan.callTime == 0, "LendingTerm: loan called");
652: require(
            GuildToken(refs.guildToken).isDeprecatedGauge(address(this)) ||
                partialRepayDelayPassed(loanId),
            "LendingTerm: cannot call"
        );
659: require(
            borrowTime < block.timestamp,
            "LendingTerm: loan opened in same block"
        );
699: require(loan.borrowTime != 0, "LendingTerm: loan not found");
702: require(loan.closeTime == 0, "LendingTerm: loan closed");
733: require(msg.sender == refs.auctionHouse, "LendingTerm: invalid caller");
734: require(
            loans[loanId].callTime != 0 && loans[loanId].callDebt != 0,
            "LendingTerm: loan not called"
        );
738: require(loans[loanId].closeTime == 0, "LendingTerm: loan closed");
744: require(
            collateralOut == loans[loanId].collateralAmount ||
                collateralOut == 0,
            "LendingTerm: invalid collateral movements"
        );
764: require(
                collateralToBorrower == 0,
                "LendingTerm: invalid collateral movement"
            );
836: require(
            AuctionHouse(refs.auctionHouse).nAuctionsInProgress() == 0,
            "LendingTerm: auctions in progress"
        );
```
[344](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L344) | [345](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L345) | [352](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L352) | [359](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L359) | [365](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L365) | [373](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L373) | [392](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L392) | [394](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L394) | [453](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L453) | [458](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L458) | [459](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L459) | [460](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L460) | [499](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L499) | [500](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L500) | [504](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L504) | [505](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L505) | [509](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L509) | [519](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L519) | [523](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L523) | [527](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L527) | [572](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L572) | [573](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L573) | [577](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L577) | [578](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L578) | [643](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L643) | [646](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L646) | [649](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L649) | [652](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L652) | [659](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L659) | [699](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L699) | [702](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L702) | [733](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L733) | [734](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L734) | [738](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L738) | [744](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L744) | [764](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L764) | [836](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L836)

```solidity
File: src/loan/SimplePSM.sol

120: require(
            !CreditToken(credit).isRebasing(msg.sender),
            "SimplePSM: already rebasing"
        );
138: require(!redemptionsPaused, "SimplePSM: redemptions paused");
```
[120](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/SimplePSM.sol#L120) | [138](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/SimplePSM.sol#L138)

```solidity
File: src/loan/SurplusGuildMinter.sol

121: require(
            lastGaugeLoss != block.timestamp,
            "SurplusGuildMinter: loss in block"
        );
125: require(amount >= MIN_STAKE, "SurplusGuildMinter: min stake");
169: require(
            amount != 0 && userStake.credit >= amount,
            "SurplusGuildMinter: invalid amount"
        );
191: require(
                userStake.credit >= MIN_STAKE,
                "SurplusGuildMinter: remaining stake below min"
            );
```
[121](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/SurplusGuildMinter.sol#L121) | [125](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/SurplusGuildMinter.sol#L125) | [169](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/SurplusGuildMinter.sol#L169) | [191](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/SurplusGuildMinter.sol#L191)

```solidity
File: src/utils/RateLimitedV2.sol

50: require(
            _rateLimitPerSecond <= _maxRateLimitPerSecond,
            "RateLimited: rateLimitPerSecond too high"
        );
64: require(
            newRateLimitPerSecond <= MAX_RATE_LIMIT_PER_SECOND,
            "RateLimited: rateLimitPerSecond too high"
        );
96: require(newBuffer != 0, "RateLimited: no rate limit buffer");
97: require(amount <= newBuffer, "RateLimited: rate limit hit");
```
[50](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/utils/RateLimitedV2.sol#L50) | [64](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/utils/RateLimitedV2.sol#L64) | [96](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/utils/RateLimitedV2.sol#L96) | [97](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/utils/RateLimitedV2.sol#L97)
</details>

