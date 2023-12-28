# [01] AuctionHouse#getBidDetails - Comments do not reflect behavior

In `getBidDetails`, the comments suggest that anyone can receive the full collateral and give 0 credit after the second phase has fully elapsed.

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/AuctionHouse.sol#L154-L160

```solidity
        // second phase fully elapsed, anyone can receive the full collateral and give 0 CREDIT
        // in practice, somebody should have taken the arb before we reach this condition.
        else {
            // receive the full collateral
            collateralReceived = auctions[loanId].collateralAmount;
            //creditAsked = 0; // implicit
        }
```

However, `forgive` must be called after the second phase. This function uses `collateralToBidder = 0`, meaning that the caller receives nothing. There is no incentive for anyone to call `forgive` after the second phase.

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/AuctionHouse.sol#L202-L230

```solidity
    function forgive(bytes32 loanId) external {
        // this view function will revert if the auction is not started,
        // or if the auction is already ended.
        (, uint256 creditAsked) = getBidDetail(loanId);
        require(creditAsked == 0, "AuctionHouse: ongoing auction");

        // close the auction in state
        auctions[loanId].endTime = block.timestamp;
        nAuctionsInProgress--;

        // notify LendingTerm of auction result
        address _lendingTerm = auctions[loanId].lendingTerm;
        LendingTerm(_lendingTerm).onBid(
            loanId,
            msg.sender,
            0, // collateralToBorrower
            0, // collateralToBidder
            0 // creditFromBidder
        );

        // emit event
        emit AuctionEnd(
            block.timestamp,
            loanId,
            LendingTerm(_lendingTerm).collateralToken(),
            0, // collateralSold
            0 // debtRecovered
        );
    }
```

# [02] LendingTerm - Repaying/closing a loan does not reset the borrowAmount to 0

When a loan is repaid, the `loan.borrowAmount` is not reset to 0.

Any external contracts that call `LendingTerm.getLoan` to get the `borrowAmount` of a closed loan will be returned the incorrect amount.

## Proof of Concept

Add this test to `test/unit/loan/LendingTerm.t.sol`:

```solidity
    function test_partialPay_loanDebtAccounting() public {

        // prepare & borrow
        uint256 borrowAmount = 100e18;
        uint256 collateralAmount = borrowAmount / 1000;

        collateral.mint(address(this), collateralAmount);
        collateral.approve(address(term), collateralAmount);
        bytes32 loanId = term.borrow(borrowAmount, collateralAmount);

        vm.roll(block.number + 1);
        skip(12);

        // full repay
        uint256 debt = term.getLoanDebt(loanId);
        credit.mint(address(this), debt);
        credit.approve(address(term), debt);
        term.repay(loanId);
        
        // checks
        assertEq(term.issuance(), 0);
        assertEq(term.getLoan(loanId).borrowAmount, 0, "borrow amount should be 0");
        assertEq(term.getLoanDebt(loanId), 0, "loan debt should be 0");
    }
```