# QA Report

| QA     | Issue                                                                                                           |
|--------|-----------------------------------------------------------------------------------------------------------------|
| [L-01] | Function and Event Arguments With Different Order                                                               |
| [L-02] | State Variables not Limited to Reasonable Values                                                                |
| [L-03] | Incorrect NatSpec                                                                                               |
| [L-04] | Panic Used for Error Handling                                                                                   |
| [L-05] | Removing an account from `canContractExceedMaxDelegates` does not Decrease its Number of Delegates if Exceeded  |
| [NC-01] | Unnecessary Casting                                                                                            |
| [NC-02] | Implicit Casting                                                                                               |
| [NC-03] | Inconsistent `uint` Types in Events                                                                            |
| [NC-04] | Typos in Comments                                                                                              |


## [L-01] Function and Event Arguments With Different Order

### Description
The `LendingTerm.sol` and `ERC20Gauges.sol` files contain some functions with parameters that emit events containing the same arguments but with a different order.

Example from `ERC20Gauges.sol`:
```solidity
function _addGauge(
    uint256 _type,
    address gauge
) internal returns (uint256 weight) {
    ...
    emit AddGauge(gauge, _type);
}
```

Instances:

- https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L339-L343
- https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L428-L434

- https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L448-L452
- https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L473-L478

- https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L490-L494
- https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L558

- https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/ERC20Gauges.sol#L395-L398
- https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/ERC20Gauges.sol#L422

### Impact
The highlighted inconsistencies could difficult traceability of events.

### Recommended Mitigation Steps
For consistency sake, make sure that arguments of functions that are also used by events are passed/emitted in the same order.


## [L-02] State Variables not Limited to Reasonable Values

Consider adding appropriate minimum/maximum value checks to ensure that the following state variables can never be used to excessively harm users, including via griefing.
Even though the bot report contains some instances for this issue already (see L-11: https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/bot-report.md#l-11-state-variables-not-limited-to-reasonable-values), it missed others like:
- https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/LendingTermOffboarding.sol#L82C9-L82C15
- https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/GuildToken.sol#L198


## [L-03] Incorrect NatSpec

### Description
The NatSpec of the file `ERC20Gauges.sol` at https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/ERC20Gauges.sol#L7-L57 is not accurate.

For instance, it indicates "Rename `calculateGaugeAllocation` to `calculateGaugeStoredAllocation` to make clear that it reads from stored weights." and "Consistency: make incrementGauges return a uint112 instead of uint256" even though `calculateGaugeStoredAllocation()` does not exist and `incrementGauges()` is still returning `uint256`.

See similar low findings from past contests:
- https://solodit.xyz/issues/l-04-incorrect-natspec-for-code4rena-yield-yield-witch-v2-contest-git
- https://solodit.xyz/issues/l-07-natspec-incorrect-code4rena-rubicon-rubicon-contest-git
- https://solodit.xyz/issues/the-param-natspec-comment-for-_key-inassetlogic_swapasset-is-incorrect-spearbit-connext-pdf
- https://solodit.xyz/issues/missingincomplete-natspec-comments-halborn-coredao-genesis-pdf

### Recommended Mitigation Steps
Suggest removing/updating the referenced comments and/or update the code accordingly.

## [L-04] Panic Used for Error Handling

### Description
In the line below, the code panics when an arithmetic overflow or underflow occurs. Panics should be reserved for programmer errors (e.g., assertion violations). Panicking on user errors dilutes the utility of the panic operation.

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/ERC20MultiVotes.sol#L349-L350

Similar low risk issues from other audits:

- https://solodit.xyz/issues/panics-used-for-error-handling-trailofbits-drift-protocol-pdf
- https://solodit.xyz/issues/panic-is-used-for-error-handling-halborn-none-a41-supernova-cosmos-pdf

### Proof of Concept
As a proof of concept, you can execute the `testDecrementOverWeightFails()` test function of the `ERC20MultiVotes.t.sol` test file by running `forge test --mc ERC20MultiVotesUnitTest --silent -vvvv --mt "\btestDecrementOverWeightFails\b"`. You should see that the execution is reverted with a "Panic" error as expected by the test.

### Recommended Mitigation Steps
Instead of using panics, custom errors should be defined and handled according to best practices.

## [L-05] Removing an account from `canContractExceedMaxDelegates` does not Decrease its Number of Delegates if Exceeded

### Description
The abstract contract `ERC20MultiVotes` is an ERC20 extension that allows delegations to multiple delegatees up to a user's balance on a given block. As indicated in the NatSpec, `maxDelegates` is a critical variable and it defines the maximum amount of addresses that users can delegate to. `_setContractExceedMaxDelegates()` is an internal function that allows certain addresses to bypass the `maxDelegates` limitation. The `CreditToken` and `GuildToken` contracts make use of it and protect it with the `CREDIT_GOVERNANCE_PARAMETERS` and `GUILD_GOVERNANCE_PARAMETERS` roles respectively.

A logic flaw was found which allow addresses that had no limitation at some point to continue exceeding the `maxDelegates`. More specifically, if an account has been set to true in the `canContractExceedMaxDelegates` mapping and has more delegations than the ones allowed, removing it from the mapping does not decrease its delegation number to the maximum allowed. See the proof of concept below for a clearer view of the issue.

This finding was only rated as low risk due to its limited exploitation and likelihood.

### Proof of Concept

Add the highlighted lines to the `testDelegateOverMaxDelegatesApproved()` test function in the `ERC20MultiVotes.t.sol` test file like shown below:

```diff
function testDelegateOverMaxDelegatesApproved() public {
    token.mint(address(this), 100e18);
    token.setMaxDelegates(2);

    token.setContractExceedMaxDelegates(address(this), true);
    token.incrementDelegation(delegate1, 50e18);
    token.incrementDelegation(delegate2, 1e18);
    token.incrementDelegation(address(this), 1e18);

    require(token.delegateCount(address(this)) == 3);
    require(token.delegateCount(address(this)) > token.maxDelegates());
    require(token.userDelegatedVotes(address(this)) == 52e18);
+   assertTrue(token.canContractExceedMaxDelegates(address(this)));

+    // remove maxDelegates privilege
+    token.setContractExceedMaxDelegates(address(this), false);
+    assertFalse(token.canContractExceedMaxDelegates(address(this)));
+    require(token.delegateCount(address(this)) == 3);
+    require(token.delegateCount(address(this)) > token.maxDelegates());
+    require(token.userDelegatedVotes(address(this)) == 52e18);
+    // see it can still increment
+    token.incrementDelegation(delegate2, 1e18);
+    require(token.userDelegatedVotes(address(this)) == 53e18);
+    token.incrementDelegation(address(this), 1e18);
+    require(token.userDelegatedVotes(address(this)) == 54e18);
}
```

Then run it with `forge test --mc ERC20MultiVotesUnitTest --silent -vvvv --mt "\btestDelegateOverMaxDelegatesApproved\b"`.
Observe that the user without the privilege to exceed `maxDelegates` (`canContractExceedMaxDelegates` set to false) can still have more delegations than allowed and increment delegations (if they are still to an already allowed address).

## [NC-01] Unnecessary Casting

### Description
The following lines contain a casting from `uint256` to `uint256`, which is completely unnecessary.

- https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/GuildToken.sol#L154
- https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/GuildToken.sol#L158

Similar finding from another contest: https://solodit.xyz/issues/g-06-unnecessary-casting-as-variable-is-already-of-the-same-type-code4rena-frankencoin-frankencoin-git

### Recommended Mitigation Steps
Consider removing the redundant casts.

## [NC-02] Implicit Casting

### Description
The lack of explicit casting between different variable types, hinders code’s readability, making it more error-prone and hard to maintain.

The following lines contain an implicit casting from `uint128` argument (`_bufferCap`) to a `uint224` (`bufferStored`):

- https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/utils/RateLimitedV2.sol#L34
- https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/utils/RateLimitedV2.sol#L43
- https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/utils/RateLimitedV2.sol#L48

### Recommended Mitigation Steps
Consider explicitly casting all integer values to their expected type and use `SafeCast` to safely convert between different integer types as a best practice.


## [NC-03] Inconsistent `uint` Types in Events

### Description
While the `RateLimitedV2.sol` file had `uint` variables of different types (`uint256`, `uint128`, `uint32` and `uint224`), the events from the inherited interface `IRateLimitedV2` were all using `uint256`.

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/utils/RateLimitedV2.sol#L106
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/utils/RateLimitedV2.sol#L134
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/utils/RateLimitedV2.sol#L155

Event definitions: https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/utils/IRateLimitedV2.sol#L39-L52


### Recommended Mitigation Steps
Consider updating the events or variable types so they are consistent.


## [NC-04] Typos in Comments

### Description
A few typos were found in some comments:
- https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/GuildToken.sol#L239
- https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/GuildGovernor.sol#L53C16-L53C25
- https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/LendingTermOnboarding.sol#L91
- https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/ProfitManager.sol#L387

### Recommended Mitigation Steps
Consider fixing the typos for improved readability.
