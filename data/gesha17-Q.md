# [L-1] Some weird tokens transfer less than amount.

Some weird tokens can transfer less than `amount`. For example using a specific integer value to denote transfer of all tokens.
This can be updated over time with upgradeable ERC20s, so it's best to add checks for it.

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L466
```solidity
        loans[loanId].collateralAmount += collateralToAdd;

        // pull the collateral from the borrower
        IERC20(params.collateralToken).safeTransferFrom(
            borrower,
            address(this),
            collateralToAdd
        );
```
Check if `IERC20(params.collateralToken).balanceOf(address(this))` before and after transfer checks out with transferred amount.

Read more here: 
https://github.com/d-xo/weird-erc20#transfer-of-less-than-amount.

# [L-2] User allowed to borrow over debt ceiling if he is taking out the first loan in a market.

If the totalBorrowedCredit for a market is zero, the postBorrowIssuance is not compared against the debt ceiling.
In practice when the first loan is issued in a new market, there should be only 1 gauge term, meaning all available credit is allocated to this specific gauge. But if by chance there is more than one, the user can take out all credit in that market, regardless of gauge allocation.

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L388
```solidity

        if (totalBorrowedCredit == 0) {
            // if the lending term is deprecated, `calculateGaugeAllocation` will return 0, and the borrow
            // should revert because the debt ceiling is reached (no borrows should be allowed anymore).
            // first borrow in the system does not check proportions of issuance, just that the term is not deprecated.
            require(_debtCeiling != 0, "LendingTerm: debt ceiling reached");
        } else {
            require(
                _postBorrowIssuance <= _debtCeiling,
                "LendingTerm: debt ceiling reached"
            );
        }
```

# [L-3] No timelock period in lending term offboarding procedure.

Lending terms can be offboarded without a timelock or the possibility for a veto. This means that a user with enough governance power can start offboarding lending terms and directly opening auctions for loans at will. A malicios user can offboard all LendingTerms in a market before governance has a chance to update the quorum. The only way to bring the offboarded lending terms back is re-onboard them with a new governance proposal.