[L-1]  Loss-experienced `gauge` can prevent the transfer of tokens.

When a user attempts to transfer tokens and there are insufficient free tokens, the system gradually decreases the weight from the gauges one by one until a sufficient number of free tokens becomes available.
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/ERC20Gauges.sol#L524
```
function _decrementWeightUntilFree(address user, uint256 weight) internal {
    for () {
        _decrementGaugeWeight(user, gauge, userGaugeWeight);
    }
}
```
If the gauge experiences any `loss`, and the user has assigned weight to this gauge, it indicates that the user has not applied `loss` yet. 
Therefore, the below function will always revert in such cases.
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/GuildToken.sol#L214-L217
```
function _decrementGaugeWeight(
    uint256 _lastGaugeLoss = lastGaugeLoss[gauge];
    uint256 _lastGaugeLossApplied = lastGaugeLossApplied[gauge][user];
    require(
        _lastGaugeLossApplied >= _lastGaugeLoss,
        "GuildToken: pending loss"
    );

}
```
Actually, the `guild` tokens that were added to the loss-experienced `gauges` are not considered free tokens. 
When a user applies `loss`, these tokens are removed from the user, and they do not contribute to increasing the free tokens required for a transfer. 
Therefore, in the `_decrementWeightUntilFree` function, it is necessary to skip the loss-experience `gauges`. 
This action helps prevent undesired DoS scenarios in token transfers.

I am uncertain whether this falls into the category of medium risk.

[L-2]  We are using tokens in the `Surplus buffer` of a `term` only for one `loss` that originates from that specific `term`.

Users have the option to select a healthy `term` and stake in that `term` using `SurplusGuildMinter`. 
Subsequently, the credit tokens of these users are contributed to the `Surplus Buffer` for that specific `term`.
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/SurplusGuildMinter.sol#L130
```
function stake(address term, uint256 amount) external whenNotPaused {
    ProfitManager(profitManager).donateToTermSurplusBuffer(term, amount);
}
```
The accumulated amounts can be substantial, given that many users stake in the same `term`. 
However, even a minor `loss` in that `term` can lead to these amounts being moved from the `Surplus Buffer` for that `term` and made available for use across all `terms`.
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/ProfitManager.sol#L309-L313
```
function notifyPnL() {
    if (_termSurplusBuffer != 0) {
        termSurplusBuffer[gauge] = 0;
        _surplusBuffer += _termSurplusBuffer;
    }
}
```
Additionally, users are unable to withdraw their credit tokens once staked. 
I guess the idea behind this solution is likely to simplify the process of redistributing staked credit tokens in `SurplusGuildMinter` when users decide to unstake. 
However, it may be feasible to explore redistribution based on the proportion of their stake amounts.

In the event of a `loss` occurring in that `term`, if the `surplus buffer` contains enough tokens, we can straightforwardly deduct the `loss` from it. 
This proposed modification could enhance the protocol in alignment with user preferences.

[L-3] The weight of a deprecated `gauge` is excluded from the debt ceiling calculation, but the issuance of a deprecated gauge is still taken into account in the debt ceiling calculation.

When a `gauge` is removed, its weight is subtracted from the total weight. 
As a result, it does not impact the debt ceiling calculation.
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/ERC20Gauges.sol#L434-L437
```
function _removeGauge(address gauge) internal {
    uint256 weight = getGaugeWeight[gauge];  
    if (weight != 0) {
        totalTypeWeight[gaugeType[gauge]] -= weight;
        totalWeight -= weight;
    }
}
```
However, the amount already issued from the deprecated `gauge` continues to influence the debt ceiling calculation.
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L294-L295
```
function debtCeiling() {
    uint256 totalBorrowedCredit = ProfitManager(refs.profitManager).totalBorrowedCredit();
}
```
The same applies to the `_borrow` function.
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L379-L380
```
function _borrow() {
    uint256 totalBorrowedCredit = ProfitManager(refs.profitManager).totalBorrowedCredit();
}
```
It has the potential to disrupt the balance in the possible issuance amount across `terms`.

[L-4] It is possible to delegate `0` tokens or add `0` gauge weight

Any user can delegate `0` tokens to others.
There is no check to verify whether the delegated amount is `0` or not.
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/ERC20MultiVotes.sol#L264-L269
```
function incrementDelegation(address delegatee, uint256 amount) public virtual {
    _incrementDelegation(msg.sender, delegatee, amount);
}
```
There is a capped limit on the amount a user can delegate. 
When a user delegates 0 tokens, it signifies that they are not delegating, but it is still recorded as the user utilizing one possible delegation
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/ERC20MultiVotes.sol#L329
```
function _incrementDelegation() {
    bool newDelegate = _delegates[delegator].add(delegatee); 
}
```
The same logic applies to the addition of gauge weight.
We should include a check for a 0 amount.

[L-5] Enhance the `_decrementWeightUntilFree` function.

To ensure a sufficient number of free tokens for a transfer, we reduce the weight from `gauges` one by one.
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/ERC20Gauges.sol#L521-L535
```
function _decrementWeightUntilFree(address user, uint256 weight) internal {
    for (uint256 i = 0; i < size && (userFreeWeight + userFreed) < weight;) {
        uint256 userGaugeWeight = getUserGaugeWeight[user][gauge];
        _decrementGaugeWeight(user, gauge, userGaugeWeight);
    }
}
```
However, in this process, we have the opportunity to skip certain gauges. 
In some `gauges`, attempting to reduce the weight may result in a revert due to the issuance check.
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/GuildToken.sol#L225-L230
```
function _decrementGaugeWeight() {
    if (issuance != 0) {
        uint256 debtCeilingAfterDecrement = LendingTerm(gauge).debtCeiling(-int256(weight));
        require(issuance <= debtCeilingAfterDecrement, "GuildToken: debt ceiling used");
    }
}
```
We can obviously skip these `gauges`.
Moreover, it is preferable to remove the weight from the deprecated `gauges`.

[L-6] `SimplePSM` is not compatible with tokens whose decimals exceed `18`.

If the decimals of `_pegToken` exceed `18`, deploying `SimplePSM` for that token is not feasible.
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/SimplePSM.sol#L75-L76
```
constructor() {
    uint256 decimals = uint256(ERC20(_pegToken).decimals());
    decimalCorrection = 10 ** (18 - decimals);
}
```

[L-7] When the shares become `0`, any unminted `rewards` will be lost.

In `ERC20RebaseDistributor`, when the shares become `0`, all values associated with the unminted `rewards` and distribution speed are reset.
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/ERC20RebaseDistributor.sol#L190-L201
```
function updateTotalRebasingShares() {
    if (sharesAfter == 0) {
        __rebasingSharePrice = InterpolatedValue({}); // reset
        __unmintedRebaseRewards = InterpolatedValue({}); // reset       
    }
}
```

[L-8] Please use `targetTotalSupply` instead of `totalSupply`.

When applying `loss`, we calculate the `newCreditMultiplier` based on the `loss` and total supply.
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/ProfitManager.sol#L331-L333
```
function notifyPnL() {
    uint256 creditTotalSupply = CreditToken(_credit).totalSupply();
    uint256 newCreditMultiplier = (creditMultiplier *
        (creditTotalSupply - loss)) / creditTotalSupply;
}
```
If there are unminted tokens, the correct total supply should be `targetTotalSupply`.

[L-9] Any borrower can receive `rewards` by adding weight to the `term` before repayment.

When interests accrue from borrowers, these interests are immediately distributed to token holders based on their respective weights.
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/ProfitManager.sol#L396-L399
```
function notifyPnL() {
    gaugeProfitIndex[gauge] = _gaugeProfitIndex + (amountForGuild * 1e18) / _gaugeWeight;
}
```
Hence, any borrower can add weight before repayment, receive `rewards`, and subsequently withdraw their added weight.
Need a logic similar to rebasing tokens.
