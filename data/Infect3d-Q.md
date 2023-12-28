
# [L-01] - If one loan is partiallyRepaid, callMany will fail

People calling loans using `callMany` would see all the calls revert if any of these is partially repaid right before in the same block.
Would be a better UX to not revert for failed `_call` but just to pass them.
This can be done using the low level `call("")` function.

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L683
```solidity
File: src\loan\LendingTerm.sol
683:     function callMany(bytes32[] memory loanIds) public {
684:         address _auctionHouse = refs.auctionHouse;
685:         for (uint256 i = 0; i < loanIds.length; i++) {
686:             _call(msg.sender, loanIds[i], _auctionHouse);	//@audit-issue L01 - If one loan is partiallyRepaid, callMany will fail. This should pass failed calls
687:         }
688:     }
```

# [L-02] -`addCollateral` has no usecase, increasing the collateral in a loan give no advantage to the borrower

A user could add collateral thinking he will make his loan more healthy, while its not the case, as this has no positive effect for him.

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L448
```solidity
File: src\loan\LendingTerm.sol448:     function _addCollateral(
449:         address borrower,
450:         bytes32 loanId,
451:         uint256 collateralToAdd
452:     ) internal {
453:         require(collateralToAdd != 0, "LendingTerm: cannot add 0");
454: 
455:         Loan storage loan = loans[loanId];
456: 
457:         // check the loan is open
458:         require(loan.borrowTime != 0, "LendingTerm: loan not found");
459:         require(loan.closeTime == 0, "LendingTerm: loan closed");
460:         require(loan.callTime == 0, "LendingTerm: loan called");
461: 
462:         // update loan in state
463:         loans[loanId].collateralAmount += collateralToAdd; 	//@audit-issue L02 - addCollateral is pointless, only action is to put more collateral at risk with no return
464: 
465:         // pull the collateral from the borrower
466:         IERC20(params.collateralToken).safeTransferFrom(
467:             borrower,
468:             address(this),
469:             collateralToAdd
470:         );
471: 
472:         // emit event
473:         emit LoanAddCollateral(
474:             block.timestamp,
475:             loanId,
476:             borrower,
477:             collateralToAdd
478:         );
479:     }
```

# [L-03] No lower bound for `minPartialRepayPercent` in `LendingTermOnboarding` can lead to un-callable loans

No lower bound for `minPartialRepayPercent` in `LendingTermOnboarding` can lead to un-callable loans [as this allow the borrower to call](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L490) `LendingTerm::_partialRepay` with a debtToRepay of 0.
Allowing him to forever roll his loan without repaying it.

```solidity
File: src\governance\LendingTermOnboarding.sol
138:         require(
139:             params.minPartialRepayPercent < 1e18, // periodic payment sizes [0, 100[%
140:             "LendingTermOnboarding: invalid minPartialRepayPercent"	//@audit-issue Lead B-0 if partialRepay = 0, loan cannot be called. should verify a low value too
141:         );
```


