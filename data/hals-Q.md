# Ethereum Guild QA Report

| ID            | Title                                                                                                                         | Severity |
| ------------- | ----------------------------------------------------------------------------------------------------------------------------- | -------- |
| [L-01](#l-01) | `CoreRef.emergencyAction` function doesn't refund residual `msg.value`                                                        | Low      |
| [L-02](#l-02) | `GuildGovernor` contract allows updating `_quorum` during an ongoing voting                                                   | Low      |
| [L-03](#l-03) | `SimplePSM` contract doesn't support tokens with decimals > 18                                                                | Low      |
| [L-04](#l-04) | `AuctionHouse` contract: bidders will not receive any amount of collateral when bidding at the same block the auction created | Low      |
| [L-05](#l-05) | Hardcoded time durations are not compatible with L2s chains                                                                   | Low      |
| [L-06](#l-06) | `GIP_0`: higher thresholds are set upon protocol deployment                                                                   | Low      |
| [L-07](#l-07) | `SurplusGuildMinter.setMintRatio` function missing lower bound check                                                          | Low      |
| [L-08](#l-08) | USDC token is an upgradeable proxy that might brick the deployed terms if upgraded to charge for transfers                                       | Low      |

## [L-01] `CoreRef.emergencyAction` function doesn't refund residual `msg.value`<a id="l-01" ></a>

## Details

`CoreRef.emergencyAction` function is meant to enable contracts that inherit `CoreRef` from making external calls, and this function is marked as `payable` to enable send target addresses native tokens, but it was noticed that the function doesn't check if there's any residual amount of native tokens after the call which will result in locking these native tokens temporarily.

## Proof of Concept

[CoreRef.emergencyAction function](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/core/CoreRef.sol#L87C1-L107C6)

```javascript
    function emergencyAction(
        Call[] calldata calls
    )
        external
        payable
        onlyCoreRole(CoreRoles.GOVERNOR)
        returns (bytes[] memory returnData)
    {
        returnData = new bytes[](calls.length);
        for (uint256 i = 0; i < calls.length; i++) {
            address payable target = payable(calls[i].target);
            uint256 value = calls[i].value;
            bytes calldata callData = calls[i].callData;


            (bool success, bytes memory returned) = target.call{value: value}(
                callData
            );
            require(success, "CoreRef: underlying call reverted");
            returnData[i] = returned;
        }
    }
```

## Recommendation

Refund residual native tokens to the sender (DAO) after `emergencyAction` call.

## [L-02] `GuildGovernor` contract allows updating `_quorum` during an ongoing voting<a id="l-02" ></a>

## Details

- `GuildGovernor` contract governor (the DAO) can adjust the `_quorum` value (which represents the minimum number of votes needed for a proposal to pass) via `setQuorum` function.

- But setting this value if there are proposals under voting will result in unfair opportunities for proposals to win.

## Proof of Concept

[GuildGovernor.setQuorum function](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/GuildGovernor.sol#L99C1-L103C6)

```javascript
    function setQuorum(
        uint256 newQuorum
    ) public onlyCoreRole(CoreRoles.GOVERNOR) {
        _setQuorum(newQuorum);
    }s
```

## Recommendation

Prevent updating this value during an ongoing voting.

## [L-03] `SimplePSM` contract doesn't support tokens with decimals > 18<a id="l-03" ></a>

## Details

- `SimplePSM` contract is meant to allow mint/redeem of CREDIT token outside of lending terms & guarantee a stable peg of the CREDIT token around the value targeted by the protocol

- Upon contract deployment, the value of `decimalCorrection` that represents multiplier for decimals correction is set as:

  ```javascript
  decimalCorrection = 10 ** (18 - decimals);
  ```

  So as can be noticed; if the `_pegToken` decimals is > 18 , the deployment will revert.

## Proof of Concept

[SimplePSM.constructor/L76 ](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/SimplePSM.sol#L76)

```javascript
decimalCorrection = 10 ** (18 - decimals);
```

## Recommendation

Update `decimalCorrection` calculation to support tokens with decimals > 18.

## [L-04] `AuctionHouse` contract: bidders will not receive any amount of collateral when bidding at the same block the auction created<a id="l-04" ></a>

## Details

If a bidder adds a bid right after the loan auction is created (at the same `block.timestamp`), he will not be receiving any collateral amount while paying the full borrow amount.

## Proof of Concept

### Code Instance:

[AuctionHouse.getBidDetail function](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/AuctionHouse.sol#L118C5-L161C6)

```javascript
    function getBidDetail(
        bytes32 loanId
    ) public view returns (uint256 collateralReceived, uint256 creditAsked) {
        // check the auction for this loan exists
        uint256 _startTime = auctions[loanId].startTime;
        require(_startTime != 0, "AuctionHouse: invalid auction");


        // check the auction for this loan isn't ended
        require(auctions[loanId].endTime == 0, "AuctionHouse: auction ended");


        // assertion should never fail because when an auction is created,
        // block.timestamp is recorded as the auction start time, and we check in previous
        // lines that start time != 0, so the auction has started.
        assert(block.timestamp >= _startTime);


        // first phase of the auction, where more and more collateral is offered
        if (block.timestamp < _startTime + midPoint) {
            // ask for the full debt
            creditAsked = auctions[loanId].callDebt;


            // compute amount of collateral received
            uint256 elapsed = block.timestamp - _startTime; // [0, midPoint[
            uint256 _collateralAmount = auctions[loanId].collateralAmount; // SLOAD
            collateralReceived = (_collateralAmount * elapsed) / midPoint;
        }
        // second phase of the auction, where less and less CREDIT is asked
        else if (block.timestamp < _startTime + auctionDuration) {
            // receive the full collateral
            collateralReceived = auctions[loanId].collateralAmount;


            // compute amount of CREDIT to ask
            uint256 PHASE_2_DURATION = auctionDuration - midPoint;
            uint256 elapsed = block.timestamp - _startTime - midPoint; // [0, PHASE_2_DURATION[
            uint256 _callDebt = auctions[loanId].callDebt; // SLOAD
            creditAsked = _callDebt - (_callDebt * elapsed) / PHASE_2_DURATION;
        }
        // second phase fully elapsed, anyone can receive the full collateral and give 0 CREDIT
        // in practice, somebody should have taken the arb before we reach this condition.
        else {
            // receive the full collateral
            collateralReceived = auctions[loanId].collateralAmount;
            //creditAsked = 0; // implicit
        }
    }
```

### Foundry PoC:

1. Set the following `testZeroCollateralReturned` test in the `AuctionHouse.t.sol` test file:

   ```javascript
   function testZeroCollateralReturned() public {
           //1. setup and call a loan:
           // prepare
           uint256 borrowAmount = 100e18;
           uint256 collateralAmount = 125e18;
           collateral.mint(address(borrower), collateralAmount);
           // borrow
           vm.startPrank(borrower);
           collateral.approve(address(term), collateralAmount);
           bytes32 loanId = term.borrow(borrowAmount, collateralAmount);
           vm.stopPrank();
           // 1 year later, interest accrued
           vm.warp(block.timestamp + term.YEAR());
           vm.roll(block.number + 1);
           // call
           guild.removeGauge(address(term));
           vm.startPrank(caller);
           term.call(loanId);
           vm.stopPrank();

           // 2. bid right after the start of the auction; the bidder will gain zero collateral
           (uint256 collateralReceived, uint256 creditAsked) = auctionHouse
               .getBidDetail(loanId);
           assertEq(
               collateralReceived,
               0,
               "collateral received should never reach zero"
           );
           assertEq(collateralReceived, 0);
           assertEq(creditAsked, 110e18);
       }
   ```

2. Test Result:

   ```bash
   $ forge test --mt testZeroCollateralReturned -vvv
   Running 1 test for test/unit/loan/AuctionHouse.t.sol:AuctionHouseUnitTest
   [PASS] testZeroCollateralReturned() (gas: 645027)
   Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 8.36ms
   Ran 1 test suites: 1 tests passed, 0 failed, 0 skipped (1 total tests)
   ```

## Recommendation

- Consider preventing bids at the same block the auction created.
- Consider adding a minimum amount parameter to the `AuctionHouse.bid` function to be determined by the bidder to prevent receiving lower collateral amount than expected.

## [L-05] Hardcoded time durations are not compatible with L2s chains <a id="l-05" ></a>

## Details

- Some time constants are hardcoded based on the number of blocks minted per second on the Ethereum mainnet, but knowing that the protocol is going to be deployed on L2s as well where it has different block/second values; which will make the hardcoded values incompatible with other chains.

- For example: in `GIP_0.sol` deployment script, the value of `BLOCKS_PER_DAY` is set based on one block minted/13 seconds which is correct only for the Ethereum mainnet, while this value is much lesser on L2s:

  ```javascript
  uint256 public constant BLOCKS_PER_DAY = 7164;
  ```

  which would result in inaccurate values for other constants that rely on `BLOCKS_PER_DAY` for its calculation.

## Proof of Concept

[LendingTermOffboarding.POLL_DURATION_BLOCKS](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/LendingTermOffboarding.sol#L36C1-L36C81)

```javascript
    uint256 public constant POLL_DURATION_BLOCKS = 46523; // ~7 days @ 13s/block
```

[xxx.xx function](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/test/proposals/gips/GIP_0.sol#L83C5-L83C51)

```javascript
uint256 public constant BLOCKS_PER_DAY = 7164;
```

## Recommendation

Ensure that any constant depending on the block/seconds is entered during deployment instead of being hardcoded.

## [L-06] `GIP_0`: higher thresholds are set upon protocol deployment <a id="l-06" ></a>

## Details

- In `GIP_0` deployment script; the value of `DAO_GOVERNOR_GUILD_QUORUM` is set to 25million token (`25_000_000e18`), where this variable represents the minimum number of votes needed for a vote to pass (CREDIT token votes), but this value would be considered high for the protocol in its early stages.

- Also knowing that a market (onboarding a `LendingTerm`) can be added via proposing and voting; the `ONBOARD_GOVERNOR_GUILD_PROPOSAL_THRESHOLD` set to ` 1_000_000e18` would result in delaying the protocol operations (adding markets and onboarding users).

- Also the minimum value set for a user to add a proposal (such as onboarding a `LendingTerm`) is considered high for the protocol in its early stages:

  ```javascript
      uint256 public constant ONBOARD_GOVERNOR_GUILD_PROPOSAL_THRESHOLD =
          1_000_000e18;
  ```

## Proof of Concept

[GIP_0](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/test/proposals/gips/GIP_0.sol#L92C5-L107C1)

```javascript
uint256 public constant DAO_GOVERNOR_GUILD_PROPOSAL_THRESHOLD =
        2_500_000e18;
    uint256 public constant DAO_GOVERNOR_GUILD_QUORUM = 25_000_000e18;
    uint256 public constant DAO_VETO_CREDIT_QUORUM = 5_000_000e18;
    uint256 public constant DAO_VETO_GUILD_QUORUM = 15_000_000e18;
    uint256 public constant ONBOARD_GOVERNOR_GUILD_VOTING_DELAY =
        0 * BLOCKS_PER_DAY;
    uint256 public constant ONBOARD_GOVERNOR_GUILD_VOTING_PERIOD =
        2 * BLOCKS_PER_DAY;
    uint256 public constant ONBOARD_GOVERNOR_GUILD_PROPOSAL_THRESHOLD =
        1_000_000e18;
    uint256 public constant ONBOARD_GOVERNOR_GUILD_QUORUM = 10_000_000e18;
    uint256 public constant ONBOARD_VETO_CREDIT_QUORUM = 5_000_000e18;
    uint256 public constant ONBOARD_VETO_GUILD_QUORUM = 10_000_000e18;
    uint256 public constant OFFBOARD_QUORUM = 10_000_000e18;
```

## Recommendation

Consider adding lower/reasonable values for threshold and quorum limits upon deployment.

## [L-07] `SurplusGuildMinter.setMintRatio` function missing lower bound check<a id="l-07" ></a>

## Details

`SurplusGuildMinter.setMintRatio` function is meant to be accessible by the DAO to set the ratio of GUILD tokens minted per CREDIT tokens contributed to the surplus buffer; but it was noted that there's no check if this value is zero before assigning it, neither does it have an upper bound check.

## Proof of Concept

[SurplusGuildMinter.setMintRatio function](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/SurplusGuildMinter.sol#L319C1-L324C6)

```javascript
    function setMintRatio(
        uint256 _mintRatio
    ) external onlyCoreRole(CoreRoles.GOVERNOR) {
        mintRatio = _mintRatio;
        emit MintRatioUpdate(block.timestamp, _mintRatio);
    }
```

## Recommendation

Consider checking against upper and lower bounds before assigning a new value to the `mintRatio` variable.
## [L-08] USDC token is an upgradeable proxy that might brick the deployed terms if upgraded to charge for transfers<a id="l-08" ></a>

## Details

- USDC token is going to be used as an underlying token for deployed terms (markets), while this token currently doesn't charge fee for each trasnfer transaction (as fee on transfer tokens); but it could be upgraded to charge fees and have the same behavior as fee on transfer tokens where the receiver will receive less than the transferred amount by fee amount.

- This would brick the accounting logic in the `LendingTerm` contract where the **recorded** balance of the CREDIT token (minted for the borrower when he deposits collateral to borrow,`issuance`) will be greater than the contract collateral token balance, which will lead to last users unable to repay their debts as the balance of the contract would be insufficient (`issuance` that represents borrows is > term collateral balance).

## Proof of Concept

[USDC token contract on Ethereum](https://etherscan.io/token/0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48#code#L16)

## Recommendation

Consider updating `LendingTerm._borrow` function to account for this possible future behavior (charging fees for transferring tokens) of USDC token.
