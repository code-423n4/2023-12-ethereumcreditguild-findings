## Title
Blacklist processing for Credit Token and Collateral token.

## Description
If bidder and borrower is black listed, onBid function will be fail.

```solidity
    function onBid(
        bytes32 loanId,
        address bidder,
        uint256 collateralToBorrower,
        uint256 collateralToBidder,
        uint256 creditFromBidder
    ) external {//@audit there is no check for bidder
        // preliminary checks
        require(msg.sender == refs.auctionHouse, "LendingTerm: invalid caller");
        require(
            loans[loanId].callTime != 0 && loans[loanId].callDebt != 0,
            "LendingTerm: loan not called"
        );
        require(loans[loanId].closeTime == 0, "LendingTerm: loan closed");

        // sanity check on collateral movement
        // these should never fail for a properly implemented AuctionHouse contract
        // collateralOut == 0 if forgive() while in auctionHouse
        uint256 collateralOut = collateralToBorrower + collateralToBidder;
        require(
            collateralOut == loans[loanId].collateralAmount ||
                collateralOut == 0,
            "LendingTerm: invalid collateral movements"
        );

        // compute pnl
        uint256 creditMultiplier = ProfitManager(refs.profitManager)
            .creditMultiplier();
        uint256 borrowAmount = loans[loanId].borrowAmount;
        uint256 principal = (borrowAmount *
            loans[loanId].borrowCreditMultiplier) / creditMultiplier;
        int256 pnl;
        uint256 interest;
        if (creditFromBidder >= principal) {
            interest = creditFromBidder - principal;
            pnl = int256(interest);
        } else {
            pnl = int256(creditFromBidder) - int256(principal);
            principal = creditFromBidder;
            require(
                collateralToBorrower == 0,
                "LendingTerm: invalid collateral movement"
            );
        }

        // save loan state
        loans[loanId].closeTime = block.timestamp;

        // pull credit from bidder
        if (creditFromBidder != 0) {
            CreditToken(refs.creditToken).transferFrom(
                bidder,
                address(this),
                creditFromBidder
            );
        }

        // burn credit principal, replenish buffer
        if (principal != 0) {
            CreditToken(refs.creditToken).burn(principal);
            RateLimitedMinter(refs.creditMinter).replenishBuffer(principal);
        }

        // handle profit & losses
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

        // decrease issuance
        issuance -= borrowAmount;

        // send collateral to borrower
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

        emit LoanClose(
            block.timestamp,
            loanId,
            LoanCloseType.Call,
            creditFromBidder
        );
    }

```

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