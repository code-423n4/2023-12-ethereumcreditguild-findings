Excess amount of eth sent in the `emergencyAction` function will be in the contract.

##Impact:

User that will send excess eth in the `emergencyAction` will need to proceed to do extra steps to retrieve the ether.


##Mitigation:

Here's how I would fix it:

```
function emergencyAction(
    Call[] calldata calls
)
    external
    payable
    onlyCoreRole(CoreRoles.GOVERNOR)
    returns (bytes[] memory returnData)
{
    returnData = new bytes[](calls.length);
    uint256 totalValue; // Variable to track the total value of ether sent

    for (uint256 i = 0; i < calls.length; i++) {
        address payable target = payable(calls[i].target);
        uint256 value = calls[i].value;
        bytes calldata callData = calls[i].callData;

        totalValue += value; // Accumulate the total value

        (bool success, bytes memory returned) = target.call{value: value}(
            callData
        );
        require(success, "CoreRef: underlying call reverted");
        returnData[i] = returned;
    }

    // Return excess ether to the sender
    if (msg.value > totalValue) {
        payable(msg.sender).transfer(msg.value - totalValue);
    }
}

````

##Links: 

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/core/CoreRef.sol#L87-L107


## The `_minBorrow` argument should be defined in the constructor

##Impact:

The `_minBorrow` variable is set as storage variable with the default value of 100e18, I think that this is certainly not a one fit for all kind of variable, for some tokens holding a larger value it might excessive. I think that this variable should be defined in the constructor. 

Remediation:

    constructor(address _core, uint256 _minBorrow) CoreRef(_core) {
          minBorrow = _minBorrow;
    emit MinBorrowUpdate(block.timestamp,minBorrow);
    
    }

##Links: 
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/ProfitManager.sol#L93-L94

## `ProfitManager` has an unused function

##Impact: 

the `ProfitManager::withdrawFromSurplusBuffer()` function is not being used by any other functions in the protocol, it is best practice to not leave unused pieces of code.

##Remediation:

Remove the unused function 

##Links: 

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/ProfitManager.sol#L267-L275

## Update comments in Auction House

## Impact:

comments indicate that auction with  0 creditAsked can be bidded on, however the code only allows auctions with 0 creditAsked to be `forgive()`. This is confusing, best to update comments.

## Remediation:

Use comments that describe the latest intended actions

## Links: 
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/AuctionHouse.sol#L154-L161

## User that staked in a losing gauge can still accrue rewards, until rewards are claimed

## Impact:

A user that has staked in a gauge that has a registered while in his stake period will still be able to accrue rewards until rewards are claimed, so it is in the user best interest to keep waiting until the gauge doesn't have any activity.

## Remediation:



## Links: 
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/SurplusGuildMinter.sol#L216-L290