# Table of contents

| Issue No                                                                          | Title                                                                      |
| --------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| [L01](#1-lendingtermsol-doesnt-allow-multiple-loans-in-same-block-from-same-term) | `Lendingterm.sol` Doesnt allow multiple loans in same block from same term |
| [L02](#2-_partialrepay-in-lendingtermsol-does-not-support-0-interest-loans)       | `_partialRepay` in `LendingTerm.sol` does not support 0 interest loans     |
| [L03](#3-nauctionsinprogress-can-be-incorrect-in-auctionhousesol-contract)        | `nAuctionsInProgress` can be incorrect in `AuctionHouse.sol` contract      |

# 1. `Lendingterm.sol` Doesnt allow multiple loans in same block from same term

## Description

The `loanId` for borrows in `LendingTerm.sol` is calculated using the following line.

```solidity
loanId = keccak256(
    abi.encode(borrower, address(this), block.timestamp)
);
```

This means that if the same borrower takes out two loans in the same block, all parameters in the hash generation will match and the contract will essentially generate the same `loanId` for both loans. This will cause the second loan to fail as the `loanId` will already be in use, due to this require statement.

```solidity
// check that the loan doesn't already exist
require(loans[loanId].borrowTime == 0, "LendingTerm: loan exists");
```

Thus users and contracts are restricted to only taking out one loan per block. Consider generating loanId with an incrementing number in the hash input to avoid this issue.

# 2. `_partialRepay` in `LendingTerm.sol` does not support 0 interest loans

The function `_partialRepay` is used to make partial repayments of a loan. The issue is that during repayment, the `interestRepaid` is always required to be above 0.

```solidity
uint256 interestRepaid = debtToRepay - principalRepaid;
require(
    principalRepaid != 0 && interestRepaid != 0,
    "LendingTerm: repay too small"
);
```

Thus 0 interest loans are not supported by this function. Consider waiving that statement, since dust repayments are already stopped by the minimum repayment percentage.

# 3. `nAuctionsInProgress` can be incorrect in `AuctionHouse.sol` contract

`nAuctionsInProgress` is meant to track the number of active auctions in the AuctionHouse. It is incremented at the start of a new auction and decremented at the end. However, if a loan is in the AuctionHouse, and the loan is forgiven via the LendingTerm contract itself, then the `nAuctionsInProgress` will not be decremented.

This happens because there are two ways to forgive a loan. Via the `forgive` function in LendingTerm.sol, and via the `forgive` function in AuctionHouse.sol. The `forgive` function in LendingTerm.sol does not decrement `nAuctionsInProgress`, while the `forgive` function in AuctionHouse.sol does. Thus loans forgiven by the `LendingTerm` contract will give wrong values of active auctions.
