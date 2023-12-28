# 1, Checking quorum offboard should minus one

## Lines of code
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOffboarding.sol#L104
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOffboarding.sol#L138

## Details
When propose offboard, voting power start with 1 wei of voting power to initialize the storage slot:

        polls[block.number][term] = 1; // voting power
But when checking quorum, it does not minus that 1 wei of token:

        if (_weight + userWeight >= quorum) {
            canOffboard[term] = true;
        }

## Impact
Wrong number of voting power is calculated. Submited as low since it do not make big impact (1 wei of voting power)

##Mitigration step:
Checking condition should be updated to:

    -    if (_weight + userWeight >= quorum) {
    +    if (_weight + userWeight - 1 >= quorum) {
            canOffboard[term] = true;
        }



# 2, When partial repay, user can not reduce position equal to minBorrow

##Lines of code
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L527-#L531

## Details
When user try to partial repay, in `_partialRepay`, there is a checking condition to check if new borrow amount left is below minBorrow or not:

        require(
            borrowAmount - issuanceDecrease > ProfitManager(refs.profitManager).minBorrow(),
            "LendingTerm: below min borrow"
        );
The checking condition should use >= instead of >, like in `_borrow()` function:

        require(
            borrowAmount >= ProfitManager(refs.profitManager).minBorrow(),
            "LendingTerm: borrow amount too low"
        );

## Impact
User can not partial repay borrow that equal to minBorrow

##Mitigration step:
Checking condition should be updated to:

        require(
        -   borrowAmount - issuanceDecrease > ProfitManager(refs.profitManager).minBorrow(),
        +   borrowAmount - issuanceDecrease >= ProfitManager(refs.profitManager).minBorrow(),
            "LendingTerm: below min borrow"
        );

#3, Offboard proposal can be exprided sooner than expected on L2s

##Lines of code:
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOffboarding.sol#L36

## Vulnerability details
On L2s chain like Optimism, `block.number` can be different from mainnet. For example: optimism is [2 seconds](https://docs.optimism.io/chain/differences). In the `LendingTermOffboarding`, it's constant value:

    uint256 public constant POLL_DURATION_BLOCKS = 46523; // ~7 days @ 13s/block

In arbitrum, despite it have same block as mainnet, there will be delay in one minute [link](https://docs.arbitrum.io/for-devs/concepts/differences-between-arbitrum-ethereum/block-numbers-and-time)
## Impact
Offboard pool can be expired different than expected on L2s chain, which is not true conpare to documents. Although none of other L2s are mentioned except arbitrum, it will become problem if protocol deployed at other L2s like Optimism

## Tools Used
Manual review

## Recommended Mitigation Steps
It should use time to check instead of block.number


# 4, No incentive to call loan to go to auction if partial repay delay of loan is passed
## Lines of code 
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L678-#L688 
## Vulnerability details 
Auction process is flawed in the way, that there is no incentive for anyone to call the warn function, which is required before liquidations. In the auction process, at first, loan need to be called at `call()` function. Problem is, there is no incentive for anyone to call the `call` function. A bidder that calls the warn function has no guarantee, that he is the one, that actually can call liquidate, when the time has come. Therefore it would be a waste of Gas to call the warn function. This might result in a situation where nobody is willing to call warn, and therefore the loan not go to auction at all, which make interest become higher and higher, which could ultimately lead to a loss of Funds for the Lender, when the Borrower starts to accrue bad Debt. 
## Impact 
No incentive to call `call` --> loan debt keep increasing without go to auction 
Loss of funds for Lender, because Borrower might accrue bad debt 
## Recommended Mitigation Steps 
Incentivice the call of `call`, to at least pay a small amount of collateral, to ensure auction is going to happen.

# 5, Attacker can front run distribute reward from GUILD token and steal newly added rewards
## Lines of code
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/SurplusGuildMinter.sol#L114-#L212
## Vulnerability details 
Every time the `notifyPnL()` get called, if in the config, there is a part of token distribute for guild, there will be credit token distribute for guild holder:

            if (amountForGuild != 0) {
                uint256 _gaugeWeight = uint256(GuildToken(guild).getGaugeWeight(gauge));
                if (_gaugeWeight != 0) {
                    uint256 _gaugeProfitIndex = gaugeProfitIndex[gauge];
                    if (_gaugeProfitIndex == 0) {
                        _gaugeProfitIndex = 1e18;
                    }
                    gaugeProfitIndex[gauge] = _gaugeProfitIndex + (amountForGuild * 1e18) / _gaugeWeight;
                }

This enables a well-known attack vector, in which the attacker will deposit peg token to get credit token, stake them and unstake right after claim reward. 
## Impact
Not everytime user can claim reward and get profit like this, it depend alot about other factor: total credit token can be minted by `RateLimitedMinter`, total profit gained, ....., but the attack effectively steal the part of the newly added rewards
## Recommendation
Reward distribute by staking guild token should be distributed like credit token rebasing,

# 6, User can keep using old GUILD token before slash to claim gauge reward
## Lines of code 
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/GuildToken.sol#L133-#L161
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/ProfitManager.sol#L409-#L436
## Vulnerability details 
In `ProfitManager#notifyPnL()` function, reward are updated based on total gauge weight:

            if (amountForGuild != 0) {
                // update the gauge profit index
                // if the gauge has 0 weight, does not update the profit index, this is unnecessary
                // because the profit index is used to reattribute profit to users voting for the gauge,
                // and if the weigth is 0, there are no users voting for the gauge.
                uint256 _gaugeWeight = uint256(
                    GuildToken(guild).getGaugeWeight(gauge)
                );
                if (_gaugeWeight != 0) {
                    uint256 _gaugeProfitIndex = gaugeProfitIndex[gauge];
                    if (_gaugeProfitIndex == 0) {
                        _gaugeProfitIndex = 1e18;
                    }
                    gaugeProfitIndex[gauge] =
                        _gaugeProfitIndex +
                        (amountForGuild * 1e18) /
                        _gaugeWeight;
                }
            }
And `ProfitManager#claimGaugeRewards()` function claim reward for user based on profit index and gauge weight of user:

    function claimGaugeRewards(
        address user,
        address gauge
    ) public returns (uint256 creditEarned) {
        //@auditz check xem nếu removed rồi thì còn claim được không (dc nha)
        //@auditn flashloan để claim ? (reported)
        uint256 _userGaugeWeight = uint256(
            GuildToken(guild).getUserGaugeWeight(user, gauge)
        );
        if (_userGaugeWeight == 0) {
            return 0;
        }
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
            creditEarned = (_userGaugeWeight * deltaIndex) / 1e18;
            userGaugeProfitIndex[user][gauge] = _gaugeProfitIndex;
        }
        if (creditEarned != 0) {
            emit ClaimRewards(block.timestamp, user, gauge, creditEarned);
            CreditToken(credit).transfer(user, creditEarned);
        }
    }

And `SurplusGuildMinter#getRewards()` function is used to mint guild reward for user:

        if (deltaIndex != 0) {
            uint256 creditReward = (uint256(userStake.guild) * deltaIndex) /
                1e18;
            uint256 guildReward = (creditReward * rewardRatio) / 1e18;
            if (slashed) {
                guildReward = 0;
            }

            // forward rewards to user
            if (guildReward != 0) {
                RateLimitedMinter(rlgm).mint(user, guildReward);
                emit GuildReward(block.timestamp, user, guildReward);
            }
            if (creditReward != 0) {
                CreditToken(credit).transfer(user, creditReward);
            }

            // save the updated profitIndex
            userStake.profitIndex = SafeCastLib.safeCastTo160(_profitIndex);
            updateState = true;
        }
It can be seen that if user's gauge weight is not reset by `applyGaugeLoss()` function, it can keep gaining reward by old gauge weight token. And there is no incentive for user to call `applyGaugeLoss()` for other user, it will lead to other user can keep earning reward with old gauge weight

The following vuln is described below:
1, Alice stake GUILD token and gain more guild token by time passed
2, bad debt appeared, but Alice's guild token is not reset because no one call `applyGaugeLoss`
3, If new reward appear, Alice can claim reward based on these GUILD token
## Impact
User who didn't apple gauge loss can keep claiming reward, although they can not gain more guild token.
## Recommendation
User who call `applyGaugeLoss` should get some reward, like credit token, from other user, to incentive everyone reset