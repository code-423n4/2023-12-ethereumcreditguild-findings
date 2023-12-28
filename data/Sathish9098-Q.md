## QA Report

##

## [L-1] ``quorum``  variable not capped with reasonable values 

The quorum is not capped, which could lead to governance issues. Extremely high or low values can affect the offboarding process's integrity

```solidity
FILE : 2023-12-ethereumcreditguild/src/governance/LendingTermOffboarding.sol

/// @notice set the quorum for offboard votes
    function setQuorum(
        uint256 _quorum
    ) external onlyCoreRole(CoreRoles.GOVERNOR) {
        emit QuorumUpdated(quorum, _quorum);
        quorum = _quorum;
    }

```
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/LendingTermOffboarding.sol#L77-L83

### Recommended Mitigation
Implement ``min`` and ``max`` value checks 

##

## [L-2]  Hardcoded ``POLL_DURATION_BLOCKS`` cause problems in other chains 

As per protocol docs its also possible to deploy in Arbitrum

```
Deployment: we anticipate to launch on Ethereum mainnet & L2s like Arbitrum.

```
Ethereum Mainnet: The average block time on Ethereum mainnet has historically been around 13-15 seconds. Using 13 seconds as an average is a common approximation for Ethereum mainnet.

Arbitrum (Layer 2): Layer 2 solutions, like Arbitrum, often have different block times compared to Ethereum mainnet. Arbitrum, for instance, doesn't have a fixed block time and can be significantly faster due to its off-chain processing before batches are submitted to Ethereum.

Given these differences, the hardcoded value of ``POLL_DURATION_BLOCKS`` based on a ``13-second `` block time may not be appropriate for all networks where the contract will be deployed:

On Ethereum mainnet, the current implementation is accurate.
On Arbitrum or similar Layer 2 networks, the block time can vary and may not align with the ``13-second ``average. Therefore, the duration of polls in terms of real-time could be significantly different from the intended 7 days.

```solidity
FILE: 2023-12-ethereumcreditguild/src/governance/LendingTermOffboarding.sol

36: uint256 public constant POLL_DURATION_BLOCKS = 46523; // ~7 days @ 13s/block

```
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/LendingTermOffboarding.sol#L36

### Recommended Mitigation
Consider implementing a mechanism to dynamically adjust POLL_DURATION_BLOCKS based on the actual block time of the network where the contract is deployed. This could be done via an oracle or other means to fetch the average block time for the network.

##

## [L-3] No same value input control 

setMinBorrow() and setGaugeWeightTolerance() does not check if input parameter is the same as the current state of the variable which is being updated. If function is being called with the same value as the current state - even though there's no change - function will success and emit an event that the change has been made.

```solidity
FILE: 2023-12-ethereumcreditguild/src/governance/ProfitManager.sol

 /// @notice set the minimum borrow amount
    function setMinBorrow(
        uint256 newValue
    ) external onlyCoreRole(CoreRoles.GOVERNOR) {
        _minBorrow = newValue;
        emit MinBorrowUpdate(block.timestamp, newValue);
    }

/// @notice set the gauge weight tolerance
    function setGaugeWeightTolerance(
        uint256 newValue
    ) external onlyCoreRole(CoreRoles.GOVERNOR) {
        gaugeWeightTolerance = newValue;
        emit GaugeWeightToleranceUpdate(block.timestamp, newValue);
    }



```
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/ProfitManager.sol#L178-L192

### Recommended Mitigation
Add same value checks

```solidity

require(newValue!=_minBorrow );
require(newValue!=gaugeWeightTolerance );

```
##

## [L-4] Evaluating the restrictiveness of same-block loan call and auction start checks

The check loan.callTime == block.timestamp assumes that the loan call and the auction start transaction occur in the same block. In reality, due to network congestion or transaction ordering, these two events could be processed in different blocks, even if the transactions are broadcast back-to-back by the user.

 If the auction start transaction does not get included in the same block as the loan call (for reasons like gas price competition, block fullness, etc.), it will fail. This could lead to increased transaction costs and operational delays.

````solidity
FILE: 2023-12-ethereumcreditguild/src/loan/AuctionHouse.sol

require(
            loan.callTime == block.timestamp,
            "AuctionHouse: loan previously called"
        );


````
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/AuctionHouse.sol#L84-L87

### Recommended Mitigation
 Instead of requiring the auction to start in the same block as the loan call, allow a reasonable time buffer (e.g., a few blocks or a fixed time period like 15 minutes). 

Where BUFFER_PERIOD is the permissible time delay.

```diff

+ require(
+            loan.callTime + BUFFER_PERIOD >= block.timestamp,
+            "AuctionHouse: loan previously called"
 +       ); 

````
##

## [L-5] CoreRoles.GAUGE_PNL_NOTIFIER can initiate the auction even particular loanId does not go underwater

The primary risk here is that any entity with the GAUGE_PNL_NOTIFIER role can initiate an auction for any loan, regardless of its association with that specific loan.

If the GAUGE_PNL_NOTIFIER role is not tightly controlled or if it's assigned to entities that manage multiple loans or different aspects of the system, there's a risk of auctions being initiated inappropriately, either accidentally or maliciously.

```solidity
FILE: 2023-12-ethereumcreditguild/src/loan/AuctionHouse.sol

function startAuction(bytes32 loanId, uint256 callDebt) external {
        // check that caller is a lending term that still has PnL reporting role
        require(
            core().hasRole(CoreRoles.GAUGE_PNL_NOTIFIER, msg.sender),
            "AuctionHouse: invalid caller"
        );

        // check the loan exists in calling lending term and has been called in the current block
        LendingTerm.Loan memory loan = LendingTerm(msg.sender).getLoan(loanId);
        require(
            loan.callTime == block.timestamp,
            "AuctionHouse: loan previously called"
        );

        // check auction for this loan has not already been created
        require(
            auctions[loanId].startTime == 0,
            "AuctionHouse: auction exists"
        );

        // save auction in state
        auctions[loanId] = Auction({
            startTime: block.timestamp,
            endTime: 0,
            lendingTerm: msg.sender,
            collateralAmount: loan.collateralAmount,
            callDebt: callDebt
        });
        nAuctionsInProgress++;

        // emit event
        emit AuctionStart(
            block.timestamp,
            loanId,
            LendingTerm(msg.sender).collateralToken(),
            loan.collateralAmount,
            callDebt
        );
    }

````
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/AuctionHouse.sol#L75-L113

##

## [L-6] Potential address collision risk with the loan ID generation in the ``_borrow`` function 

The function could reject the creation of a new loan if it detects that the generated loan ID already exists, even though it's a new loan request.

If the same borrower requests multiple loans within the same block, there is a theoretical risk of generating the same loan ID. This risk arises because block.timestamp remains constant for all transactions within a single block.

In blockchain transactions, the block timestamp doesn't change from transaction to transaction within the same block. Hence, if a borrower initiates multiple loan requests in transactions that are included in the same block, the only differing factor in the hash function would be the borrower address, which remains the same.

Even a small chance of ID collision could pose a significant risk in financial contexts, as it affects the integrity and reliability of the loan management system.

````solidity
FILE: 2023-12-ethereumcreditguild/src/loan
/LendingTerm.sol

 loanId = keccak256(
            abi.encode(borrower, address(this), block.timestamp)
        );

```
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L347-L349

##

## [L-7] `All pending transactions become DOS if borrower borrows all issuance from particular LendingTerm

A potential Denial-of-Service (DoS) attack on the lending platform, specifically targeting the borrow function in conjunction with the hardcap limit.

The borrow function allows a borrower to take out a loan, provided the total amount borrowed (_postBorrowIssuance) does not exceed the platform's hardcap (params.hardCap). There is no maximum limit for 

An attacker (or a legitimate borrower with enough resources) could potentially borrow an amount that brings the total issuance right up to, or very close to, the hardcap.
This would be done by submitting a borrow transaction with a high borrowAmount that is just below the remaining space under the hardcap.

Once the hardcap is nearly reached or reached, any pending transactions in the mempool that also attempt to borrow funds would fail.
These transactions would be denied service because they would cause the total borrowed amount to exceed the hardcap, thus failing the check _postBorrowIssuance <= params.hardCap.

Borrowers with pending transactions (who may have submitted their requests before the attacker but with lower gas fees) find their transactions failing.
The platform effectively experiences a DoS situation for new borrow requests until the total borrowed amount falls below the hardcap again.

````solidity
FILE: 2023-12-ethereumcreditguild/src/loan/LendingTerm.sol

/// @notice initiate a new loan
    function _borrow(
        address borrower,
        uint256 borrowAmount,
        uint256 collateralAmount
    ) internal returns (bytes32 loanId) {
        require(borrowAmount != 0, "LendingTerm: cannot borrow 0");
        require(collateralAmount != 0, "LendingTerm: cannot stake 0");

        loanId = keccak256(
            abi.encode(borrower, address(this), block.timestamp)
        );

        // check that the loan doesn't already exist
        require(loans[loanId].borrowTime == 0, "LendingTerm: loan exists");

        // check that enough collateral is provided
        uint256 creditMultiplier = ProfitManager(refs.profitManager)
            .creditMultiplier();
        uint256 maxBorrow = (collateralAmount *
            params.maxDebtPerCollateralToken) / creditMultiplier;
        require(
            borrowAmount <= maxBorrow,
            "LendingTerm: not enough collateral"
        );

        // check that enough CREDIT is borrowed
        require(
            borrowAmount >= ProfitManager(refs.profitManager).minBorrow(),
            "LendingTerm: borrow amount too low"
        );

        // check the hardcap
        uint256 _issuance = issuance;
        uint256 _postBorrowIssuance = _issuance + borrowAmount;
        require(
            _postBorrowIssuance <= params.hardCap,
            "LendingTerm: hardcap reached"
        );

        // check the debt ceiling
        uint256 totalBorrowedCredit = ProfitManager(refs.profitManager)
            .totalBorrowedCredit();
        uint256 gaugeWeightTolerance = ProfitManager(refs.profitManager)
            .gaugeWeightTolerance();
        uint256 _debtCeiling = (GuildToken(refs.guildToken)
            .calculateGaugeAllocation(
                address(this),
                totalBorrowedCredit + borrowAmount
            ) * gaugeWeightTolerance) / 1e18;
        if (totalBorrowedCredit == 0) {
            // if the lending term is deprecated, `calculateGaugeAllocation` will return 0, and the borrow
            // should revert because the debt ceiling is reached (no borrows should be allowed anymore).
            // first borrow in the system does not check proportions of issuance, just that the term is not deprecated.
            require(_debtCeiling != 0, "LendingTerm: debt ceiling reached");
        } else {
            require(
                _postBorrowIssuance <= _debtCeiling,
                "LendingTerm: debt ceiling reached"
            );
        }

        // save loan in state
        loans[loanId] = Loan({
            borrower: borrower,
            borrowTime: block.timestamp,
            borrowAmount: borrowAmount,
            borrowCreditMultiplier: creditMultiplier,
            collateralAmount: collateralAmount,
            caller: address(0),
            callTime: 0,
            callDebt: 0,
            closeTime: 0
        });
        issuance = _postBorrowIssuance;
        if (params.maxDelayBetweenPartialRepay != 0) {
            lastPartialRepay[loanId] = block.timestamp;
        }

        // mint debt to the borrower
        RateLimitedMinter(refs.creditMinter).mint(borrower, borrowAmount);

        // pull the collateral from the borrower
        IERC20(params.collateralToken).safeTransferFrom(
            borrower,
            address(this),
            collateralAmount
        );

        // emit event
        emit LoanOpen(
            block.timestamp,
            loanId,
            borrower,
            collateralAmount,
            borrowAmount
        );
    }

````
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L338-L435

### Recommended Mitigation
Add Maximum allowed borrow limits 

##

## [L-8] ``Decreased ``creditMultiplier`` burden to borrowers

### Impact

When the creditMultiplier is decreased, the effective value of each CREDIT token is reduced. This means that borrowers will owe more CREDIT tokens than they initially borrowed to account for the same value.

For example, if a borrower took out a loan of 1,000 CREDIT when the creditMultiplier was 1.0 (or 1e18 in contract terms), and the creditMultiplier is reduced to 0.8 (or 0.8e18), their debt would increase. The new debt would be calculated as 
Original Debt
×
1
New Credit Multiplier
Original Debt× 
New Credit Multiplier
1
​
 , which in this case is 
1
,
000
×
1
0.8
=
1
,
250
1,000× 
0.8
1
​
 =1,250 CREDIT. The borrower now owes 250 more CREDIT than they initially borrowed.

- If the value of the collateral held against the loan does not increase in line with the increase in debt due to the lowering of the creditMultiplier, the loan could become undercollateralized.

- This situation poses a risk of liquidation if the collateral value falls below certain thresholds, adding further financial pressure on the borrower.

```solidity
FILE: 

```


 