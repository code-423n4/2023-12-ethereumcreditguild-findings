## [L-1] The ` executeVeto `  function of the GuildVetoGovernor.sol should be ` payable ` 

There is 1 instance of this

- https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/GuildVetoGovernor.sol#L349

 The   ` executeVeto `  is only a convinience function for the standard ` execute `   function but lacks the ` payable `  keyword so every call with msg.value greater than 0 made to the ` executeVeto `  will revert. 

This will make the ` executeVeto `  unusable for transaction executions that include ETH values.

``` 
File: GuildVetoGovernor.sol
349: function executeVeto(bytes32 timelockId) external returns (uint256) {//@audit this should be payable.
```

Recommendation: Add the ` payable `  keyword to the ` executeVeto `  function just like you added to the ` execute `  function.



## [L-2] Group mathematical operations with brackets to reduce ambiguity

There are 2 instances of this

- https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/ERC20RebaseDistributor.sol#L266C9-L268C26
- https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/ERC20RebaseDistributor.sol#L365C13-L368C62


Using brackets enhances code readability by helping communicating your intent easily other than relying on order of precedence which can lead to mistakes. Brackets helps group operations and removes ambiguity.

```
File: ERC20RebaseDistributor.sol
260:  function _shares2balance(
261:         uint256 shares,
262:         uint256 sharePrice,
263:         uint256 deltaBalance,
264:         uint256 minBalance
265:         ) internal pure returns (uint256) {
266:         uint256 rebasedBalance = (shares * sharePrice) /
267:          START_REBASING_SHARE_PRICE +
268:         deltaBalance;//@audit-info group operands with brackets

273:    }
```

```
File: ERC20RebaseDistributor.sol
338: function distribute(uint256 amount) external {   
    ...
365:        uint256 newTargetSharePrice = (amount *
366:        START_REBASING_SHARE_PRICE +
367:        __rebasingSharePrice.targetValue *
368:        _totalRebasingShares) / _totalRebasingShares; //@audit-info group operands with brackets.
```
Recommendation: Group operands with brackets.


## [L-3] Improve _setMaxGauges function efficiency

There is 1 instance of this

-  https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/ERC20Gauges.sol#L444C5-L449C6

In the ` _setMaxGauges `  setter function, there is no need to create another variable just to cache the old value before setting the new value. Instead emit the event earlier before setting the value this way and you will emit the old and new values without creating an extra ` oldMax `  variable just for caching.

```diff
     function _setMaxGauges(uint256 newMax) internal {
--        uint256 oldMax = maxGauges;
++        emit MaxGaugesUpdate(maxGauges, newMax);
          maxGauges = newMax;
--        emit MaxGaugesUpdate(oldMax, newMax);
    }
```

Recommedation: Emit event first to avoid creating an extra local variable to improve efficiency like in the diff above.





