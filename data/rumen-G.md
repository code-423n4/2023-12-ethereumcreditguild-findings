### Optimizations
File: ```src/core/CoreRef.sol```

Function: emergencyAction
- Cache Call as a variable instead of accessing calls[i] multiple times;
- Use cachedCall.value and cachedCall.callData without assigning variables for them because they are used once

Results: Saves 1,220 gas on average; 6,539 gas on median

| Function Name | min| avg   | median | max   | # calls | When |
|---------------|----|-------|--------|-------|---------|-------|
| emergencyAction | 8236 | 27822 | 16823  | 59517 | 5 | Before edits |
| emergencyAction | 8236 | 26662 | 10284  | 59891 | 5 | After edits |

--------

Before:
```solidity
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

After all edits:

```solidity
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
            Call memory cachedCall = calls[i];
            address payable target = payable(cachedCall.target);

            (bool success, bytes memory returned) = target.call{value: cachedCall.value}(
                cachedCall.callData
            );
            require(success, "CoreRef: underlying call reverted");
            returnData[i] = returned;
        }
    }
```