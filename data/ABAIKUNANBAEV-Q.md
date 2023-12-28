## Findings Summary

| ID | Description | Severity |
| :-: | - | :-: |
| [L-01](#l-01-consider-adding-the-whennotpaused-modifier-to-slashandreward) | The user tx may unexpectedly revert when repaying| Low |
| [L-02](#l-02-consider-adding-the-whennotpaused-modifier-to-migrate) | Using `getPastVotes()` on Arbitrum can lead to inconsistencies | Low |



## [L-01]  The user tx may unexpectedly revert when repaying

In `LendingTerm.sol`, there is a function `_partialRepay()` that is used by a borrower to partially repay the debt.
This function doesn't allow to fully repay the debt:

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L509
```
 require(debtToRepay < loanDebt, "LendingTerm: full repayment");
```
However, there is a `minBorrow()` param that is used to make sure that the debt amount is still above a certain level after repayment:

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L527-531

```
    require(
            borrowAmount - issuanceDecrease >
                ProfitManager(refs.profitManager).minBorrow(),
            "LendingTerm: below min borrow"
        );
```

The user tx can unexpectedly revert if the amount after repayment is lower than `minBorrow()`.

### Recommendaton

It's recommended to have fixed partial repayment amounts.


## [L-02]  Using `getPastVotes()` on Arbitrum can lead to inconsistencies

When calling `supportOffboard()`  in `LendingTermOffboarding`, `block.number` is used to determine how much weight the user has at a given moment of time. The situation can lead to some mistakes when executing transaction on Arbitrum as it returns the last synched `block.number` with L1. The time of synchronization is about 1 minutes and such time difference can be abused when using `getPastVotes()`:

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOffboarding.sol#L126-128

```
   uint256 userWeight = GuildToken(guildToken).getPastVotes(
            msg.sender,
            snapshotBlock
        );
```

### Recommendaton

The user's past votes should be determined using block.timestamp.


