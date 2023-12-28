## [L-1] Initialize  TimelockController in a wrong way in contract GuildTimelockController

### Link:  https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/GuildTimelockController.sol#L15-L25

```solidity
  constructor(
        address _core,
        uint256 _minDelay
    )
        CoreRef(_core)
        TimelockController(
            _minDelay,
            new address[](0),
            new address[](0),
            address(0)
        )
    {}
```
### In OpenZeppelin official contract, it is clearly mentioned fourth params 'admin' is disabled with zero address, but address(0) is passed as admin for TimelockController here, so this is a false initialization
 [https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/governance/TimelockController.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/a72c9561b9c200bac87f14ffd43a8c719fd6fa5a/contracts/governance/TimelockController.sol#L108C87-L108C87)
```solidity
 * - `admin`: optional account to be granted admin role; disable with zero address
     *
     * IMPORTANT: The optional admin can aid with initial configuration of roles after deployment
     * without being subject to delay, but this role should be subsequently renounced in favor of
     * administration through timelocked proposals. Previous versions of this contract would assign
     * this admin to the deployer automatically and should be renounced as well.
     */
```

### Mitigation
Follow instructions in OpenZeppelin Contract, don't use address(0)

## [L-2] Clone should be replaced by CloneDeterministic in LendingTermOnboarding::createTerm function
https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/LendingTermOnboarding.sol#L153

### An attacker can steal funds via a reorg attack if a contract is funded within a few blocks of being created inside a factory.
### cloneDeterministic uses the opcode and a `salt` to deterministically deploy the clone. Using the same `implementation` and `salt` multiple times will revert since the clones cannot be deployed twice at the same address.

### Mitigation
Utilize cloneDeterministic rather than clone
