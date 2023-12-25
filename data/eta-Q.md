Unbounded For-loops and open access to admin roles leave system vulnerable to DoS attacks, triggering early loan demands and cheap asset fire sales

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/core/CoreRef.sol#L87C1-L107C6https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/core/Core.sol#L10C1-L15C52https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/core/Core.sol#L50C1-L55C6

## Impact
A smart contract managing loans suffers from two major flaws. First, uncontrolled loops processing "Calls" can spiral out of gas, crashing transactions and opening up denial-of-service attacks. This can even sabotage otherwise successful processes. Second, anyone can grant themselves crucial roles like owner or administrator, allowing malicious actors to trigger mass loan demands before due dates, creating bad debt and allowing them to cheaply buy up the distressed assets. To fix this, the contract needs limits on loop iterations and robust error handling, plus an access control system for critical roles to prevent unauthorized manipulation.

## Proof of Concept
Executing emergencyAction() in the CoreRef.sol within unbounded for-loops poses a significant risk of transaction revert, where the number of iterations is unbounded and controlled by the number of Calls. If a large amount of Calls gets added, functionalities, excessive iterations may consume an exorbitant amount of gas, potentially reaching the block gas limit, causing the transaction to fail or revert and then DOS. This can result in unintended outcomes, especially if a majority of the iterations would have otherwise succeeded without the problematic call. 
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/core/CoreRef.sol#L87C1-L107C6

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
 
Furthermore, Core.createRole function can be called by any msg.sender to set CoreRoles.GOVERNOR, owner and admin, since there is no access control. Which means, malicious attackers can take such core roles to make a large number of "calls" to  demand full repayment of loans before the scheduled maturity date.

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/core/Core.sol#L10C1-L15C52

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/core/Core.sol#L50C1-L55C6
```solidity
contract Core is AccessControlEnumerable {
    /// @notice construct Core
    constructor() {
        // For initial setup before going live, deployer can then call
        // renounceRole(bytes32 role, address account)
        _grantRole(CoreRoles.GOVERNOR, msg.sender);
```
```solidity
function createRole(
        bytes32 role,
        bytes32 adminRole
    ) external onlyRole(CoreRoles.GOVERNOR) {
        _setRoleAdmin(role, adminRole);
    }
```
When borrowers are unable to repay their loans, it can lead to a large amount of bad debt. Attackers can then auction off the collateral of these bad debts to purchase them at a discount, allowing them to acquire a large amount of non-performing assets at a low price for arbitrage. 

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/AuctionHouse.sol#L75C5-L80C11

```solidity
function startAuction(bytes32 loanId, uint256 callDebt) external {
        // check that caller is a lending term that still has PnL reporting role
        require(
            core().hasRole(CoreRoles.GAUGE_PNL_NOTIFIER, msg.sender),
            "AuctionHouse: invalid caller"
        );
```

## Tools Used
Foundry

## Recommended Mitigation Steps
1)To enhance contract reliability and manage gas consumption efficiently, consider limiting the maximum number of Calls that can be added and implement robust error-handling mechanisms for potential failures in the looped calls. 2) In the contracts, consider check for address(0) and tracking the addresses of the deployed Core contracts, and to implement access control in the Core.createRole function. 