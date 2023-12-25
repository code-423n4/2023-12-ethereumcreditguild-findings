# 

# [L-1] Protocol use old solidity version (0.8.13) that contain vulnerabilities

Solidity version 0.8.13 have a known 2 bugs on it, there are :

1. [Inline assembly memory side effect](https://soliditylang.org/blog/2022/06/15/inline-assembly-memory-side-effects-bug/)
2. [Storage write removal before conditional termination](https://soliditylang.org/blog/2022/09/08/storage-write-removal-before-conditional-termination/) 

Consider using new version of solidity.

*There is 20 instances :*

```solidity
File : src/core/Core.sol

2      pragma solidity 0.8.13;
```

```solidity
File : src/core/CoreRef.sol

2      pragma solidity 0.8.13;
```

```solidity
File : src/core/CoreRoles.sol

2      pragma solidity 0.8.13;
```

```solidity
File : src/governance/GuildGovernor.sol

2      pragma solidity 0.8.13;
```

```solidity
File : src/governance/GuildTimelockController.sol

2      pragma solidity 0.8.13;
```

```solidity
File : src/governance/LendingTermOffboarding.sol

2      pragma solidity 0.8.13;
```

```solidity
File : src/governance/LendingTermOnboarding.sol

2      pragma solidity 0.8.13;
```

```solidity
File : src/governance/ProfitManager.sol

2      pragma solidity 0.8.13;
```

```solidity
File : src/governance/GuildVetoGovernor.sol

2      pragma solidity 0.8.13;
```

```solidity
File : src/tokens/ERC20Gauges.sol

2      pragma solidity 0.8.13;
```

```solidity
File : src/tokens/ERC20MultiVotes.sol

2      pragma solidity 0.8.13;
```

```solidity
File : src/tokens/ERC20RebaseDistributor.sol

2      pragma solidity 0.8.13;
```

```solidity
File : src/tokens/CreditToken.sol

2      pragma solidity 0.8.13;
```

```solidity
File : src/tokens/GuildToken.sol

2      pragma solidity 0.8.13;
```

```solidity
File : src/loan/AuctionHouse.sol

2      pragma solidity 0.8.13;
```

```solidity
File : src/loan/LendingTerm.sol

2      pragma solidity 0.8.13;
```

```solidity
File : src/loan/SimplePSM.sol

2      pragma solidity 0.8.13;
```

```solidity
File : src/loan/SurplusGuildMinter.sol

2      pragma solidity 0.8.13;
```

```solidity
File : src/rate-limits/RateLimitedMinter.sol

2      pragma solidity 0.8.13;
```

```solidity
File : src/utils/RateLimitedV2.sol

2      pragma solidity 0.8.13;
```

# [L2] Inheriting function from another contract but no operations are used

There are possible vulnerability related to inheriting a function from other contract (OZ) but no operations are used for that function and there is also a by pass check for systems inherited from it which aims to introduce a new system role in the core contract. This can be problem if the inherited contract is updated and affects the inherited function.

*Instances :*

```solidity
File : src/governance/GuildTimelockController.sol

    /// @dev override of OZ access/AccessControl.sol, noop because role management is handled in Core.
    function _setRoleAdmin(bytes32 role, bytes32 adminRole) internal override {}

    /// @dev override of OZ access/AccessControl.sol, noop because role management is handled in Core.
    function _grantRole(bytes32 role, address account) internal override {}

    /// @dev override of OZ access/AccessControl.sol, noop because role management is handled in Core.
    function _revokeRole(bytes32 role, address account) internal override {}
```