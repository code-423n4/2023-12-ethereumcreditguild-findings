In the `Profitmananger.sol` contract, there is a precision loss owing to performing division before multiplication, when the `GOVERNOR` calls `setProfitSharingConfig(...)` to set the `profitSharingConfig` each of the config values are divided by 1e9, however in the `getProfitSharingConfig()` function, each of the config values are read a multiplied by 1e9. This finding is being reported as a low because the output of the `getProfitSharingConfig()` function is not used in other parts of the codebase and as such the precision loss is not propagated by its use but the output may be use by external actors.

```solidity
function setProfitSharingConfig(
        uint256 surplusBufferSplit,
        uint256 creditSplit,
        uint256 guildSplit,
        uint256 otherSplit,
        address otherRecipient
    ) external onlyCoreRole(CoreRoles.GOVERNOR) {
        if (otherRecipient == address(0)) {
            require(otherSplit == 0, "GuildToken: invalid config");
        } else {
            require(otherSplit != 0, "GuildToken: invalid config");
        }
        require(
            surplusBufferSplit + otherSplit + guildSplit + creditSplit == 1e18,
            "GuildToken: invalid config"
        );
        // @audit dividing by 1e9 already introduces a precision loss wh
        profitSharingConfig = ProfitSharingConfig({
            surplusBufferSplit: uint32(surplusBufferSplit / 1e9),
            guildSplit: uint32(guildSplit / 1e9),
            otherSplit: uint32(otherSplit / 1e9),
            otherRecipient: otherRecipient
        });

        emit ProfitSharingConfigUpdate(
            block.timestamp,
            surplusBufferSplit,
            creditSplit,
            guildSplit,
            otherSplit,
            otherRecipient
        );
    }
```

```solidity
function getProfitSharingConfig()
        external
        view
        returns (
            uint256 surplusBufferSplit,
            uint256 creditSplit,
            uint256 guildSplit,
            uint256 otherSplit,
            address otherRecipient
        )
    {
        // @audit precision loss
        surplusBufferSplit =
            uint256(profitSharingConfig.surplusBufferSplit) * 1e9;
        guildSplit = uint256(profitSharingConfig.guildSplit) * 1e9;
        otherSplit = uint256(profitSharingConfig.otherSplit) * 1e9;
        
        creditSplit = 1e18 - surplusBufferSplit - guildSplit - otherSplit;
        otherRecipient = profitSharingConfig.otherRecipient;
    }
```

## Recommendation

Consider removing the division by 1e9 when setting the `profitSharingConfig` and make up for that scaling in other parts of the codebase where the config is used.