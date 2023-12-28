# Low and Non-Critical

### [Low Risk](#low-risk-1)

| Total Low Risk Issues | 3 |
|:--:|:--:|

| Count | Title | Instances |
|:--:|:-------| :--: |
| [L-01](#l-01-lendingtermpartialrepaydelaypassed-will-return-false-when-maxdelaybetweenpartialrepay-has-passed) | `LendingTerm::partialRepayDelayPassed()` will return false when `maxDelayBetweenPartialRepay` has passed | 1 |
| [L-02](#l-02-governor-role-can-forgive-a-called-loan-loan-in-active-auction) | Governor role can forgive a called loan (loan in active auction) | 1 |
| [L-03](#l-03-simplepsmdecimalcorrection-does-not-work-with-high-decimal-tokens-decimals--18) | `SimplePSM::decimalCorrection` does not work with high decimal tokens (decimals > 18) | 1 |

### [Non-Critical](#non-critical-1)

| Total Non-Critical Issues | 3 |
|:--:|:--:|

| Count | Title | Instances |
|:--:|:-------| :--: |
| [NC-01](#nc-01-lastpartialrepay-should-be-in-struct-loan) | `lastPartialRepay` should be in struct Loan | 1 |
| [NC-02](#nc-02-misleading-comment-initial-credit-debt-of-a-loan) | Misleading comment "initial CREDIT debt of a loan" for `borrowAmount` | 1 |
| [NC-03](#nc-03-typo) | Typo | 1 |

## Low Risk

### [L-01] `LendingTerm::partialRepayDelayPassed()` will return false when `maxDelayBetweenPartialRepay` has passed

If for example `lastPartialRepay[loanId] = 1000`, `block.timestamp = 1500`, `maxDelayBetweenPartialRepay = 500` it will return `false`

Change `<` to `<=`.

[LendingTerm#L250-L252](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L250-L252)

```solidity
function partialRepayDelayPassed(
    bytes32 loanId
) public view returns (bool) {
    ...
    // return true if delay is passed
    return
        lastPartialRepay[loanId] <
        block.timestamp - params.maxDelayBetweenPartialRepay;
    }
```

### [L-02] Governor role can forgive a called loan (loan in active auction)

When a loan is called / in an active auction it should not be allowed to be forgiven. Only after the auction concluded with no bids should the loan be forgiven through the `AuctionHouse` contract. However, when a loan is in an active auction it is able to be forgiven by the Governor role through the `LendingTerm` contract. This should be considered as QA since the Governor role is trusted, however the action should not be allowed.

**Recommendation:** add check `require(loan.callTime == 0, "LendingTerm: loan called");`

[LendingTerm.sol#L695-L722](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L695-L722)

Add the following test in `test/unit/loan/LendingTerm.t.sol` to test it out.

```solidity
function testForgiveAfterCallSuccess() public {
    // prepare & borrow
    uint256 borrowAmount = 20_000e18;
    uint256 collateralAmount = 15e18;
    collateral.mint(address(this), collateralAmount);
    collateral.approve(address(term), collateralAmount);
    bytes32 loanId = term.borrow(borrowAmount, collateralAmount);

    // offboard term
    vm.warp(block.timestamp + 13);
    vm.roll(block.number + 1);
    guild.removeGauge(address(term));

    // call the loan to auction
    term.call(loanId);

    // successful call check - loan called into auction
    assertEq(term.getLoan(loanId).caller, address(this));
    assertEq(term.getLoan(loanId).callTime, block.timestamp);

    // governor can forgive the auctioned loan from LendingTerm contract
    vm.prank(governor);
    term.forgive(loanId);

    // successful forgive check - loan closed
    assertEq(term.getLoan(loanId).closeTime, block.timestamp);
    }
```

### [L-03] `SimplePSM::decimalCorrection` does not work with high decimal tokens (decimals > 18)

If a token with more than 18 decimals is used the calculation shown below will overflow. One example of such token is a token with 24 decimals `YAM-V2`

[SimplePSM.sol#L76](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/SimplePSM.sol#L76)

```solidity
decimalCorrection = 10 ** (18 - decimals);
```

## Non-critical

### [NC-01] `lastPartialRepay` should be in struct Loan

`lastPartialRepay` is accessed via `bytes32 loanId` just like state variable `loans` so the variable should just be included in `struct Loan`

[LendingTerm.sol#L73](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L73)

### [NC-02] Misleading comment "initial CREDIT debt of a loan"

`borrowAmount` is not "initial CREDIT debt of a loan" but the "**current** CREDIT debt of a loan" because it decreases in `_partialRepay()`

[LendingTerm.sol#L78](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L78)

### [NC-03] Typo

Typo in [LendingTerm.sol#L724](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L724) "...when **au** auction concludes" should be "...when **an** auction concludes".
