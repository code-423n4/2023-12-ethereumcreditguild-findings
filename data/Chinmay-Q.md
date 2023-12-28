## [L-1] Collateral transfer to the borrower may fail leading to less recovered collateral for them

The Bidding mechanism in an auction involves sending some collateral to the borrower of the loan if the bid has happened before midPoint( of auction). AuctionHouse.bid function calls LendingTerm.onBid which transfers collateralToken to the borrower if it is !=0. The problem is that if the collateralToken chosen has a blacklisting functionality which is present in some tokens (such as USDC/USDT etc.),  then this transfer can fail if the borrower gets blacklisted for some reason. This will lead to bid failure and brick the auction mechanism till the time of midPoint after which collateralToBorrower will be zero by default. 

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L805

This will lead to the borrower losing on some collateral he could get back if a fair bid was there, or if he himself is trying to repay via bids. 

Wrap this collateral transfer in a try-catch block and go on with fulfilling the auction even if the transfer isn't successful, the collateral may be claimed later by other means. 

## [NC-1] Unused import in SimplePSM.sol

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/SimplePSM.sol#L12

The RateLimitedMinter.sol has ben imported into the SimplePSM.sol contract but not used anywhere in it. The minting is simply done via the CreditToken.mint function so this import is not needed

## [NC-2] USDC has different deployment addresses on other chains

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/test/proposals/gips/GIP_0.sol#L187C45-L187C45

In the mentioned line of code, the deployment script uses a pegToken address to intialize the psm contract. Since USDC is the first market and the first pegToken, so make sure that before deploying it on chains other than Mainnet, the correct address of USDC on that particular chain is used because the address is different on all chains. 