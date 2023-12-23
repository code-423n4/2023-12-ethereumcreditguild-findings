## Incorrect number of seconds in POLL_DURATION_BLOCKS variable 


## IMPACT
In the contract `LendingTermOffboarding.sol` , there is a state variable named `POLL_DURATION_BLOCKS` i.e. 

```
uint256 public constant POLL_DURATION_BLOCKS = 46523; // ~7 days @ 13s/block

```
The given variable is calculated based on 13seconds per block. However, as per recent Ethereum's average block time, it should be: 12 seconds per block. 
Reference: https://etherscan.io/chart/blocktime

This state variable is used in functions like `proposeOffboard` and `supportOffboard` 
in important require checks, so it should be initialized correctly. 
Therefore, the given state variable should be initialized with correct value, i.e. 
86,400 seconds per day x 7 days / 12 seconds per block = 50,400 


Also, in the contract `GuildVetoGovernor.sol`, there is a function `votingPeriod` i.e. 

```
    function votingPeriod() public pure override returns (uint256) {
        return 2425847; // ~1 year with 1 block every 13s
    }
```

It has been calculated using 13 seconds per block which is incorrect. 
It should actually return: 

86,400 seconds per day x 365.25 days(leap year) = 31557600 seconds per year 

31557600 seconds / 12 seconds per block = 2629800


## Proof of Concept: 
`POLL_DURATION_BLOCKS`
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/LendingTermOffboarding.sol#L36


`proposeOffboard` 
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/LendingTermOffboarding.sol#L94-L97

`supportOffboard`
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/LendingTermOffboarding.sol#L120-L123

`votingPeriod`
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/GuildVetoGovernor.sol#L230


## MITIGATION
Consider making the appropriate changes i.e. set the `POLL_DURATION_BLOCKS` to 50,400 and the function `votingPeriod` should be changed to 2629800
