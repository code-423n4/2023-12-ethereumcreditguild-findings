# `LendingTerm::lastPartialRepay` should be in struct `LendingTerm::loan`

Last partial repay is one of the loans' properties, they should be managed in one struct.

Link: https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L73

Recommendation:
```diff
-    mapping(bytes32 => uint256) public lastPartialRepay;

+    struct Loan {
          ...
+        uint256 lastPartialRepay;
+    }
```