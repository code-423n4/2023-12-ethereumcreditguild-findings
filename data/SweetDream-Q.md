## Title
Blacklist processing for Credit Token and Collateral token.

## Description
If bidder and borrower is black listed, onBid function will be fail.

```solidity
    if (creditFromBidder != 0) {
        CreditToken(refs.creditToken).transferFrom(
            bidder,
            address(this),
            creditFromBidder
        );
    }
```

```solidity
    if (pnl != 0) {
        // forward profit, if any
        if (interest != 0) {
            CreditToken(refs.creditToken).transfer(
                refs.profitManager,
                interest
            );
        }
        ProfitManager(refs.profitManager).notifyPnL(address(this), pnl);
    }
```

```solidity
    if (collateralToBorrower != 0) {
        IERC20(params.collateralToken).safeTransfer(
            loans[loanId].borrower,
            collateralToBorrower
        );
    }

    // send collateral to bidder
    if (collateralToBidder != 0) {
        IERC20(params.collateralToken).safeTransfer(
            bidder,
            collateralToBidder
        );
    }
```

In this function, ```saferTransfer``` and ```transferFrom``` fucntion will be failed if the borrower or bidder is blacklisted.

Imagine that if the borrower cannot repay or partialRepay after borrowing some credit tokens from lendingTerm.

The borrower or system manager will call that loan for liquidate.

And then the borrower is blacklisted in collateral token.

In that case, the borrower can get any collateral token from auction.



## POC

```solidity
    function testBid() public {
        bytes32 loanId = _setupAndCallLoan();
        uint256 PHASE_1_DURATION = auctionHouse.midPoint();
        uint256 PHASE_2_DURATION = auctionHouse.auctionDuration() - auctionHouse.midPoint();
        vm.roll(block.number + 1);
        vm.warp(block.timestamp + PHASE_1_DURATION + PHASE_2_DURATION / 2);
        vm.warp(block.timestamp + PHASE_1_DURATION / 10);

        // At this time, get full collateral, repay half debt
        (uint256 collateralReceived, uint256 creditAsked) = auctionHouse.getBidDetail(loanId);
        console2.log(collateralReceived, creditAsked);
        assertEq(collateralReceived, 15e18);
        assertEq(creditAsked, 11_000e18);

        vm.expectRevert("black listed");
        // bid
        credit.mint(bidder, 11_000e18);
        vm.startPrank(bidder);
        credit.approve(address(term), 11_000e18);
        collateral.setBlacklist(bidder, true);
        auctionHouse.bid(loanId);
        vm.stopPrank();
    }
```