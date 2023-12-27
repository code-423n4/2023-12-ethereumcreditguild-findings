# Deviation From Check-Effects-Interactions Pattern

The _redeem_ function in the _SimplePSM_ contract performs an external call to _CreditToken.burnFrom_ before updating the state variable _pegTokenBalance_. This ordering of operations deviates from the recommended Check-Effects-Interactions pattern and could potentially expose the contract to reentrancy risks .

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/SimplePSM.sol#L141C16-L141C16
