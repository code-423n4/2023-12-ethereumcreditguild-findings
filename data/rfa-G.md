# Gas report

## Table Of Contents

- [Gas report](#gas-report)
  - [Table Of Contents](#table-of-contents)
  - [Removing user parameter on _incrementUserAndGlobalWeights function](#removing-user-parameter-on-_incrementuserandglobalweights-function)
  - [Caching `auctions[loanId]` inside memory might save a lot of gas](#caching-auctionsloanid-inside-memory-might-save-a-lot-of-gas)


## Unnecessary parameter declaration

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20Gauges.sol#L250
```solidity
249: function _incrementUserAndGlobalWeights(

250:    address user,
```

Since the argument for `user` will always `msg.sender`, it's unnecessary to declare it as a parameter (saves 7 gas per call):

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/tokens/ERC20Gauges.sol#L225
```solidity
225: return _incrementUserAndGlobalWeights(msg.sender, weight);

//
292: return _incrementUserAndGlobalWeights(msg.sender, weightsSum);

```

Same case happens in [_call](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/LendingTerm.sol#L635) function

```solidity
634:    function _call(

635:        address caller,
```

```solidity
679:        _call(msg.sender, loanId, refs.auctionHouse);


686:            _call(msg.sender, loanIds[i], _auctionHouse);
```

### Recommended Mitigation:

```diff
--- a/src/tokens/ERC20Gauges.sol  2023-12-26 17:45:18
+++ b/src/tokens/ERC20Gauges.sol      2023-12-27 15:48:58
@@ -222,7 +222,7 @@
     ) public virtual returns (uint256 newUserWeight) {
         require(isGauge(gauge), "ERC20Gauges: invalid gauge");
         _incrementGaugeWeight(msg.sender, gauge, weight);
-        return _incrementUserAndGlobalWeights(msg.sender, weight);
+        return _incrementUserAndGlobalWeights(weight);
     }
 
     /// @dev this function does not check if the gauge exists, this is performed
@@ -247,15 +247,14 @@
     }
 
     function _incrementUserAndGlobalWeights(
-        address user,
         uint256 weight
     ) internal returns (uint256 newUserWeight) {
-        newUserWeight = getUserWeight[user] + weight;
+        newUserWeight = getUserWeight[msg.sender] + weight;
         // Ensure under weight
-        require(newUserWeight <= balanceOf(user), "ERC20Gauges: overweight");
+        require(newUserWeight <= balanceOf(msg.sender), "ERC20Gauges: overweight");
 
         // Update gauge state
-        getUserWeight[user] = newUserWeight;
+        getUserWeight[msg.sender] = newUserWeight;
 
         totalWeight += weight;
     }
@@ -289,7 +288,7 @@
                 ++i;
             }
         }
-        return _incrementUserAndGlobalWeights(msg.sender, weightsSum);
+        return _incrementUserAndGlobalWeights(weightsSum);
     }
 
     /** 
```

## Caching `auctions[loanId]` inside memory might save a lot of gas

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/loan/AuctionHouse.sol#L166-L196

`auctions[loanId]` is called multiple times inside the `getBidDetail` function. It also does assignment of `loanId` var as a key into `auction` getter function to get the value of `Auction` multiple times (in case, if we still want to pointing to storage)

### Recommended Mitigation

```diff
--- a/src/loan/AuctionHouse.sol   2023-12-26 17:45:18
+++ b/src/loan/AuctionHouse.sol       2023-12-29 01:07:52
@@ -118,12 +118,13 @@
     function getBidDetail(
         bytes32 loanId
     ) public view returns (uint256 collateralReceived, uint256 creditAsked) {
         // Cache auctions[loanId]
+        Auction memory _cache = auctions[loanId];
         // check the auction for this loan exists
-        uint256 _startTime = auctions[loanId].startTime;
+        uint256 _startTime = _cache.startTime;
         require(_startTime != 0, "AuctionHouse: invalid auction");
 
         // check the auction for this loan isn't ended
-        require(auctions[loanId].endTime == 0, "AuctionHouse: auction ended");
+        require(_cache.endTime == 0, "AuctionHouse: auction ended");
 
         // assertion should never fail because when an auction is created,
         // block.timestamp is recorded as the auction start time, and we check in previous
@@ -133,29 +134,30 @@
         // first phase of the auction, where more and more collateral is offered
         if (block.timestamp < _startTime + midPoint) {
             // ask for the full debt
-            creditAsked = auctions[loanId].callDebt;
+            creditAsked = _cache.callDebt;
 
             // compute amount of collateral received
             uint256 elapsed = block.timestamp - _startTime; // [0, midPoint[
-            uint256 _collateralAmount = auctions[loanId].collateralAmount; // SLOAD
-            collateralReceived = (_collateralAmount * elapsed) / midPoint;
+            // No need to cache anymore
+            // uint256 _collateralAmount = _cache.collateralAmount; // SLOAD
+            collateralReceived = (_cache.collateralAmount; * elapsed) / midPoint;
         }
         // second phase of the auction, where less and less CREDIT is asked
         else if (block.timestamp < _startTime + auctionDuration) {
             // receive the full collateral
-            collateralReceived = auctions[loanId].collateralAmount;
+            collateralReceived = _cache.collateralAmount;
 
             // compute amount of CREDIT to ask
             uint256 PHASE_2_DURATION = auctionDuration - midPoint;
             uint256 elapsed = block.timestamp - _startTime - midPoint; // [0, PHASE_2_DURATION[
-            uint256 _callDebt = auctions[loanId].callDebt; // SLOAD
+            uint256 _callDebt = _cache.callDebt; // SLOAD
             creditAsked = _callDebt - (_callDebt * elapsed) / PHASE_2_DURATION;
         }
         // second phase fully elapsed, anyone can receive the full collateral and give 0 CREDIT
         // in practice, somebody should have taken the arb before we reach this condition.
         else {
             // receive the full collateral
-            collateralReceived = auctions[loanId].collateralAmount;
+            collateralReceived = _cache.collateralAmount;
             //creditAsked = 0; // implicit
         }
     }
```