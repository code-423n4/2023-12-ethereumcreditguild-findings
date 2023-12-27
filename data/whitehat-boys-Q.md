# L-01 Anyone could increase interpolation endTime by 30 days just by distributing 1 wei.

## Lines of code

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20RebaseDistributor.sol#L364

## Impact
Anyone can call distribute with 1 Wei as amount and update the interpolation of `__rebasingSharePrice` & `__unmintedRebaseRewards` by 30 days 

## Tools Used
manual review
## Recommended Mitigation Steps
Add a minimum amount check to the distribute function

# L-02 Lack of input validations for uint inputs

## Lines of code

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildGovernor.sol#L78
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildGovernor.sol#L85
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/SurplusGuildMinter.sol#L319
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/SurplusGuildMinter.sol#L328

## Impact
lack of input validation would delay reverting of a transaction for inavlid inputs so ideally a tx could be reverted early and gas could be saved

## Tools Used
manual review
## Recommended Mitigation Steps
add missing input validation checks

# L-03 `nonRebasingSupply` should revert if `_rebasingSupply > _totalSupply`

## Lines of code
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20RebaseDistributor.sol#L399
## Impact
It is mentioned in the comments that `nonRebasingSupply` should revert if `_rebasingSupply > _totalSupply` but it is not the case and instead 0 is returned

## Tools Used
manual review
## Recommended Mitigation Steps
Revert when `_rebasingSupply > _totalSupply`

# L-04 `delegateBySig` does not use the delegate address to recover the signer from the signature

## Lines of code
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20MultiVotes.sol#L471

## Impact
`delegateBySig` does not uses the `delegate` address in the method, although there is a check that signer address is non-zero but it would be better in terms of the good practices in using permit to pass the `delegate` address

## Tools Used
manual review
## Recommended Mitigation Steps
Update the method signature to include the delegate address

# L-05 No events emitted in some critical functions

## Lines of code

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildVetoGovernor.sol#L315
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L829
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L846
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/SurplusGuildMinter.sol#L293

## Impact
Events are necessary for these functions for better logging since there are multiple things happening in each method in the contract

## Tools Used
manual review
## Recommended Mitigation Steps
add missing events

# L-06 A term can be added even if the contract is paused

## Lines of code

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOnboarding.sol#L105

## Impact
All critical operations should be paused once the contract is paused, but in this case, onboarding a lending term isn't

## Tools Used
manual review
## Recommended Mitigation Steps
prevent a lending term from being added when a contract is paused