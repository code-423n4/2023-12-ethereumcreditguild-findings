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

## [L-5] CoreRoles.GAUGE_PNL_NOTIFIER can startAuction even that particular loan is not matured 

 