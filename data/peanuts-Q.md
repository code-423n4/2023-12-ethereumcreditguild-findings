### [L-01] ProfitSharingConfig is not checked correctly

The comments in ProfitManager mentions that the sum of `surplusBufferSplit`, `guildSplit`, and `otherSplit` should be <= 1e9

```
    /// `surplusBufferSplit`, `guildSplit`, and `otherSplit` are expressed as percentages with 9 decimals,
    /// so a value of 1e9 would direct 100% of profits. The sum should be <= 1e9.
```

but in `setProfitSharingConfig` , it does not check that the three variables adds up to 1e9 maximum, but rather adds up to 1e18 with creditSplit.

```
/// @notice set the profit sharing config.
    function setProfitSharingConfig(
        uint256 surplusBufferSplit,
        uint256 creditSplit,
        uint256 guildSplit,
        uint256 otherSplit,
        address otherRecipient
    ) external onlyCoreRole(CoreRoles.GOVERNOR) {
        if (otherRecipient == address(0)) {
            require(otherSplit == 0, "GuildToken: invalid config");
        } else {
            require(otherSplit != 0, "GuildToken: invalid config");
        }
        require(
            surplusBufferSplit + otherSplit + guildSplit + creditSplit == 1e18,
            "GuildToken: invalid config"
        );
```

This means that the three variables can add up to 1e18, not 1e9, which will break other functions (say if surplusBufferSplit is set at 1e15). Have a require statement that checks that the three variables is <= 1e9 as well.

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/ProfitManager.sol#L195-L210

### [L-02] There is no incentive to call forgive() in AuctionHouse

The `forgive()` function is only called when there is 100% collateral and 0% debt. This case is unlikely, but when it happens, there is no incentive to call `forgive()`. The caller will only waste gas fees, unlike calling `bid()` where the caller may get a profit. 

```
   /// @notice forgive a loan, by marking the debt as a total loss
    /// @dev this is meant to be used when an auction concludes without anyone bidding,
    /// even if 0 CREDIT is asked in return. This situation could arise
    /// if collateral assets are frozen within the lending term contract.
    function forgive(bytes32 loanId) external {
        // this view function will revert if the auction is not started,
        // or if the auction is already ended.
        (, uint256 creditAsked) = getBidDetail(loanId);
        require(creditAsked == 0, "AuctionHouse: ongoing auction");
```

A problem that arises when `forgive()` is not called is that the `creditMultiplier` will not be updated through `ProfitManager(refs.profitManager).notifyPnL(address(this), pnl);`. This means that the `creditMultiplier` may be wrong when new borrowers come and interact with the protocol. 

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/AuctionHouse.sol#L202-L215
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L797

### [L-03] Collateral is stuck when forgive() is called

When `forgive()` is called, that means there is 100% collateral and 0% debt. The function will call `onBid()` with 0 value for `collateralToBorrower`,  `collateralToBidder`, `creditFromBidder`. 

```
LendingTerm(_lendingTerm).onBid(
            loanId,
            msg.sender,
            0, // collateralToBorrower
            0, // collateralToBidder
            0 // creditFromBidder
        );
```

Since there is no collateral being sent to the borrower or bidder, the collateral is effectively stuck in the protocol as the auction is closed for the loanID after `forgive()` is called.

```
  // close the auction in state
        auctions[loanId].endTime = block.timestamp;
        nAuctionsInProgress--;
```

Let's say collateral is 1 SOLO (random established token), and SOLO drops from 2000 to 1 dollar. The borrower borrowed 1000 USDC (200% Collateral Ratio at the time). Because of the flash drop in price, the loanID went to the auction, and since the drop is so drastic, no one wants to pay for the collateral, even if they will get 100% collateral for 1% debt. In this sense, 1 SOLO (worth 1 dollar) is essentially stuck in the contract.

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/AuctionHouse.sol#L202-L230

### [L-04] A blocklisted address can affect the auction procedure

If the collateral token has a blocklist, and the borrower is insider the blocklist, then the auction will not work as intended. Users cannot call `bid()` as the function will revert when the lendingTerm contract tries to send the collateral back to the borrower.

```
  // send collateral to borrower
        if (collateralToBorrower != 0) {
            IERC20(params.collateralToken).safeTransfer(
>               loans[loanId].borrower,
>               collateralToBorrower
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
If the blocklist lasts for a long duration, then the auction will reach the 100% collateral 0% debt stage, at which the function `forgive()` can be called. If the borrower has their address unblocked after, the borrower cannot get back his own collateral anymore and the collateral will be stuck in the contract.

Recommend having a way to extract collateral stuck in lendingterm, although it may introduce centralization issues.

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/AuctionHouse.sol#L211-L221


### [L-05] Low-decimal collateral tokens will affect the calculations of Auction, causing bidders to lose money

There are some tokens that have 2 decimal places, like GUSD. If these tokens are used as collateral, it will affect the Auction process and calculation.

In `getBidDetail()`, during the first phase of the auction, the function will calculate how much collateral the bidder receives for paying 100% debt.

```
        // first phase of the auction, where more and more collateral is offered
        if (block.timestamp < _startTime + midPoint) {
            // ask for the full debt
            creditAsked = auctions[loanId].callDebt;

            // compute amount of collateral received
            uint256 elapsed = block.timestamp - _startTime; // [0, midPoint[
            uint256 _collateralAmount = auctions[loanId].collateralAmount; // SLOAD
>           collateralReceived = (_collateralAmount * elapsed) / midPoint;
```

If the second elapsed is low and the `midPoint` duration is decently high, then the `collateralReceived` value will truncate to zero. For example, if the collateral amount is 10e2, the time elasped is 5 and the midPoint is 86400 (1 day), the `collateralReceived` will be 0. 

Ensure that the collateralReceived is not truncated so that the bidder can get will get the appropriate amount of collateral, otherwise the truncation will just benefit the borrower. 

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/AuctionHouse.sol#L133-L144

### [L-06] POLL_DURATION_BLOCKS assumes that every chain uses a 13s/block timing

Chains like Arbitrum syncs their block.number to Ethereum occasionally and chains like Optimism has a 2 second block timing. POLL_DURATION_BLOCKs in LendingTermOffboarding.sol should not be hardcoded to 46523, which is the ethereum block timing.

```
function proposeOffboard(address term) external whenNotPaused {
        require(
            polls[block.number][term] == 0,
            "LendingTermOffboarding: poll exists"
        );
        require(
            block.number > lastPollBlock[term] + POLL_DURATION_BLOCKS,
            "LendingTermOffboarding: poll active"
        );
```

If deployed on other L2s with different block timing, the time for the proposal term may be more or less than intended, which will affect the governance decision in every chain. Make sure the variable is not hardcoded.

Also, Ethereum might not have a strict 13/s per block time. In fact, the block time is now 12/s after the merge. Best is to have the owner set the POLL_DURATION_BLOCKS than to give it a constant value.

https://docs.arbitrum.io/for-devs/concepts/differences-between-arbitrum-ethereum/block-numbers-and-time

### [N-01] Midpoint and auction duration cannot be changed

Once the midpoint and auctionduration is set in the constructor of Auction House, it cannot be changed. All the loans in a particular lending term will have to follow the duration. This may cause inflexibility and limitations to the governance (governance can vote to change the midpoint and auction duration)

```
   constructor(
        address _core,
        uint256 _midPoint,
        uint256 _auctionDuration
    ) CoreRef(_core) {
        require(_midPoint < _auctionDuration, "AuctionHouse: invalid params");
        midPoint = _midPoint;
        auctionDuration = _auctionDuration;
    }
```

### [N-02] AuctionHouse follows block.timestamp, instead of block.numbers as stated in the comments

The comments state that the first phase will last around 50 blocks in AuctionHouse.sol

```
    /// @notice maximum duration of auctions, in seconds.
    /// with a midpoint of 650 (10m50s) and an auction duration of 30min, and a block every
    /// 13s, first phase will last around 50 blocks and each block will offer an additional
    /// 1/(650/13)=2% of the collateral during the first phase. During the second phase,
    /// every block will ask 1/((1800-650)/13)=1.13% less CREDIT in each block.
    uint256 public immutable auctionDuration;
```

but uses block.timestamp when calculating the time

```
  // first phase of the auction, where more and more collateral is offered
        if (block.timestamp < _startTime + midPoint) {
            // ask for the full debt
            creditAsked = auctions[loanId].callDebt;

            // compute amount of collateral received
            uint256 elapsed = block.timestamp - _startTime; // [0, midPoint[
            uint256 _collateralAmount = auctions[loanId].collateralAmount; // SLOAD
            collateralReceived = (_collateralAmount * elapsed) / midPoint;
        }
```

Some l2s like optimism and even ethereum itself don't really follow the 13s == 1 block timing. Best is to talk about timing in terms of block.timestamp than block.number

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/AuctionHouse.sol#L133-L143

