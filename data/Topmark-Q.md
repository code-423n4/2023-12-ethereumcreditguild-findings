### Report 1:
#### Wrong State Representation
Assigning the value of "totalTypeWeight" as "totalWeight" in the debtCeiling(...) function is wrong and should be corrected as totalWeight could be misinterpreted as the general total weight instead of the specific gauge type weight.
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
