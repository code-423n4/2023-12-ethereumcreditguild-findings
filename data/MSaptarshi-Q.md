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