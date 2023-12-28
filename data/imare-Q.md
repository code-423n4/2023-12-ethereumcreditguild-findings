
### LendingTermOffboarding poll duration will not work correctly if deployed on Arbitrum

From the github [documentation](https://github.com/code-423n4/2023-12-ethereumcreditguild#additional-context) the deployment is expected to launch on Ethereum mainnet & L2s like Arbitrum.

If the contract is deployed as it is on L2 like Arbitrum the functionality for proposing offboard polls will not work as expected.

## Proof of Concept

By using a `block.number` for the constant variable that limits the creation of term offboarding proposal 

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/LendingTermOffboarding.sol#L36

on ethereum mainnet will limit to a proposal a week ..average block time generation is 13 seconds.
on arbitrum L2 will be just one day ...average block time generation is 2 seconds.

also voting for offboarding support will be limited. Less time on the L2 then mainnet chain :

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/LendingTermOffboarding.sol#L120-L123

## Recommended Mitigation Steps

When constructing the contract on a specific chain pass the value of block creation or use `block.timestamp` instead like is done in `AuctionHouse` contract



### when changing to a new auction house from the lending term there is no check if the new auction has already other auctions in progress

when the change happens 
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L836-L841

and if it happens that the GOVERNOR needs to immediately change to another auction house it can also immediately fail becouse the check of the number of auction in progress is only done on the current auction house and not the new one.

## Recommended Mitigation Steps

Restrict changing in both direction :

```diff
    function setAuctionHouse(
        address _newValue
    ) external onlyCoreRole(CoreRoles.GOVERNOR) {
        // allow configuration changes only when there are no auctions in progress.
        // updating the auction house while auctions are in progress could break the loan
        // lifecycle, as it would prevent the former auctionHouse (that have active auctions)
        // from reporting the result to the lending term.
        require(
            AuctionHouse(refs.auctionHouse).nAuctionsInProgress() == 0,
            "LendingTerm: auctions in progress"
        );

+       require(
+           AuctionHouse(_newValue).nAuctionsInProgress() == 0,
+           "LendingTerm: auctions in progress for new auction house"
+       );
+
        refs.auctionHouse = _newValue;
    }
```



### token support in the SimplePSM is limited to only token with 18 or lover decimals

This code will revert for token that have more then 18 decimals

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/SimplePSM.sol#L75C1-L76

## Solution

```diff
    constructor(
        address _core,
        address _profitManager,
        address _credit,
        address _pegToken
    ) CoreRef(_core) {
        profitManager = _profitManager;
        credit = _credit;
        pegToken = _pegToken;


        uint256 decimals = uint256(ERC20(_pegToken).decimals());
-       decimalCorrection = 10 ** (18 - decimals);
+       if (decimals < 18) {
+           decimalCorrection = 10**(18 - decimals);
+       } else if (decimals > 18) {
+           decimalCorrection = 10**(decimals - 18);
+       } else {
+           decimalCorrection = 1; // Already has 18 decimal places
+       }
    }
```


### RateLimitedV2#_updateBufferStored can be simplified

```diff
    function _updateBufferStored(uint128 newBufferCap) internal {
        uint224 newBufferStored = buffer().safeCastTo224();
-       uint32 newBlockTimestamp = block.timestamp.safeCastTo32();
+       lastBufferUsedTime = block.timestamp.safeCastTo32();

        if (newBufferStored > newBufferCap) {
            bufferStored = uint224(newBufferCap); /// safe upcast as no precision can be lost when going from 128 -> 224
-           lastBufferUsedTime = newBlockTimestamp;
        } else {
            bufferStored = newBufferStored;
-           lastBufferUsedTime = newBlockTimestamp;
        }
    }
}
```