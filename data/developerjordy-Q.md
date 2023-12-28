## [L-01] Signature from address is nog checked

When a user signs a message, the recovered signer address is not checked if it is from the same address as the signature comes from. 

### Recommended mitigation steps

Add an extra check that makes sure the signature is from the same address

```solidity
function delegateBySig(
+  address from,
  address delegatee,
  uint256 nonce,
  uint256 expiry,
  uint8 v,
  bytes32 r,
  bytes32 s
) internal returns (bool success) { 
	// function logic

+	require(signer != address(0) && signer == from);
}
```

## [L-02] Whole Gauge weight is slashed, even with a loss of 1wei

Malicious actor can look for lending terms where a lot a Credit is staked and has a high gauge weight, and play with the liquidation process until the term get slashed. Resulting in a loss of all tokens being slashed, this happens even with a loss of 1 wei.

The team mentioned to build in partial slashing, but would recommend this doing before deploying. 

### Recommended mitigation steps

Build in partial slashing when applying gauge loss.