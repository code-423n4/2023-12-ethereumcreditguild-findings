## [L-1] INCORRECT COMMENTS
The comments on the  overidden ` votingPeriod()`   function of the GuildVetoGovernor.sol documents that ` voting period is unused ` but the votingPeriod function is actually used in the ` propose `  function of the Governor base contract.

```
/// @dev voting period is unused, it is a duration in blocks for the vote//@audit this comment is wrong.
    /// but the timestamp of the action in the timelock is used to know if the
    /// vote period is over (after action is ready in the timelock, the veto
    /// vote failed).
    function votingPeriod() public pure override returns (uint256) {
        return 2425847; // ~1 year with 1 block every 13s //@audit-info remember there are  leap year 2,628,000
    }
```
The @dev comment above contradicts what is actually implented because the votingPeriod function is used in the parent Governor.sol contract to calculate ` voteEnd `  time.

```
function propose(
        address[] memory targets,
        uint256[] memory values,
        bytes[] memory calldatas,
        string memory description
    ) public virtual override returns (uint256) {
...
uint256 snapshot = currentTimepoint + votingDelay();
        uint256 deadline = snapshot + votingPeriod();//@audit votingPeriod is used here.

        _proposals[proposalId] = ProposalCore({
            proposer: proposer,
            voteStart: SafeCast.toUint64(snapshot),
            voteEnd: SafeCast.toUint64(deadline),
...

}
```
File: 

Recommendation: Update the @dev comment on the votingPeriod function not to contradict implementation because comments also serve as documentation to implementation.

## [L-2] The ` executeVeto `  function of the GuildVetoGovernor.sol should be ` payable `
 The   ` executeVeto `  is only a convinience function for the standard ` execute `   function but lacks the ` payable `  keyword so every call with msg.value greater than 0 made to the ` executeVeto `  will revert. 

This will make the ` executeVeto `  unusable for transaction executions that include ETH values.

```
function executeVeto(bytes32 timelockId) external returns (uint256) {//@audit this should be payable.
```

File: https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/GuildVetoGovernor.sol#L349
Recommendation: Add the ` payable `  keyword to the ` executeVeto `  function just like you added to the ` execute `  function.

## [L-3] Group mathematical operations with brackets to reduce ambiguity
Using brackets enhances code readability by helping communicating your intent easily other than relying on order of precedence which can lead to mistakes. Brackets helps group operations and removes ambiguity.

```
function _shares2balance(
        uint256 shares,
        uint256 sharePrice,
        uint256 deltaBalance,
        uint256 minBalance
    ) internal pure returns (uint256) {
        uint256 rebasedBalance = (shares * sharePrice) /
            START_REBASING_SHARE_PRICE +
            deltaBalance;//@audit-info add a bracket

    }
```
Brackets can be added to the last line to become
```
        uint256 rebasedBalance = ((shares * sharePrice) /
            START_REBASING_SHARE_PRICE) +
            deltaBalance;

    }
```
In the same manner the code below is in the distribute function of the same ERC20RebaseDistributor.sol file
```
       uint256 newTargetSharePrice = (amount *
                START_REBASING_SHARE_PRICE +
                __rebasingSharePrice.targetValue *
                _totalRebasingShares) / _totalRebasingShares;
```
Files: 
- https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/ERC20RebaseDistributor.sol#L266C9-L268C26
- https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/ERC20RebaseDistributor.sol#L365C13-L368C62

Recommendation: Group operands with brackets.

## [L-4] Improve _setMaxGauges function efficiency
In the ` _setMaxGauges `  setter function, there is no need to create another variable just to cache the old value before setting the new value. Instead emit the event earlier before setting the value this way and you will emit the old and new values without creating an extra ` oldMax `  variable just for caching.

```diff
    function _setMaxGauges(uint256 newMax) internal {
--     uint256 oldMax = maxGauges;
++     emit MaxGaugesUpdate(maxGauges, newMax);
        maxGauges = newMax;
--      emit MaxGaugesUpdate(oldMax, newMax);
    }
```
File: https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/ERC20Gauges.sol#L444C5-L449C6

Recommedation: Emit event first to avoid creating an extra local variable to improve efficiency like in the diff above.





