## 1. Use of Solidity version 0.8.13 which has two known issues.

The solidity version 0.8.13 has below two issues applicable to ECG contracts 

[Vulnerability related to ABI-encoding](https://blog.soliditylang.org/2022/05/18/solidity-0.8.14-release-announcement)
This vulnerability can be misused since the function hashOrder() and hashOppositeOrder() has applicable conditions.
"...pass a nested array directly to another external function call or use abi.encode on it."

[Vulnerability related to 'Optimizer Bug Regarding Memory Side Effects of Inline Assembly'](https://blog.soliditylang.org/2022/06/15/solidity-0.8.15-release-announcement)
ECG inherits solidity contracts from openzeppelin and solmate, and both these uses inline assembly, and optimization is enabled while compiling.

Recommend using recent Solidity version 0.8.15 which has the fix for these issues 

## 2. Core.sol `CreateRole` function should check for role existence before setting role.

```
    function createRole( //creates a new role
        bytes32 role,
        bytes32 adminRole
    ) external onlyRole(CoreRoles.GOVERNOR) { //@note
        _setRoleAdmin(role, adminRole);
    }

```
Lack of check for role existence allows for setting roles to non-existent roles.

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/core/Core.sol#L50
***
## 3. SimplePSM contract will be undeployable if peg token has more that 18 decimals
```
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
        decimalCorrection = 10 ** (18 - decimals); //@note decimals
    }
```
Some ERC20 tokens have more than 18 decimals. Using these tokens as peg tokens will lead to a revert in the `decimalCorrection` calculation, which cause the SimplePSM contract to be undeployable. 


https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SimplePSM.sol#L76

***
## 4. Credit tokens can be used as peg token.
The `SimplePSM` allows to credit tokens be minted or redeemed based on a stable peg token. As, there's no check preventing that the credit token is not the peg token, this introduces the possibility of minting the credit tokens by providing the credit token as the peg token. This can inflate the total supply of credit tokens without the new tokens having any asset backing. The contract, for some reason, error or malicious will be deployed with the credit token as peg token. The `mint` and `mintAndEnterRebase` function, will then require credit will be minted to users, based on credit that they can provide (think FTX), which will inflate the credit totalsupply. 

https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SimplePSM.sol#L65C1-L77C6
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SimplePSM.sol#L103
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SimplePSM.sol#L117
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/8f439544b8a634e8f3c3db718416a39752cd471e/src/loan/SimplePSM.sol#L134

Consider adding a check to the constructor ensuring that `pegToken != credit`.