# [NC-01] Smart contract code format not correct in many cases throughout the protocol 
## Context
functions are declared before variables
## Recommendation
Solidity recommends this code format
https://docs.soliditylang.org/en/latest/style-guide.html

# [NC-02] function not marked payable 
## Context
functions that are related with token checking or transfer or mint/ burn not marked payable
## Recommendation
Mark those functions payable