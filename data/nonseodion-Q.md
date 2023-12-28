## L-01: Emit other events for variables set when ProfitManager is deployed

The variables gaugeWeightToleranceUpdate & creditMultiplier are also set during deployment and their events should be emitted like the [minBorrow event](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/ProfitManager.sol).


```
    constructor(address _core) CoreRef(_core) {
        emit MinBorrowUpdate(block.timestamp, 100e18);
    }
```

## L-02: Missing event parameters

The events [Stake and Unstake](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/SurplusGuildMinter.sol#L32-L42) in SurplusGuildMinter should have a user parameter to know the user unstaking. 

```
    event Stake(
        uint256 indexed timestamp,
        address indexed term,
        uint256 amount
    );
    /// @notice emitted when a user unstakes CREDIT on a target lending term
    event Unstake(
        uint256 indexed timestamp,
        address indexed term,
        uint256 amount
    );
```

## N-01: Invalid Comments

The [forgive function](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/AuctionHouse.sol#L203) in AuctionHouse contract is not a view function. The comment should be updated to reflect this

```
    function forgive(bytes32 loanId) external {
        // this view function will revert if the auction is not started,
        // or if the auction is already ended.
        (, uint256 creditAsked) = getBidDetail(loanId);
        require(creditAsked == 0, "AuctionHouse: ongoing auction");

```


## N-02: Ambigous Comments

The [comment](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L742-L743) below from the onBid function in LendingTerm contract is ambiguous because it does not specify if the forgive() function mentioned refers to the forgive function in LendingTerm or AuctionHouse.

```
// collateralOut == 0 if forgive() while in auctionHouse
uint256 collateralOut = collateralToBorrower + collateralToBidder;
        require(
            collateralOut == loans[loanId].collateralAmount ||
                collateralOut == 0,
            "LendingTerm: invalid collateral movements"
        );
```
