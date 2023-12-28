## Any one can send Users their rewards against their will
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/ProfitManager.sol#L409
Not necessarily a venerability but anyone can give users their rewards even if they don't want to collect it at the moment
```soidity
 function claimGaugeRewards(
        address user,
        address gauge
    ) public returns (uint256 creditEarned) {
...
        if (creditEarned != 0) {
            emit ClaimRewards(block.timestamp, user, gauge, creditEarned);
            CreditToken(credit).transfer(user, creditEarned);// @audit analysis anyone can call thsi function and give users their rewards dispite of user will
        }
    }
```


## Remove irrelevant piece of code

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildVetoGovernor.sol#L164
As they are already declared in returns

```diff
    function proposalVotes(
        uint256 proposalId
    )
        public
        view
        virtual
        returns (uint256 againstVotes, uint256 forVotes, uint256 abstainVotes)
    {
        // againstVotes are supporting the execution of Veto
        againstVotes = _proposalVotes[proposalId].againstVotes;
        // no forVotes can be cast in the Veto module, keep 0 value
--       forVotes = 0;// @audit no need to make it 0 as by default they will be zeiro as defined in retuns
        // no abstainVotes can be cast in the Veto module, keep 0 value
--        abstainVotes = 0;
    }
```

## Use `1e17` instead of`0.1e18` to improve readablity

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOnboarding.sol#L144


```diff
-- params.openingFee <= 0.1e18, // open fee expected [0, 10]%
++ params.openingFee <= 1e17, // open fee expected [0, 10]%
```


