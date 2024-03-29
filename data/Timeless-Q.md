
## [LOW-01] Incorrect number of seconds in POLL_DURATION_BLOCKS variable 


## Impact
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


## Proof of Concept: 
`POLL_DURATION_BLOCKS`
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/LendingTermOffboarding.sol#L36


`proposeOffboard` 
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/LendingTermOffboarding.sol#L94-L97

`supportOffboard`
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/LendingTermOffboarding.sol#L120-L123


## Recommended Mitigation Steps
Consider making the appropriate changes i.e. set the `POLL_DURATION_BLOCKS` to 50,400 




## [LOW-02] Decreasing maxGauges does not account for `users` previous Gauge list size

## Description
`ERC20Gauges` contract has a `maxGauges` state variable meant to represent the maximum amount of gauges a user can allocate to. 
As per the natspec, it is meant to protect against gas DOS attacks upon token transfer to allow complicated transactions to fit in a block. There is also a function `setMaxGauges` for authorized users to decrease or increase this state variable.


## Proof of Concept
`# Function _incrementGaugeWeight`
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/ERC20Gauges.sol#L236


https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/ERC20Gauges.sol#L444-L449
**2023-12-ethereumcreditguild/src/tokens/ERC20Gauges.sol**
```
 function _setMaxGauges(uint256 newMax) internal {
        uint256 oldMax = maxGauges;
        maxGauges = newMax;

        emit MaxGaugesUpdate(oldMax, newMax);
    }
```

However, if it is decreased and there are users that have already reached the previous maximum that was larger, there may be unexpected behavior.
All of these users gauges will remain active and manageable, such as have user's existing gauge weights incremented or decremented. However, they wont be able to add new gauges. 

While in the current contract logic this does not cause issues, `maxGauges` is a public variable that may be used by other systems. If unaccounted for, this discrepancy between the contract’s `maxGauges` and the users’ actual number of gauges given by `_userGauges[user].length()` could, for example, cause gauges to be skipped or fail loops bounded by `maxGauges` in other systems’ logic that try and go through all user gauges.


## Recommended Mitigation Steps
Either document the potential discrepancy between the user gauges size and the `maxGauges` state variable, or limit `maxGauges` to be only called within the contract thereby forcing other contracts to retrieve user gauge list size through `_userGauges[user].length()`


## [LOW-03] Incorrect changes to the function `_setCanExceedMaxGauges` in contract `ERC20Gauges`

## Description: 

In the contract `ERC20Gauges.sol`  
In the comments, the @dev mentions in beginning that, 
**ECG made the following changes to the original flywheel-v2 version:**
One of the **changes** to be made is: 

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/ERC20Gauges.sol#L42-L45
```
- Remove public setContractExceedMaxGauges(address, bool) requiresAuth method
    ... Add internal _setCanExceedMaxGauges(address, bool) method
     ... Remove check of "target address has nonzero code size"
    ... Rename to remove "contract" from name because we don't check if target is a contract
    
```

It seems the intention here is: 
To remove the require check, because in the subsequent line it is mentioned `because we don't check if target is a contract`. 
Meaning, we don't want to check whether the address(parameter) is a contract or not. So, that would mean, we should NOT include the require check. 


However, in the function `_setCanExceedMaxGauges`, the require check still exists i.e. 
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/ERC20Gauges.sol#L451-L466

```
function _setCanExceedMaxGauges(
        address account,
        bool canExceedMax
    ) internal {
        if (canExceedMax) {
        
            require(
                account.code.length != 0,
                "ERC20Gauges: not a smart contract"
            );  // This require check should be removed  
        }

```

This would lead to undesired behavior. Attempting to set whether an Externally Owned Account (EOA) could exceed the Max Gauges limit would result in this check failing each time it is called. 

## Recommended Mitigation Steps
Consider removing the given require statement.


