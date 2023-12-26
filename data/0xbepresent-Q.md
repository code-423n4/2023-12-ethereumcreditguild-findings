 # Summary

 | |Issue
 |-:|:-|
| [L-01] | In `LendingTerm::_partialRepay()` it uses the `>` sign to validate the `minBorrow` amount, however in `LendingTerm::_borrow()` it uses `>=`
| [L-02] | The `LendingTerm::forgive()` function does not validate that the loan is on call
| [N-01] | Unused `RateLimitedMinter` contract import
| [N-02] | Return custom error when user does not have gauge weight in the `applyGaugeLoss()` function


## L-01 -  In `LendingTerm::_partialRepay()` it uses the `>` sign to validate the `minBorrow` amount, however in `LendingTerm::_borrow()` it uses `>=`

In the [LendingTerm::_borrow()](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L339) function the following [validation](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L366) is used to check that the borrower is getting a minimum loan amount, this validation uses the `>=` sign:

```solidity
File: LendingTerm.sol
365:         require(
366:             borrowAmount >= ProfitManager(refs.profitManager).minBorrow(),
367:             "LendingTerm: borrow amount too low"
368:         );
```

On the other hand, the [LendingTerm::_partialRepay()](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L490) function uses the same [validation](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L528) but uses the `>` sign:

```solidity
File: LendingTerm.sol
527:         require(
528:             borrowAmount - issuanceDecrease >
529:                 ProfitManager(refs.profitManager).minBorrow(),
530:             "LendingTerm: below min borrow"
531:         );
```

This means that in `LendingTerm::_partialRepay()` the borrower needs to maintain more debt than specified when requesting the loan in `LendingTerm::_borrow()`.

**Recommended Mitigation Steps**

In `LendingTerm::_partialRepay()` maintain the same logic as in `LendingTerm::_borrow()`:

```diff
File: LendingTerm.sol
    function _partialRepay(
        address repayer,
        bytes32 loanId,
        uint256 debtToRepay
    ) internal {
...
...
        require(
--          borrowAmount - issuanceDecrease >
++          borrowAmount - issuanceDecrease >=
                ProfitManager(refs.profitManager).minBorrow(),
            "LendingTerm: below min borrow"
        );
```

## L-02 - The `LendingTerm::forgive()` function does not validate that the loan is on call

The [LendingTerm::forgive()](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L695) function helps the protocol accept loan losses and update the system accordingly. On the other hand, the [LendingTerm::_call()](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L634) function helps to put the loan up for auction.

The problem is that the `LendingTerm::forgive()` function does not validate that the loan is on call or in auction, causing the system to update incorrectly.

Consider the following scenario:

1. For some reason, there is a proposal to call `LendingTerm::forgive()` on `loanId 10`.
2. The `loanId 10` enters the auction and the variable [`loans[loanId].callTime` is set to `block.timestamp`](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L666).
3. The proposal from `step 1` is executed before the auction ends, setting [`loans[loanId].closeTime` to the value of `block.timestamp`](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L705).
4. The auction continues and is reversed by the [following validation](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L738C9-L738C75), making the auction unable to be completed since the code will be reversed:
    ```solidity
    File: LendingTerm.sol
    738: require(loans[loanId].closeTime == 0, "LendingTerm: loan closed");
    ```
5. Since the auction will always be reversed in `LendingTerm::onBid()` the [nAuctionsInProgress variable will not be updated](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/AuctionHouse.sol#L176), not allowing the variable to be subtracted and an auction house [cannot be updated](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L837) using the `LendingTerm::setAuctionHouse()` function.

**Recommended Mitigation Steps**

The `LendingTerm::forgive()` function must validate that the loan is not up for auction:

```diff
    function forgive(bytes32 loanId) external onlyCoreRole(CoreRoles.GOVERNOR) {
        Loan storage loan = loans[loanId];

        // check that the loan exists
        require(loan.borrowTime != 0, "LendingTerm: loan not found");

        // check that the loan is not already closed
        require(loan.closeTime == 0, "LendingTerm: loan closed");

++      // Check the loan is not in auction
++      require(loan.callTime == 0, "LendingTerm: loan on cal");

        // close the loan
        loans[loanId].closeTime = block.timestamp;
        issuance -= loan.borrowAmount;

        // mark loan as a total loss
        uint256 creditMultiplier = ProfitManager(refs.profitManager)
            .creditMultiplier();
        uint256 borrowAmount = loans[loanId].borrowAmount;
        uint256 principal = (borrowAmount *
            loans[loanId].borrowCreditMultiplier) / creditMultiplier;
        int256 pnl = -int256(principal);
        ProfitManager(refs.profitManager).notifyPnL(address(this), pnl);

        // set hardcap to 0 to prevent new borrows
        params.hardCap = 0;

        // emit event
        emit LoanClose(block.timestamp, loanId, LoanCloseType.Forgive, 0);
    }
```

## N-01 - Unused `RateLimitedMinter` contract import

The [RateLimitedMinter contract](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/SimplePSM.sol#L12C9-L12C26) import is not used:

```solidity
File: SimplePSM.sol
12: import {RateLimitedMinter} from "@src/rate-limits/RateLimitedMinter.sol";
```

## N-02 - Return custom error when user does not have gauge weight in the `applyGaugeLoss()` function

The [applyGaugeLoss()](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/GuildToken.sol#L133C14-L133C28) function can be called in order to apply loss to an specific `term` and `user`. The problem is that the funcion will revert without any custom error when `getUserGaugeWeight` returns zero. The next shows how the function is reverted without any custom error:

```solidity
// File: test/unit/loan/SurplusGuildMinter.t.sol:SurplusGuildMinterUnitTest
// $ forge test --match-test "testUnstakeWithLossIsFrontRan" -vvv
//
    function testCustomErrorWhenThereIsNotUserWeight() public {
        vm.prank(governor);
        profitManager.setProfitSharingConfig(
            0.5e18, // surplusBufferSplit
            0, // creditSplit
            0.5e18, // guildSplit
            0, // otherSplit
            address(0) // otherRecipient
        );
        credit.mint(address(profitManager), 35e18);
        profitManager.notifyPnL(term, -27.5e18);
        guild.applyGaugeLoss(term, address(sgm));
    }
```