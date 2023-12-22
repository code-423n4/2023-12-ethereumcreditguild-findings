### Report 1:
#### Wrong State Representation
Assigning the value of "totalTypeWeight" as "totalWeight" in the debtCeiling(...) function is wrong and should be corrected as totalWeight could be misinterpreted as the general total weight instead of the specific gauge type weight.
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L279
```solidity
 function debtCeiling(
        int256 gaugeWeightDelta
    ) public view returns (uint256) {
        address _guildToken = refs.guildToken; // cached SLOAD
        uint256 gaugeWeight = GuildToken(_guildToken).getGaugeWeight(
            address(this)
        );
        gaugeWeight = uint256(int256(gaugeWeight) + gaugeWeightDelta);
        uint256 gaugeType = GuildToken(_guildToken).gaugeType(address(this));
---        uint256 totalWeight = GuildToken(_guildToken).totalTypeWeight(
+++        uint256 totalTypeWeight = GuildToken(_guildToken).totalTypeWeight(
            gaugeType
        );
...
```
This should also be adjusted where necessary in the function
### Report 2:
#### Unnecessary Precaution to Prevent Precision Loss
The initial assignment of 1e18 to "_gaugeProfitIndex" & "_userGaugeProfitIndex" in the claimGaugeRewards(...) function in the ProfitManager.sol contract as provided below, this is done when the value is zero, sponsor said it is used to handle precision but at no point are this two variables used directly in a multiplication or division operation, which shows there is no way precision can be lost, the only use of this variables can be seen in the subtraction calculation used to derive "deltaIndex", just as noted to sponsor, 
deltaIndex = (100 + 1e18) - ( 20 + 1e18) is the same thing as 100 - 20 with no precision lose. Necessary adjustments should be made, This is because Solidity uses fixed-point arithmetic for numbers, and the precision is determined by the type of the variable unlike other programming languages like javascript etc
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/ProfitManager.sol#L422-L427
```solidity
 function claimGaugeRewards(
        address user,
        address gauge
    ) public returns (uint256 creditEarned) {
        uint256 _userGaugeWeight = uint256(
            GuildToken(guild).getUserGaugeWeight(user, gauge)
        );
        if (_userGaugeWeight == 0) {
            return 0;
        }
        uint256 _gaugeProfitIndex = gaugeProfitIndex[gauge];
        uint256 _userGaugeProfitIndex = userGaugeProfitIndex[user][gauge];
        if (_gaugeProfitIndex == 0) {
>>>            _gaugeProfitIndex = 1e18;
        }
        if (_userGaugeProfitIndex == 0) {
 >>>           _userGaugeProfitIndex = 1e18;
        }
>>>        uint256 deltaIndex = _gaugeProfitIndex - _userGaugeProfitIndex;
        if (deltaIndex != 0) {
            creditEarned = (_userGaugeWeight * deltaIndex) / 1e18;
            userGaugeProfitIndex[user][gauge] = _gaugeProfitIndex;
        }
```


