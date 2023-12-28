# There is no check whether a `LendingTerm` has received enough weight

## Description

`supportOffboard()` function in the `LendingTermOffboarding.sol` allow users to support the offboarding of a specific Lending Term:
```solidity
    function supportOffboard(
        uint256 snapshotBlock,
        address term
    ) external whenNotPaused {
        require(
            block.number <= snapshotBlock + POLL_DURATION_BLOCKS,
            "LendingTermOffboarding: poll expired"
        );
        uint256 _weight = polls[snapshotBlock][term];
        require(_weight != 0, "LendingTermOffboarding: poll not found");
        uint256 userWeight = GuildToken(guildToken).getPastVotes(
            msg.sender,
            snapshotBlock
        );
        require(userWeight != 0, "LendingTermOffboarding: zero weight");
        require(
            userPollVotes[msg.sender][snapshotBlock][term] == 0,
            "LendingTermOffboarding: already voted"
        );

        userPollVotes[msg.sender][snapshotBlock][term] = userWeight;
        polls[snapshotBlock][term] = _weight + userWeight;
        if (_weight + userWeight >= quorum) {
            canOffboard[term] = true;
        }
        emit OffboardSupport(
            block.timestamp,
            term,
            snapshotBlock,
            msg.sender,
            userWeight
        );

    }
```

If quorum is reached then `canOffboard[term]` settles to true.:
```solidity
        if (_weight + userWeight >= quorum) {
            canOffboard[term] = true;
        }
```
However, users can still continue to vote because there is no check at the beginning of the function whether the quorum has been reached.
## References

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/LendingTermOffboarding.sol#L115-L148
## Recommendation

Add a check at the beginning of the function if the quorum is reached.
***

# No one will have the initiative to call `forgive()`  in `AuctionHouse.sol`

## Description

When the auction is over and no one has placed a bid then the protocol has a bad debt and `forgive()` should be called:
```soldiity
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

The protocol has written that someone can get the collateral for 0 tokens but this is not true:
```solidity
        // second phase fully elapsed, anyone can receive the full collateral and give 0 CREDIT
        // in practice, somebody should have taken the arb before we reach this condition.
        else {
            // receive the full collateral
            collateralReceived = auctions[loanId].collateralAmount;
            //creditAsked = 0; // implicit
        }
```

Since no one gets anything in return when calling the `forgive` function then no one will call it.
## References

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/AuctionHouse.sol#L154-L160
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/AuctionHouse.sol#L198-L230
## Recommendation

Add some sort of reward in the `forgive()` function to incentivize someone to call it.

***
# The second phase of the auction does not include the last block

## Description

`getBidDetail()` function is used in `bid()` and `forgive()` function to get the bid details for an active auction.
We are interested in the second phase of the auction:
```solidity
        else if (block.timestamp < _startTime + auctionDuration) { //@audit not include last block bid
            // receive the full collateral
            collateralReceived = auctions[loanId].collateralAmount;

            // compute amount of CREDIT to ask
            uint256 PHASE_2_DURATION = auctionDuration - midPoint;
            uint256 elapsed = block.timestamp - _startTime - midPoint; // [0, PHASE_2_DURATION[
            uint256 _callDebt = auctions[loanId].callDebt; // SLOAD
            creditAsked = _callDebt - (_callDebt * elapsed) / PHASE_2_DURATION;
        }
```

We can see from the `else if` this phase does not include the last block bid. 
If someone decides to place a bid right in the last 12 seconds (Ethereum) of the auction, they will get nothing.
## References

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/AuctionHouse.sol#L144
## Recommendation

Change conditioning from:
```solidity
else if (block.timestamp < _startTime + auctionDuration) {
```
To:
```solidity
else if (block.timestamp <= _startTime + auctionDuration) {
```

