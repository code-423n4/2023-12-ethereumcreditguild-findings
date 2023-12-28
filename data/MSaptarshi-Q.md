# [L -01] Function call not returned 
## Context 
The function is not present where the contract is calling the function
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/SimplePSM.sol#L121
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/SurplusGuildMinter.sol#L136
## Recommendation
Change the contract where we are calling the function


# [NC-01] Smart contract code format not correct in many cases throughout the protocol 
## Context
functions are declared before variables
## Recommendation
Solidity recommends this code format
https://docs.soliditylang.org/en/latest/style-guide.html

# [NC-02] function not marked payable 
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/ERC20MultiVotes.sol#L72
## Context
functions that are related with token checking or transfer or mint/ burn not marked payable
## Recommendation
Mark those functions payable