
## Potential Loss of Assets in emergencyAction Function due to Unchecked msg.value


Contract Name: CoreRef 

Scope: https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/core/CoreRef.sol#L87-L107

### Impact 

During the security audit of the smart contract, a critical issue has been identified in the `emergencyAction` function, particularly in its handling of native coin transfers to external contracts (i.e. targets). The function allows the caller to send `msg.value` to multiple targets without ensuring that the total value sent does not exceed the targets' contributions. This oversight may lead to the loss of excess assets that remain inaccessible within the contract.

In other words, The `emergencyAction` function allows the caller to send native coins (msg.value) to multiple external contracts without verifying that the total value sent does not exceed the summation of the values sent to the targets. The excess native coins sent beyond the summation of values to the targets remain locked in the contract resulting in potential asset loss.


### Proof of Concept

https://gist.github.com/beber89/cc6129be4d29a12c9e53d5ce0844f4cf


### Recommended Mitigation Steps

To mitigate the risks associated with unchecked `msg.value` in the `emergencyAction` function, it is recommended to:

Implement Validation Checks: Introduce a validation checks within the `emergencyAction` function to ensure that the total `msg.value` sent does not exceed the intended amount for external calls. No need for another for-loop to carry out that check, leverage the existing for loop and carry out the check after the loop is executed.

Refund Mechanism: Consider implementing a refund mechanism to return excess `msg.value` to the caller if it exceeds the total required for external calls. This enhances transparency and prevents asset loss.



## Unspent Native Coin gets rejected on calling emergencyAction function


Contract Name: CoreRef 

Scope: https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/core/CoreRef.sol#L87-L107

### Impact 

During the security audit of the smart contract, a potential vulnerability has been identified in the `emergencyAction` function. This function performs low-level calls to external contracts, including the transfer of native coins (`msg.value`). The audit reveals that the implementation does not account for the possibility that external contracts might return unspent native coins, leading to potential loss of assets. It is a common pattern in which contracts accept native coin payments and return the unspent coin back to the caller.

### Proof of Concept

https://gist.github.com/beber89/cc6129be4d29a12c9e53d5ce0844f4cf


### Recommended Mitigation Steps

Implement Asset Recovery Mechanism: Incorporate a mechanism within the `emergencyAction` function to handle and recover native coins that are rejected or returned by external contracts to then be returned to `msg.sender`. This also requires to add `receive()` method to the contract implementation.


## Use of assert Instead of require in Smart Contract 

Contract Name: LendingTerm

Scope: https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L167

### Impact

The developers have utilized assert statement in initialize function in line 167 of the code where require may be more suitable. Unlike require, which is used to validate user inputs and conditions, assert is typically employed to check for internal errors and invariant conditions. Inappropriate use of assert can lead to unnecessary loss of funds.


```solidity
    function initialize(
        address _core,
        LendingTermReferences calldata _refs,
        LendingTermParams calldata _params
    ) external {
        // can initialize only once
        assert(address(core()) == address(0));
        assert(_core != address(0));
        ...
    }
```


Another negative impact is Inadequate Error Messaging: Unlike require, which provides a clear error message indicating the cause of the failure, assert simply terminates the contract without detailed information. This lack of informative error messaging hinders developers and users in understanding the root cause of contract termination.
 
### Recommended Mitigation Steps


Replace assert with require: Utilize require for user input validation and condition checking where failure should be gracefully handled with informative error messages.

Implement Custom Error Messaging: Enhance error messaging in require statements to provide clear information about the conditions that need to be met and the consequences of failure.


## CEI Should Be Followed

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOnboarding.sol#L114

In the above instance , an external call is performed before updating the state at L154 and L165.
It is crucial to  follow CEI throughout the codebase to prevent from attacks such as reentrancy (even if not exploitable).

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/ProfitManager.sol#L260

transfer done before state updates.

## Add A Min Check On setGaugeWeightTolerance

The gaugeWeightTolerance is used to facilitate growth of the protocol and is expressed as a percentage with 18 decimals , 
i.e. 1e18 means 0% deviation and this might lead to a deadlock situation where no new borrows are allowed.
There should be a minimum check here

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/ProfitManager.sol#L190

i.e. require(gaugeWeightTolerance > 1e18) to ensure a healthy weight tolerance.

## surplusBufferSplit/guildSplit/othersSplit Might Round Down To 0

The governor can set the profit sharing config here 

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/ProfitManager.sol#L195

If the current system requirements were to keep `otherSplit` to be as low as possible (for example) and maximise the guild and
supplyBuffer split , say `otherSplit` is 1e8 , then otherSplit would round down to 0
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/ProfitManager.sol#L215
Because of this the split that should have been assigned to `otherAddress` would be lost forever.

A possible remediation for this can be to ensure otherSplit/1e9 > 0 or have a minimum split requirement.

## A Market Can Not Be Deployed For A Token > 18 Decimals

Each market has its own SimplePSM (peg stability module) , and inside the constructor of the SimplePSM the decimalCorrection is 
calculated as follows ->

` decimalCorrection = 10 ** (18 - decimals);`

In case of USDC as the peg token it would be 10 ** 12

It is evident that it would be impossible to deploy a SimplePSM for a token with decimals > 12 since the constructor would keep reverting
due to the underflow in 10 ** (18 - decimals);

To remediate , make sure if the decimal of the peg token is more than 18 then it is normalised properly.


## Have A Minimum Value For maxDelayBetweenPartialRepay

A loan can be called if a partial repayment is missed , the check to see if a partial repayment is missed is done here ->

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L654

The code to check  if a partial repayment is missed ->

```solidity

lastPartialRepay[loanId] <
            block.timestamp - params.maxDelayBetweenPartialRepay;
            
 ```
 
 If the maxDelayBetweenPartialRepay was set maliciously to a very low value then an attacker can deliberately make
 the victim miss a partial repayment so that his loan goes off to auction , to make the victim miss a partial repayment the
 attacker can perform block stuffing just before the victim's call to partial repay.
 
 To remediate make sure there is a minimum value for the maxDelayBetweenPartialRepay of a loan.
       


## Missing 0 Address/Value Checks (Missed By The Bots)

There are instances where 0-address checks are missing for variables set in the constructor that are missed by the 
automation . These are ->

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOnboarding.sol#L83-L88
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/ProfitManager.sol#L182
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildVetoGovernor.sol#L73
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20MultiVotes.sol#L149

A zero address in a crucial role can cause serious issues, as it cannot perform actions like a normal address, and any funds sent to it will be irretrievable. 
 

## Incorrect Comment

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/ProfitManager.sol#L58

The comment above says if the sum of `guildSplit` + `otherSplit` is < 1e9 then remaining is sent to the lenders of the system.
Rather it should be if the sum of `guildSplit` + `otherSplit` + `surplusBufferSplit` is < 1e9 , since surplusBuffer is also a part 
of the split.

## Handle The 0 surplusBuffer Case In The If Clause

Inside the function notifyPnL() , there can arise a condition where loss is equal to the surplus buffer (in this case the surplus
buffer would be 0) , currently this case is handled in the else clause here 
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/ProfitManager.sol#L323

but it would be more effecient to handle  this case in the if block. This would be done by changing the if condition to
`if (loss <= _surplusBuffer)`

This would be more streamlined plus less gas as else has more code to run.

## Sanity Checks

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20MultiVotes.sol#L349

A sanity check can be added before this , `require(amount >= _delegatesVotesCount[delegator][delegatee])`

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20MultiVotes.sol#L451

This is wrapped inside a require, but this can never fail due to the code at L437 (delegateList only contains the delegatees inside the
`_delegates[user]`)


## Make Sure CREDIT Can Not Be Redeemed To The SimplePSM Itself

CREDIT can be redeemed for underlying token in the SimplePSM.sol , 
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/SimplePSM.sol#L135
But if the `to` address is provied as address(this) , then the redeemed CREDIT token would go back to the SimplePSM contract and the 
pegTokenBalance would be reduced at L141 whereas it remained the same. 
Make sure `to` is not the address(this).
