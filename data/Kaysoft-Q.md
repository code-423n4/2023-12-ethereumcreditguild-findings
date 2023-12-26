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


## [L-4] Unnecessary casting of uint256 variable with uint256() explicit cast function.

There are 2 insteances of this
- https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/GuildToken.sol#L154
- https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/GuildToken.sol#L158

Since the ` _userGaugeWeight `   is already declared as a ` uint256 `  so there is no need to cast it to ` uint256 `  again with the ` uint256(...) `   explicit cast function as done in the code.
On line 143 in the code below,  ` _userGaugeWeight `   is already declared as a ` uint256 _userGaugeWeight `

```
File: GuildToken.sol
133:    function applyGaugeLoss(address gauge, address who) external {
            ...
142:        // read user weight allocated to the lossy gauge
143:        uint256 _userGaugeWeight = getUserGaugeWeight[who][gauge];
144:
145:         // remove gauge weight allocation
146:        lastGaugeLossApplied[gauge][who] = block.timestamp;
147:        _decrementGaugeWeight(who, gauge, _userGaugeWeight);
148:        if (!_deprecatedGauges.contains(gauge)) {
149:            totalTypeWeight[gaugeType[gauge]] -= _userGaugeWeight;
150:            totalWeight -= _userGaugeWeight;
151:        }
152:
153:        // apply loss
154:        _burn(who, uint256(_userGaugeWeight));//@audit unnecessary uint256 cast
155:        emit GaugeLossApply(
156:            gauge,
157:            who,
158:            uint256(_userGaugeWeight),//@audit unnecessary uint256 cast
159:            block.timestamp
160:        );
161:    }
```
Recommendation: Remove the uint256(...) cast function on the ` _userGaugeWeight `  variable.

## [L-5] Many contracts inherit from Pausable through the CoreRef base contract but did not add ` whenNotPaused `  to their critical functions.

There are 4 instances of these:
File: 
- https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/core/CoreRef.sol#L12
- https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/CreditToken.sol#L18C1-L19C13
- https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/GuildToken.sol#L37
- https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/GuildVetoGovernor.sol#L25
- https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/ProfitManager.sol#L30

When a contract inherits from the Pausable contract and implements the ` pause() `  and ` unpause() `  functions, it means the contract can be paused.

However there are many functions that inherit the Pauseable contract just by inheriting the CoreRef.sol contract.
The CoreRef.sol contract implements the Pauseable mechanism by implementing the necessary ` pause() `  and ` unpause() `  functions.

All contracts that inherit from the CoreRef base contract automatically implements the Pauseable security mechanism and also have the pause and unpause functions.

The issue is that some of these contracts that implement Pauseable feature through the CoreRef.sol parent contract do not add the ` whenNotPaused `  modifier to critical functions especially functions that update state. 
For example the CreditToken.sol automatically have the Pauseable feature with the pause and unpause functions but when the contract is paused, transfers can still be made.

Recommendation: Only implement the Pauseable feature on contracts that are meant to be Pauseable and add the ` whenNotPaused `  modifier to critical user functions of the ` pauseable contracts ` .





