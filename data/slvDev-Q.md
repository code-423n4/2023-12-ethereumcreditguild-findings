## Low Findings

|    | Issue | Instances |
|----|-------|:---------:|
| [L-01] | Subtraction may underflow if previous math operation is too large | 1 |
| [L-02] | `guardianCancel()` does not check input arrays lengths | 1 |
| [L-03] | Unchecked Return Values of `transfer()/transferFrom()` | 14 |
| [L-04] | `assert()` should only be used for tests | 5 |
| [L-05] | Risk of Lost Ownership While System is Paused | 1 |
| [L-06] | Input Validation Missing in Setter Functions | 2 |


## NonCritical Findings

|    | Issue | Instances |
|----|-------|:---------:|
| [N-01] | Consider use `Address.sendValue` instead Low-Level Call for eth Transfers | 1 |
| [N-02] | Double Type Casting | 1 |
| [N-03] | Equal `block.timestamp` Cases Not Handled | 8 |
| [N-04] | Missing Event Emission After Critical Initialize() Function | 2 |
| [N-05] | Prefer Custom Errors Over Unnamed `revert()` | 3 |
| [N-06] | Avoid using constant decimals in expressions | 8 |
| [N-07] | Consider Avoid Loops in Public Functions | 6 |
| [N-08] | Prefer Casting to `bytes` or `bytes32` Over `abi.encodePacked()` for Single Arguments | 1 |
| [N-09] | `if`-statement can be converted to a ternary | 1 |
| [N-10] | Non-Standard Annotations Detected Use `@inheritdoc` instead | 4 |
| [N-11] | Function/Constructor Argument Names Not in mixedCase | 19 |
| [N-12] | Insufficient Comments in Complex Functions | 16 |
| [N-13] | Leverage Recent Solidity Features with `0.8.21` | 20 |
| [N-14] | Invalid NatSpec comment style | 1 |
| [N-15] | Use of `override` is unnecessary | 49 |


## Low Findings Details

### [L-01] Subtraction may underflow if previous math operation is too large

If `elapsed` variable will be beggir that `PHASE_2_DURATION` then `(_callDebt * elapsed) / PHASE_2_DURATION` will be bigger than `_callDebt` and transaction will be reverted becourse of underflow.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/loan/AuctionHouse.sol

152: creditAsked = _callDebt - (_callDebt * elapsed) / PHASE_2_DURATION;
```
[152](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/loan/AuctionHouse.sol#L152)
</details>

### [L-02] `guardianCancel()` does not check input arrays lengths

`guardianCancel()` function call internal `_cancel()` that call `super._cancel()` from inherited OZ `governance/Governor.sol`
This internal function does not check array lengths and calculate `proposalId` based on array lengths. 

```solidity
    function _cancel(
        address[] memory targets,
        uint256[] memory values,
        bytes[] memory calldatas,
        bytes32 descriptionHash
    ) internal virtual returns (uint256) {
        uint256 proposalId = hashProposal(targets, values, calldatas, descriptionHash);
```

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/governance/GuildGovernor.sol

110: function guardianCancel(
        address[] memory targets,
        uint256[] memory values,
        bytes[] memory calldatas,
        bytes32 descriptionHash
    ) public onlyCoreRole(CoreRoles.GUARDIAN) returns (uint256) {
```
[110](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildGovernor.sol#L110)
</details>

### [L-03] Unchecked Return Values of `transfer()/transferFrom()`

`CreditToken` has overriden the `transfer()` and `transferFrom()` methods of `IERC20` witch call `ERC20RebaseDistributor.transfer/from()`
that has custom logic and returns a boolean value. The return value is not checked in highlighted cases.

<details>
<summary><i>14 issue instances in 3 files:</i></summary>

```solidity
File: src/governance/ProfitManager.sol

252: CreditToken(credit).transferFrom(msg.sender, address(this), amount);
260: CreditToken(credit).transferFrom(msg.sender, address(this), amount);
273: CreditToken(credit).transfer(to, amount);
285: CreditToken(credit).transfer(to, amount);
371: CreditToken(_credit).transfer(
                    _profitSharingConfig.otherRecipient,
                    amountForOther
                );
434: CreditToken(credit).transfer(user, creditEarned);
```
[252](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/ProfitManager.sol#L252) | [260](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/ProfitManager.sol#L260) | [273](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/ProfitManager.sol#L273) | [285](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/ProfitManager.sol#L285) | [371](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/ProfitManager.sol#L371) | [434](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/ProfitManager.sol#L434)

```solidity
File: src/loan/LendingTerm.sol

539: CreditToken(refs.creditToken).transferFrom(
            repayer,
            address(this),
            debtToRepay
        );
546: CreditToken(refs.creditToken).transfer(
            refs.profitManager,
            interestRepaid
        );
590: CreditToken(refs.creditToken).transferFrom(
            repayer,
            address(this),
            loanDebt
        );
597: CreditToken(refs.creditToken).transfer(
                refs.profitManager,
                interest
            );
775: CreditToken(refs.creditToken).transferFrom(
                bidder,
                address(this),
                creditFromBidder
            );
792: CreditToken(refs.creditToken).transfer(
                    refs.profitManager,
                    interest
                );
```
[539](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L539) | [546](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L546) | [590](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L590) | [597](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L597) | [775](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L775) | [792](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L792)

```solidity
File: src/loan/SurplusGuildMinter.sol

128: CreditToken(credit).transferFrom(msg.sender, address(this), amount);
263: CreditToken(credit).transfer(user, creditReward);
```
[128](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/SurplusGuildMinter.sol#L128) | [263](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/SurplusGuildMinter.sol#L263)
</details>

### [L-04] `assert()` should only be used for tests

From (documentation)[https://docs.soliditylang.org/en/v0.8.20/control-structures.html#panic-via-assert-and-error-via-require]:
```text
The assert function creates an error of type Panic(uint256). The same error is created by the compiler in certain situations as listed below.
Assert should only be used to test for internal errors, and to check invariants. Properly functioning code should never create a Panic, not even on invalid external input. If this happens, then there is a bug in your contract which you should fix. Language analysis tools can evaluate your contract to identify the conditions and function calls which will cause a Panic.
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

### [L-05] Risk of Lost Ownership While System is Paused

If role-holder loses their `role` while the system is paused, it can lead to user assets becoming permanently locked.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/core/CoreRef.sol

57: function pause() public onlyCoreRole(CoreRoles.GUARDIAN) {
```
[57](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/core/CoreRef.sol#L57)
</details>

### [L-06] Input Validation Missing in Setter Functions

Setter functions are utilized to update the state variables of a contract.
It is critical to ensure these functions have adequate input sanitization to prevent unwanted alterations.

Consider incorporating appropriate validation mechanisms.

<details>
<summary><i>2 issue instances in 1 files:</i></summary>

```solidity
File: src/loan/LendingTerm.sol

/// @audit `setAuctionHouse` function does not validate `_newValue` input
829: function setAuctionHouse(
        address _newValue
    ) external onlyCoreRole(CoreRoles.GOVERNOR) {
/// @audit `setHardCap` function does not validate `_newValue` input
846: function setHardCap(
        uint256 _newValue
    ) external onlyCoreRole(CoreRoles.GOVERNOR) {
```
[829](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L829) | [846](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L846)
</details>


## NonCritical Findings Details

### [N-01] Consider use `Address.sendValue` instead Low-Level Call for eth Transfers

Directly using low-level calls for Ether transfers can introduce vulnerabilities and obscure the intent of your transfers.
Adopt modern Solidity best practices by switching to recognized libraries like `SafeTransferLib.safeTransferETH` or `Address.sendValue`.
This ensures safer transactions, enhances code clarity, and aligns with the standards of the Solidity community.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/core/CoreRef.sol

101: (bool success, bytes memory returned) = target.call{value: value}(
    callData
);
```
[101](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/core/CoreRef.sol#L101)
</details>

### [N-02] Double Type Casting

`gaugeWeight` is uint256 that casted to int256 and then back to uint256. 
Casting from uint256 to int256 can leat to unhendlend overflow.
Conside to use `SafeCast` library from OpenZeppelin.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/loan/LendingTerm.sol

/// @audit `gaugeWeight` is uint256
277: gaugeWeight = uint256(int256(gaugeWeight) + gaugeWeightDelta);
```
[277](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L277)
</details>

### [N-03] Equal `block.timestamp` Cases Not Handled

Solidity code that checks if `block.timestamp` is greater than a certain variable but fails to handle the case when `block.timestamp` is equal to the variable may lead to unintended behaviors and potential vulnerabilities.

To ensure correct and secure function execution, it's recommended to include checks for cases where `block.timestamp` equals the compared variable.

<details>
<summary><i>8 issue instances in 4 files:</i></summary>

```solidity
File: src/governance/LendingTermOnboarding.sol

189: lastProposal[term] + MIN_DELAY_BETWEEN_PROPOSALS < block.timestamp,
```
[189](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOnboarding.sol#L189)

```solidity
File: src/governance/GuildVetoGovernor.sol

283: if (timelockOperationTimestamp > block.timestamp) {
```
[283](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildVetoGovernor.sol#L283)

```solidity
File: src/loan/AuctionHouse.sol

134: if (block.timestamp < _startTime + midPoint) {
144: else if (block.timestamp < _startTime + auctionDuration) {
```
[134](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/AuctionHouse.sol#L134) | [144](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/AuctionHouse.sol#L144)

```solidity
File: src/loan/LendingTerm.sol

251: lastPartialRepay[loanId] <
            block.timestamp - params.maxDelayBetweenPartialRepay;
501: borrowTime < block.timestamp,
574: borrowTime < block.timestamp,
660: borrowTime < block.timestamp,
```
[251](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L251) | [501](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L501) | [574](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L574) | [660](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L660)
</details>

### [N-04] Missing Event Emission After Critical Initialize() Function

Emitting an initialization event offers clear, on-chain evidence of the contract's initialization state, enhancing transparency and auditability.
This practice aids users and developers in accurately tracking the contract's lifecycle, pinpointing the precise moment of its initialization.
Moreover, it aligns with best practices for event logging in smart contracts, ensuring that significant state changes are both observable and verifiable through emitted events.

<details>
<summary><i>2 issue instances in 2 files:</i></summary>

```solidity
File: src/governance/ProfitManager.sol

156: function initializeReferences(
        address _credit,
        address _guild,
        address _psm
    ) external onlyCoreRole(CoreRoles.GOVERNOR)
```
[156](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/ProfitManager.sol#L156)

```solidity
File: src/loan/LendingTerm.sol

160: function initialize(
        address _core,
        LendingTermReferences calldata _refs,
        LendingTermParams calldata _params
    ) external
```
[160](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/loan/LendingTerm.sol#L160)
</details>

### [N-05] Prefer Custom Errors Over Unnamed `revert()`

Using custom error types with `revert()` in Solidity provides clearer, more informative error handling compared to unnamed `revert()` calls.
Custom errors, introduced in Solidity 0.8.4, allow you to define specific error conditions with descriptive names and optionally include additional data about the error.

<details>
<summary><i>3 issue instances in 2 files:</i></summary>

```solidity
File: src/governance/LendingTermOnboarding.sol

177: revert("LendingTermOnboarding: cannot propose arbitrary actions")
```
[177](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/LendingTermOnboarding.sol#L177)

```solidity
File: src/governance/GuildVetoGovernor.sol

213: revert("GuildVetoGovernor: can only vote against in veto proposals")
310: revert("GuildVetoGovernor: cannot propose arbitrary actions")
```
[213](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/GuildVetoGovernor.sol#L213) | [310](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/GuildVetoGovernor.sol#L310)
</details>

### [N-06] Avoid using constant decimals in expressions

The use of fixed decimal values such as `1e18/10 ** 18` in Solidity contracts can lead to bugs, and vulnerabilities, when interacting with tokens having different decimal configurations.
Not all ERC20 tokens follow the standard 18 decimal places, and assumptions about decimal places can lead to miscalculations.
        
Always retrieve and use the `decimals()` function from the token contract itself when performing calculations involving token amounts.
This ensures that your contract correctly handles tokens with any number of decimal places, mitigating the risk of numerical errors or under/overflows.

<details>
<summary><i>8 issue instances in 2 files:</i></summary>

```solidity
File: src/governance/ProfitManager.sol

152: return (_minBorrow * 1e18) / creditMultiplier;
246: creditSplit = 1e18 - surplusBufferSplit - guildSplit - otherSplit;
429: creditEarned = (_userGaugeWeight * deltaIndex) / 1e18;
483: creditEarned[i] = (_userGaugeWeight * deltaIndex) / 1e18;
```
[152](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/ProfitManager.sol#L152) | [246](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/ProfitManager.sol#L246) | [429](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/ProfitManager.sol#L429) | [483](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/governance/ProfitManager.sol#L483)

```solidity
File: src/loan/LendingTerm.sol

226: loanDebt += (borrowAmount * _openingFee) / 1e18;
516: uint256 principalRepaid = (principal * percentRepaid) / 1e18;
518: uint256 issuanceDecrease = (borrowAmount * percentRepaid) / 1e18;
524: debtToRepay >= (loanDebt * params.minPartialRepayPercent) / 1e18,
```
[226](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/loan/LendingTerm.sol#L226) | [516](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/loan/LendingTerm.sol#L516) | [518](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/loan/LendingTerm.sol#L518) | [524](https://github.com/code-423n4/2023-12-ethereumcreditguild//blob/main/src/loan/LendingTerm.sol#L524)
</details>


### [N-07] Consider Avoid Loops in Public Functions

Using loops within `public` or `external` functions can pose risks and inefficiencies.
Unpredictable gas consumption due to loop iterations can hinder a function's usability and cost-effectiveness. 
Furthermore, if the loop's logic can be externally influenced or altered, it might be exploited to intentionally drain gas, making the function impractical or uneconomical to call.
To ensure consistent performance and avoid potential vulnerabilities, it's advisable to avoid or limit loops in public functions, especially if their logic can change.

<details>
<summary><i>6 issue instances in 3 files:</i></summary>

```solidity
File: src/governance/ProfitManager.sol

443: for (uint256 i = 0; i < gauges.length; ) {
467: for (uint256 i = 0; i < gauges.length; ) {
```
[443](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/ProfitManager.sol#L443) | [467](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/ProfitManager.sol#L467)

```solidity
File: src/tokens/ERC20Gauges.sol

134: for (uint256 i; i < allGauges.length && j < _liveGauges.length; ) {
280: for (uint256 i = 0; i < size; ) {
352: for (uint256 i = 0; i < size; ) {
```
[134](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20Gauges.sol#L134) | [280](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20Gauges.sol#L280) | [352](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20Gauges.sol#L352)

```solidity
File: src/loan/LendingTerm.sol

685: for (uint256 i = 0; i < loanIds.length; i++) {
```
[685](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L685)
</details>

### [N-08] Prefer Casting to `bytes` or `bytes32` Over `abi.encodePacked()` for Single Arguments

When using `abi.encodePacked()` on a single argument, it is often clearer to use a cast to `bytes` or `bytes32`.
This improves the semantic clarity of the code, making it easier for reviewers to understand the developer's intentions.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/governance/GuildVetoGovernor.sol

389: string(abi.encodePacked(timelockId))
```
[389](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildVetoGovernor.sol#L389)
</details>

### [N-09] `if`-statement can be converted to a ternary

The ternary operator provides a shorthand way to write if / else statements.
It can improve readability and reduce the number of lines of code.
Traditional `if / else` statements, particularly those spanning only a one lines, could potentially be refactored using the ternary operator.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/tokens/ERC20Gauges.sol

406: if (newAdd) {
            // save gauge type on first add
            gaugeType[gauge] = _type;
        } else {
            // cannot change gauge type on re-add of a previously deprecated gauge
            require(gaugeType[gauge] == _type, "ERC20Gauges: invalid type");
        }
```
[406](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20Gauges.sol#L406)
</details>

### [N-10] Non-Standard Annotations Detected Use `@inheritdoc` instead

Using non-standard annotations like `@dev see Ellipsis` can lead to inconsistencies and lack of clarity in your smart contract documentation.
It's recommended to use the `@inheritdoc` annotation for enhanced clarity and uniformity in smart contract development.

`@inheritdoc` copies all state variables, internal and public function comments from the base contract to avoid repetition.

<details>
<summary><i>4 issue instances in 1 files:</i></summary>

```solidity
File: src/governance/GuildVetoGovernor.sol

127: * @dev See {IGovernor-COUNTING_MODE}.
141: * @dev See {IGovernor-hasVoted}.
170: * @dev See {Governor-_quorumReached}.
193: * @dev See {Governor-_countVote}. In this module, the support follows the `VoteType` enum (from Governor Bravo).
```
[127](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildVetoGovernor.sol#L127) | [141](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildVetoGovernor.sol#L141) | [170](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildVetoGovernor.sol#L170) | [193](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildVetoGovernor.sol#L193)
</details>

### [N-11] Function/Constructor Argument Names Not in mixedCase

Underscore before of after function argument names is a common convention in Solidity NOT a documentation requirement.

Function arguments should use mixedCase for better readability and consistency with Solidity style guidelines. 
Examples of good practice include: initialSupply, account, recipientAddress, senderAddress, newOwner. 
[More information in Documentation](https://docs.soliditylang.org/en/v0.8.20/style-guide.html#function-argument-names)

Rule exceptions
- Allow constant variable name/symbol/decimals to be lowercase (ERC20).
- Allow `_` at the beginning of the mixedCase match for `private variables` and `unused parameters`.

<details>
<summary><i>19 issue instances in 11 files:</i></summary>

```solidity
File: src/governance/LendingTermOffboarding.sol

78: function setQuorum(
        uint256 _quorum
    ) external onlyCoreRole(CoreRoles.GOVERNOR) {
65: constructor(
        address _core,
        address _guildToken,
        address _psm,
        uint256 _quorum
    ) CoreRef(_core) {
```
[78](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOffboarding.sol#L78) | [65](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOffboarding.sol#L65)

```solidity
File: src/governance/LendingTermOnboarding.sol

62: constructor(
        LendingTerm.LendingTermReferences memory _lendingTermReferences,
        uint256 _gaugeType,
        address _core,
        address _timelock,
        uint256 initialVotingDelay,
        uint256 initialVotingPeriod,
        uint256 initialProposalThreshold,
        uint256 initialQuorum
    )
        GuildGovernor(
            _core,
            _timelock,
            _lendingTermReferences.guildToken,
            initialVotingDelay,
            initialVotingPeriod,
            initialProposalThreshold,
            initialQuorum
        )
    {
```
[62](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOnboarding.sol#L62)

```solidity
File: src/governance/ProfitManager.sol

156: function initializeReferences(
        address _credit,
        address _guild,
        address _psm
    ) external onlyCoreRole(CoreRoles.GOVERNOR) {
```
[156](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/ProfitManager.sol#L156)

```solidity
File: src/tokens/ERC20Gauges.sol

394: function _addGauge(
        uint256 _type,
        address gauge
    ) internal returns (uint256 weight) {
```
[394](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20Gauges.sol#L394)

```solidity
File: src/tokens/GuildToken.sol

76: function addGauge(
        uint256 _type,
        address gauge
    ) external onlyCoreRole(CoreRoles.GAUGE_ADD) returns (uint256) {
197: function setProfitManager(address _newProfitManager) external onlyCoreRole(CoreRoles.GOVERNOR) {
42: constructor(
        address _core,
        address _profitManager
    )
        CoreRef(_core)
        ERC20("Ethereum Credit Guild - GUILD", "GUILD")
        ERC20Permit("Ethereum Credit Guild - GUILD")
    {
```
[76](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/GuildToken.sol#L76) | [197](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/GuildToken.sol#L197) | [42](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/GuildToken.sol#L42)

```solidity
File: src/loan/AuctionHouse.sol

55: constructor(
        address _core,
        uint256 _midPoint,
        uint256 _auctionDuration
    ) CoreRef(_core) {
```
[55](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/AuctionHouse.sol#L55)

```solidity
File: src/loan/LendingTerm.sol

160: function initialize(
        address _core,
        LendingTermReferences calldata _refs,
        LendingTermParams calldata _params
    ) external {
634: function _call(
        address caller,
        bytes32 loanId,
        address _auctionHouse
    ) internal {
829: function setAuctionHouse(
        address _newValue
    ) external onlyCoreRole(CoreRoles.GOVERNOR) {
846: function setHardCap(
        uint256 _newValue
    ) external onlyCoreRole(CoreRoles.GOVERNOR) {
```
[160](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L160) | [634](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L634) | [829](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L829) | [846](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L846)

```solidity
File: src/loan/SimplePSM.sol

64: constructor(
        address _core,
        address _profitManager,
        address _credit,
        address _pegToken
    ) CoreRef(_core) {
```
[64](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/SimplePSM.sol#L64)

```solidity
File: src/loan/SurplusGuildMinter.sol

319: function setMintRatio(
        uint256 _mintRatio
    ) external onlyCoreRole(CoreRoles.GOVERNOR) {
328: function setRewardRatio(
        uint256 _rewardRatio
    ) external onlyCoreRole(CoreRoles.GOVERNOR) {
87: constructor(
        address _core,
        address _profitManager,
        address _credit,
        address _guild,
        address _rlgm,
        uint256 _mintRatio,
        uint256 _rewardRatio
    ) CoreRef(_core) {
```
[319](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/SurplusGuildMinter.sol#L319) | [328](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/SurplusGuildMinter.sol#L328) | [87](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/SurplusGuildMinter.sol#L87)

```solidity
File: src/rate-limits/RateLimitedMinter.sol

30: constructor(
        address _core,
        address _token,
        bytes32 _role,
        uint256 _maxRateLimitPerSecond,
        uint128 _rateLimitPerSecond,
        uint128 _bufferCap
    )
        CoreRef(_core)
        RateLimitedV2(_maxRateLimitPerSecond, _rateLimitPerSecond, _bufferCap)
    {
```
[30](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/rate-limits/RateLimitedMinter.sol#L30)

```solidity
File: src/utils/RateLimitedV2.sol

40: constructor(
        uint256 _maxRateLimitPerSecond,
        uint128 _rateLimitPerSecond,
        uint128 _bufferCap
    ) {
```
[40](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/utils/RateLimitedV2.sol#L40)
</details>

### [N-12] Insufficient Comments in Complex Functions

Complex functions spanning multiple lines should have more comments to better explain the purpose of each logic step.
Proper commenting can greatly improve the readability and maintainability of the codebase. Consider adding explicit comments
to provide insights into the function's operation.

<details>
<summary><i>16 issue instances in 5 files:</i></summary>

```solidity
File: src/governance/LendingTermOnboarding.sol

/// @audit function with 50 lines has only 8 comment lines
105: function createTerm(
        address implementation,
        LendingTerm.LendingTermParams calldata params
    ) external returns (address) {
```
[105](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOnboarding.sol#L105)

```solidity
File: src/governance/ProfitManager.sol

/// @audit function with 79 lines has only 16 comment lines
292: function notifyPnL(
        address gauge,
        int256 amount
    ) external onlyCoreRole(CoreRoles.GAUGE_PNL_NOTIFIER) {
```
[292](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/ProfitManager.sol#L292)

```solidity
File: src/tokens/ERC20RebaseDistributor.sol

/// @audit function with 44 lines has only 15 comment lines
168: function updateTotalRebasingShares(
        uint256 currentRebasingSharePrice,
        int256 sharesDelta
    ) internal {
/// @audit function with 37 lines has only 6 comment lines
338: function distribute(uint256 amount) external {
/// @audit function with 37 lines has only 4 comment lines
461: function _burn(address account, uint256 amount) internal virtual override {
/// @audit function with 31 lines has only 5 comment lines
509: function _mint(address account, uint256 amount) internal virtual override {
/// @audit function with 69 lines has only 10 comment lines
553: function transfer(
        address to,
        uint256 amount
    ) public virtual override returns (bool) {
/// @audit function with 69 lines has only 9 comment lines
646: function transferFrom(
        address from,
        address to,
        uint256 amount
    ) public virtual override returns (bool) {
```
[168](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20RebaseDistributor.sol#L168) | [338](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20RebaseDistributor.sol#L338) | [461](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20RebaseDistributor.sol#L461) | [509](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20RebaseDistributor.sol#L509) | [553](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20RebaseDistributor.sol#L553) | [646](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20RebaseDistributor.sol#L646)

```solidity
File: src/loan/LendingTerm.sol

/// @audit function with 50 lines has only 15 comment lines
270: function debtCeiling(
        int256 gaugeWeightDelta
    ) public view returns (uint256) {
/// @audit function with 69 lines has only 12 comment lines
339: function _borrow(
        address borrower,
        uint256 borrowAmount,
        uint256 collateralAmount
    ) internal returns (bytes32 loanId) {
/// @audit function with 52 lines has only 7 comment lines
490: function _partialRepay(
        address repayer,
        bytes32 loanId,
        uint256 debtToRepay
    ) internal {
/// @audit function with 40 lines has only 9 comment lines
567: function _repay(address repayer, bytes32 loanId) internal {
/// @audit function with 70 lines has only 13 comment lines
725: function onBid(
        bytes32 loanId,
        address bidder,
        uint256 collateralToBorrower,
        uint256 collateralToBidder,
        uint256 creditFromBidder
    ) external {
```
[270](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L270) | [339](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L339) | [490](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L490) | [567](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L567) | [725](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L725)

```solidity
File: src/loan/SurplusGuildMinter.sol

/// @audit function with 30 lines has only 5 comment lines
114: function stake(address term, uint256 amount) external whenNotPaused {
/// @audit function with 35 lines has only 10 comment lines
158: function unstake(address term, uint256 amount) external {
/// @audit function with 46 lines has only 9 comment lines
216: function getRewards(
        address user,
        address term
    )
        public
        returns (
            uint256 lastGaugeLoss, // GuildToken.lastGaugeLoss(term)
            UserStake memory userStake, // stake state after execution of getRewards()
            bool slashed // true if the user has been slashed
        )
    {
```
[114](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/SurplusGuildMinter.sol#L114) | [158](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/SurplusGuildMinter.sol#L158) | [216](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/SurplusGuildMinter.sol#L216)
</details>

### [N-13] Leverage Recent Solidity Features with `0.8.21`

The recent updates in Solidity provide several features and optimizations that, when leveraged appropriately, can significantly improve your contract's code clarity and maintainability.
Key enhancements include the use of push0 for placing 0 on the stack for EVM versions starting from "Shanghai", making your code simpler and more straightforward.
Moreover, Solidity has extended NatSpec documentation support to enum and struct definitions, facilitating more comprehensive and insightful code documentation.

Additionally, the re-implementation of the UnusedAssignEliminator and UnusedStoreEliminator in the Solidity optimizer provides the ability to remove unused assignments in deeply nested loops.
This results in a cleaner, more efficient contract code, reducing clutter and potential points of confusion during code review or debugging.
It's recommended to make full use of these features and optimizations to enhance the robustness and readability of your smart contracts.

<details>
<summary><i>20 issue instances in 20 files:</i></summary>

```solidity
File: src/core/Core.sol

2: pragma solidity 0.8.13;
```
[2](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/core/Core.sol#L2)

```solidity
File: src/core/CoreRef.sol

2: pragma solidity 0.8.13;
```
[2](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/core/CoreRef.sol#L2)

```solidity
File: src/core/CoreRoles.sol

2: pragma solidity 0.8.13;
```
[2](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/core/CoreRoles.sol#L2)

```solidity
File: src/governance/GuildGovernor.sol

2: pragma solidity 0.8.13;
```
[2](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildGovernor.sol#L2)

```solidity
File: src/governance/GuildTimelockController.sol

2: pragma solidity 0.8.13;
```
[2](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildTimelockController.sol#L2)

```solidity
File: src/governance/LendingTermOffboarding.sol

2: pragma solidity 0.8.13;
```
[2](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOffboarding.sol#L2)

```solidity
File: src/governance/LendingTermOnboarding.sol

2: pragma solidity 0.8.13;
```
[2](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOnboarding.sol#L2)

```solidity
File: src/governance/ProfitManager.sol

2: pragma solidity 0.8.13;
```
[2](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/ProfitManager.sol#L2)

```solidity
File: src/governance/GuildVetoGovernor.sol

2: pragma solidity 0.8.13;
```
[2](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildVetoGovernor.sol#L2)

```solidity
File: src/tokens/ERC20Gauges.sol

2: pragma solidity 0.8.13;
```
[2](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20Gauges.sol#L2)

```solidity
File: src/tokens/ERC20MultiVotes.sol

4: pragma solidity 0.8.13;
```
[4](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20MultiVotes.sol#L4)

```solidity
File: src/tokens/ERC20RebaseDistributor.sol

2: pragma solidity 0.8.13;
```
[2](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20RebaseDistributor.sol#L2)

```solidity
File: src/tokens/CreditToken.sol

2: pragma solidity 0.8.13;
```
[2](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/CreditToken.sol#L2)

```solidity
File: src/tokens/GuildToken.sol

2: pragma solidity 0.8.13;
```
[2](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/GuildToken.sol#L2)

```solidity
File: src/loan/AuctionHouse.sol

2: pragma solidity 0.8.13;
```
[2](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/AuctionHouse.sol#L2)

```solidity
File: src/loan/LendingTerm.sol

2: pragma solidity 0.8.13;
```
[2](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L2)

```solidity
File: src/loan/SimplePSM.sol

2: pragma solidity 0.8.13;
```
[2](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/SimplePSM.sol#L2)

```solidity
File: src/loan/SurplusGuildMinter.sol

2: pragma solidity 0.8.13;
```
[2](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/SurplusGuildMinter.sol#L2)

```solidity
File: src/rate-limits/RateLimitedMinter.sol

2: pragma solidity 0.8.13;
```
[2](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/rate-limits/RateLimitedMinter.sol#L2)

```solidity
File: src/utils/RateLimitedV2.sol

2: pragma solidity 0.8.13;
```
[2](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/utils/RateLimitedV2.sol#L2)
</details>

### [N-14] Invalid NatSpec comment style

NatSpec comments in Solidity are meant to be recognized by tools for a variety of purposes such as documentation generation.
The correct style is to use `///` for single-line comments and `/* ... */` for multi-line comments.
Incorrect styles can cause tools to not recognize the comments as intended.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: src/governance/GuildVetoGovernor.sol

252: // @dev all proposals that are in this Governor's state should have been created
```
[252](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildVetoGovernor.sol#L252)
</details>

### [N-15] Use of `override` is unnecessary

In Solidity version 0.8.8 and later, the use of the `override` keyword becomes superfluous when a function is overriding solely from an interface and the function isn't present in multiple base contracts.
Previously, the `override` keyword was required as an explicit indication to the compiler. However, this is no longer the case, and the extraneous use of the keyword can make the code less clean and more verbose.

Solidity documentation on [Function Overriding](https://docs.soliditylang.org/en/v0.8.20/contracts.html#function-overriding).

<details>
<summary><i>49 issue instances in 9 files:</i></summary>

```solidity
File: src/governance/GuildGovernor.sol

57: function quorum(
        uint256 /* blockNumber*/
    ) public view override returns (uint256) {
78: function setVotingDelay(
        uint256 newVotingDelay
    ) public override onlyCoreRole(CoreRoles.GOVERNOR) {
85: function setVotingPeriod(
        uint256 newVotingPeriod
    ) public override onlyCoreRole(CoreRoles.GOVERNOR) {
92: function setProposalThreshold(
        uint256 newProposalThreshold
    ) public override onlyCoreRole(CoreRoles.GOVERNOR) {
122: function _cancel(
        address[] memory targets,
        uint256[] memory values,
        bytes[] memory calldatas,
        bytes32 descriptionHash
    ) internal override(Governor, GovernorTimelockControl) returns (uint256) {
131: function _execute(
        uint256 proposalId,
        address[] memory targets,
        uint256[] memory values,
        bytes[] memory calldatas,
        bytes32 descriptionHash
    ) internal override(Governor, GovernorTimelockControl) {
141: function _executor()
        internal
        view
        override(Governor, GovernorTimelockControl)
        returns (address)
    {
150: function proposalThreshold()
        public
        view
        override(Governor, GovernorSettings)
        returns (uint256)
    {
159: function state(
        uint256 proposalId
    )
        public
        view
        override(Governor, GovernorTimelockControl)
        returns (ProposalState)
    {
170: function supportsInterface(
        bytes4 interfaceId
    ) public view override(Governor, GovernorTimelockControl) returns (bool) {
```
[57](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildGovernor.sol#L57) | [78](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildGovernor.sol#L78) | [85](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildGovernor.sol#L85) | [92](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildGovernor.sol#L92) | [122](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildGovernor.sol#L122) | [131](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildGovernor.sol#L131) | [141](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildGovernor.sol#L141) | [150](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildGovernor.sol#L150) | [159](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildGovernor.sol#L159) | [170](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildGovernor.sol#L170)

```solidity
File: src/governance/GuildTimelockController.sol

30: function hasRole(
        bytes32 role,
        address account
    ) public view virtual override returns (bool) {
38: function _setRoleAdmin(bytes32 role, bytes32 adminRole) internal override {}
41: function _grantRole(bytes32 role, address account) internal override {}
44: function _revokeRole(bytes32 role, address account) internal override {}
```
[30](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildTimelockController.sol#L30) | [38](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildTimelockController.sol#L38) | [41](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildTimelockController.sol#L41) | [44](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildTimelockController.sol#L44)

```solidity
File: src/governance/LendingTermOnboarding.sol

171: function propose(
        address[] memory /* targets*/,
        uint256[] memory /* values*/,
        bytes[] memory /* calldatas*/,
        string memory /* description*/
    ) public pure override(IGovernor, Governor) returns (uint256) {
```
[171](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOnboarding.sol#L171)

```solidity
File: src/governance/GuildVetoGovernor.sol

53: function quorum(
        uint256 /* blockNumber*/
    ) public view override returns (uint256) {
130: function COUNTING_MODE()
        public
        pure
        virtual
        override
        returns (string memory)
    {
143: function hasVoted(
        uint256 proposalId,
        address account
    ) public view virtual override returns (bool) {
172: function _quorumReached(
        uint256 proposalId
    ) internal view virtual override returns (bool) {
186: function _voteSucceeded(
        uint256 /* proposalId*/
    ) internal pure virtual override returns (bool) {
195: function _countVote(
        uint256 proposalId,
        address account,
        uint8 support,
        uint256 weight,
        bytes memory // params
    ) internal virtual override {
222: function votingDelay() public pure override returns (uint256) {
230: function votingPeriod() public pure override returns (uint256) {
246: function state(
        uint256 proposalId
    ) public view override returns (ProposalState) {
304: function propose(
        address[] memory /* targets*/,
        uint256[] memory /* values*/,
        bytes[] memory /* calldatas*/,
        string memory /* description*/
    ) public pure override returns (uint256) {
```
[53](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildVetoGovernor.sol#L53) | [130](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildVetoGovernor.sol#L130) | [143](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildVetoGovernor.sol#L143) | [172](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildVetoGovernor.sol#L172) | [186](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildVetoGovernor.sol#L186) | [195](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildVetoGovernor.sol#L195) | [222](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildVetoGovernor.sol#L222) | [230](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildVetoGovernor.sol#L230) | [246](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildVetoGovernor.sol#L246) | [304](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildVetoGovernor.sol#L304)

```solidity
File: src/tokens/ERC20Gauges.sol

475: function _burn(address from, uint256 amount) internal virtual override {
480: function transfer(
        address to,
        uint256 amount
    ) public virtual override returns (bool) {
488: function transferFrom(
        address from,
        address to,
        uint256 amount
    ) public virtual override returns (bool) {
```
[475](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20Gauges.sol#L475) | [480](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20Gauges.sol#L480) | [488](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20Gauges.sol#L488)

```solidity
File: src/tokens/ERC20MultiVotes.sol

402: function _burn(address from, uint256 amount) internal virtual override {
407: function transfer(
        address to,
        uint256 amount
    ) public virtual override returns (bool) {
415: function transferFrom(
        address from,
        address to,
        uint256 amount
    ) public virtual override returns (bool) {
```
[402](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20MultiVotes.sol#L402) | [407](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20MultiVotes.sol#L407) | [415](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20MultiVotes.sol#L415)

```solidity
File: src/tokens/ERC20RebaseDistributor.sol

426: function balanceOf(
        address account
    ) public view virtual override returns (uint256) {
444: function totalSupply() public view virtual override returns (uint256) {
461: function _burn(address account, uint256 amount) internal virtual override {
509: function _mint(address account, uint256 amount) internal virtual override {
553: function transfer(
        address to,
        uint256 amount
    ) public virtual override returns (bool) {
646: function transferFrom(
        address from,
        address to,
        uint256 amount
    ) public virtual override returns (bool) {
```
[426](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20RebaseDistributor.sol#L426) | [444](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20RebaseDistributor.sol#L444) | [461](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20RebaseDistributor.sol#L461) | [509](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20RebaseDistributor.sol#L509) | [553](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20RebaseDistributor.sol#L553) | [646](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20RebaseDistributor.sol#L646)

```solidity
File: src/tokens/CreditToken.sol

82: function _mint(
        address account,
        uint256 amount
    ) internal override(ERC20, ERC20RebaseDistributor) {
89: function _burn(
        address account,
        uint256 amount
    ) internal override(ERC20, ERC20MultiVotes, ERC20RebaseDistributor) {
97: function balanceOf(
        address account
    ) public view override(ERC20, ERC20RebaseDistributor) returns (uint256) {
103: function totalSupply()
        public
        view
        override(ERC20, ERC20RebaseDistributor)
        returns (uint256)
    {
112: function transfer(
        address to,
        uint256 amount
    )
        public
        override(ERC20, ERC20MultiVotes, ERC20RebaseDistributor)
        returns (bool)
    {
124: function transferFrom(
        address from,
        address to,
        uint256 amount
    )
        public
        override(ERC20, ERC20MultiVotes, ERC20RebaseDistributor)
        returns (bool)
    {
```
[82](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/CreditToken.sol#L82) | [89](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/CreditToken.sol#L89) | [97](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/CreditToken.sol#L97) | [103](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/CreditToken.sol#L103) | [112](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/CreditToken.sol#L112) | [124](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/CreditToken.sol#L124)

```solidity
File: src/tokens/GuildToken.sol

182: function _beforeTokenTransfer(
        address from,
        address to,
        uint256 /* amount*/
    ) internal view override {
207: function _decrementGaugeWeight(
        address user,
        address gauge,
        uint256 weight
    ) internal override {
242: function _incrementGaugeWeight(
        address user,
        address gauge,
        uint256 weight
    ) internal override {
278: function _burn(
        address from,
        uint256 amount
    ) internal virtual override(ERC20, ERC20Gauges, ERC20MultiVotes) {
287: function transfer(
        address to,
        uint256 amount
    )
        public
        virtual
        override(ERC20, ERC20Gauges, ERC20MultiVotes)
        returns (bool)
    {
301: function transferFrom(
        address from,
        address to,
        uint256 amount
    )
        public
        virtual
        override(ERC20, ERC20Gauges, ERC20MultiVotes)
        returns (bool)
    {
```
[182](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/GuildToken.sol#L182) | [207](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/GuildToken.sol#L207) | [242](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/GuildToken.sol#L242) | [278](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/GuildToken.sol#L278) | [287](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/GuildToken.sol#L287) | [301](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/GuildToken.sol#L301)
</details>