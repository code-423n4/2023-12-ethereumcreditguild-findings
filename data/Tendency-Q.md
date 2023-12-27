
## [L-01] Repayment Could Be Bricked
If a lending term is onboarded that uses a collateral token that has a blacklist or freeze functionality like `USDC`, in the event where a borrower is blacklisted the repayment` of borrowed `CREDIT` via `LendingTerm::repay` will become impossible, since all collateral transfer attempts to the borrower will result in a revert.
Here are aspects of the system that will be bricked:

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L805

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L618


Note that, auctioning will be bricked as well

**RECOMMENDATION**

A possible fix on repayment will be to allow the borrower to input a receiver address, such that if `caller == borrower`, check if the receiver address is not the zero address, if true then send the collateral to the receiver address, else send to the stored loan state borrower.

## [L-02] AuctionHouse Should be Updateable in LendingTermOnboarding

from [lendingTerm::setAuctionHouse](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L829) functionality shows that there is a possibility that the auction house for terms might be updated, but this address is currently immutable in the [onboarder](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/LendingTermOnboarding.sol#L43) contract, so all new terms will still be using the old auction house. so the process will always have to deploy the lending term, and then the governor updates the auction house.

Consider allowing the auction house to be updateable in the onboarder contract via governance since there will only be one onboarder per market.

## [L-03] Still Reverts When Remaining Borrow Amount equals `minBorrow`

During partial repayment in `LendingTerm`, the function still reverts when the amount left after partial repayment equals the `minBorrow()`:
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L529

**RECOMMENDATION**

Change the requirement check to:
```solidity
        require(
            borrowAmount - issuanceDecrease >=
                ProfitManager(refs.profitManager).minBorrow(),
            "LendingTerm: below min borrow"
        );


```

## [L-04] Deploying a SimplePSM with a PegToken With >18 dp Is Currently Bricked
deploying SimplePSM with a peg token with more than 18 decimal precision will result in a revert here:
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/SimplePSM.sol#L75-L76

Consider handling cases where the token has a precision with more than 18 decimal precision

## [L-05] In SurplusGuildMinter, Staking and Unstaking Might be Bricked at a Point

Currently, [SurplusGuildMinter::stake](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/SurplusGuildMinter.sol#L114-L156) function calls [getRewards()](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/SurplusGuildMinter.sol#L216) function, this function does some computation then calls [ProfitManager::claimRewards](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/ProfitManager.sol#L439-L449) on the `SurplusGuildMinter`, the `claimRewards` function then loops through all the available terms users have staked on in the `SurplusGuildMinter`, since these terms aren't whitelisted and the number of terms a user can stake on isn't limited this number will only grow exponentially, at a point both staking and unstaking will revert due to long computation which eventually results in an OOG revert.

**RECOMMENDATION**

Consider whitelisting some selected terms, to avoid this number growing too large

## [N-01] Remove Unnecessary Casting
there is no need wrapping this in `uint256()`, the weight is returned in `uint256`:
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/ProfitManager.sol#L388-L390
```solidity
                uint256 _gaugeWeight = uint256(
                    GuildToken(guild).getGaugeWeight(gauge)
                );
```
`GuildToken::getGaugeWeight` is already returned in `uint256`
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/ERC20Gauges.sol#L74

## [N-02] Remove Misleading Comment
```
        // in practice, somebody should have taken the arb before we reach this condition.
        else {
            // receive the full collateral
            collateralReceived = auctions[loanId].collateralAmount;
            //creditAsked = 0; // implicit
```
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/AuctionHouse.sol#L154-L156
According to a sponsor at that stage, the loan can only be forgiven:
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/AuctionHouse.sol#L198-L202