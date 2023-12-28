# Gas Optimizations

## Table of contents

- ### [G-01. Useless casting waste gas](#G-01)

- ### [G-02. OR in if-condition can be rewritten as multiple](#G-02)

- ### [G-03. Use immutable rather than constant on unchanging value expressions that use keccack256](#G-03)

- ### [G-04. Unnecessary else statement](#G-04)

- ### [G-05. Use <= instead of <](#G-05)

- ### [G-06. Do not initialize variables with default values](#G-06)

## <a id='G-01'></a>[G-01] Useless casting waste gas

`GuildToken` inherits from `ERC20Gauges` where `getGaugeWeight` storage variable is defined.

```solidity
File: src/governance/ProfitManager.sol

388:    uint256 _gaugeWeight = uint256(
389:        GuildToken(guild).getGaugeWeight(gauge)
390:    );


File: src/tokens/ERC20Gauges.sol

74: mapping(address => uint256) public getGaugeWeight;
```

getGaugeWeight(gauge) is already a uint256 number. Casting this variable in `ProfitManager::notifyPnL()` wastes gas.

Below are simple forge tests, which demonstrate gas usage for both versions.

```solidity
function testUselessCasting() public {
    uint256 _gaugeWeight = uint256(
        guild.getGaugeWeight(gauge1)
    );
}
function testUselessCasting2() public {
    uint256 _gaugeWeight = guild.getGaugeWeight(gauge1);
}
```

![useless casting 110 gas](https://gist.github.com/assets/28733538/d19754aa-730c-4217-9b97-d7eb9e977f05)

This shows that useless casting should be removed as it consumes 110 more gas.

## <a id='G-02'></a>[G-02] OR in if-condition can be rewritten as multiple if-conditions

If the if statement has a logical operator `||` and is not followed by an else statement, it can be replaced with multiple if statements.

```solidity
File: src/loan/SurplusGuildMinter.sol

298:    if (userStake.stakeTime == 0 || slashed) return;
```

Below are simple forge tests, which demonstrate gas usage for both versions.

```solidity
function test_IfWithOR(uint48 stakeTime, bool slashed) public {
    if (stakeTime == 0) return;
    if (slashed) return;
}

function test_IfWithoutOR(uint48 stakeTime, bool slashed) public {
    if (stakeTime == 0) return;
    if (slashed) return;
}
```

![split if statement](https://gist.github.com/assets/28733538/f2eb4eef-5dc5-49a4-a677-ca08d0058b49)

This shows that if-conditions with `||` operator should be split into multiple single if-conditions.

## <a id='G-03'></a>[G-03] Use immutable rather than constant on unchanging value expressions that use keccack256()

`Constant` variables are evaluated at runtime and their value are included in the bytecode of the contract. Any expensive operation performed as part of the constant expression, such as a call to `keccak256()`, will be executed every time the contract is deployed, even if the result is always the same.

On the other hand, `immutable` variables are evaluated at compilation time, and their values are included in the bytecode of the contract as constants. Any expensive operation performed as part of the immutable expression is only executed once, when the contract is compiled. Then the result is reused every time the contract is deployed. This can result in lower gas costs compared to using constant variables.

```solidity
File: src/core/CoreRoles.sol

12: bytes32 internal constant GOVERNOR = keccak256("GOVERNOR_ROLE");

15: bytes32 internal constant GUARDIAN = keccak256("GUARDIAN_ROLE");

20: bytes32 internal constant CREDIT_MINTER = keccak256("CREDIT_MINTER_ROLE");

23: bytes32 internal constant RATE_LIMITED_CREDIT_MINTER =

27: bytes32 internal constant GUILD_MINTER = keccak256("GUILD_MINTER_ROLE");

30: bytes32 internal constant RATE_LIMITED_GUILD_MINTER =

36: bytes32 internal constant GAUGE_ADD = keccak256("GAUGE_ADD_ROLE");

39: bytes32 internal constant GAUGE_REMOVE = keccak256("GAUGE_REMOVE_ROLE");

42: bytes32 internal constant GAUGE_PARAMETERS =

46: bytes32 internal constant GAUGE_PNL_NOTIFIER =

50: bytes32 internal constant GUILD_GOVERNANCE_PARAMETERS =

54: bytes32 internal constant GUILD_SURPLUS_BUFFER_WITHDRAW =

60: bytes32 internal constant CREDIT_GOVERNANCE_PARAMETERS =

64: bytes32 internal constant CREDIT_REBASE_PARAMETERS =

71: bytes32 internal constant TIMELOCK_PROPOSER = keccak256("PROPOSER_ROLE");

74: bytes32 internal constant TIMELOCK_EXECUTOR = keccak256("EXECUTOR_ROLE");

77: bytes32 internal constant TIMELOCK_CANCELLER = keccak256("CANCELLER_ROLE");


src/tokens/ERC20MultiVotes.sol

467:    bytes32 public constant DELEGATION_TYPEHASH =
468:        keccak256("Delegation(address delegatee,uint256 nonce,uint256 expiry)");
```

## <a id='G-04'></a>[G-04] Unnecessary else statement

```solidity
File: src/governance/GuildVetoGovernor.sol

292:    else {
293:        return ProposalState.Active;
294:    }

298:    else {
299:        return ProposalState.Defeated;
300:    }


File: src/tokens/ERC20RebaseDistributor.sol

144:    } else {
145:        // block.timestamp is within [lastTimestamp, targetTimestamp[
146:        uint256 elapsed = block.timestamp - lastTimestamp;
147:        uint256 delta = targetValue - lastValue;
148:        return
149:            lastValue +
150:            (delta * elapsed) /
151:            (targetTimestamp - lastTimestamp);
152:    }

408L    } else {
409:        return _totalSupply - _rebasingSupply;
410:    }

432:    } else {
433:        return
434:            _shares2balance(
435:                _rebasingState.nShares,
436:                rebasingSharePrice(),
437:                0,
438:                ERC20.balanceOf(account)
439:            );
440:    }
```

## <a id='G-05'></a>[G-05] Use <= instead of <

```solidity
File: src/loan/LendingTerm.sol

291:    _hardCap < creditMinterBuffer ? _hardCap : creditMinterBuffer;

303:    _hardCap < creditMinterBuffer ? _hardCap : creditMinterBuffer;

317:    _hardCap < creditMinterBuffer ? _hardCap : creditMinterBuffer;
```

## <a id='G-06'></a>[G-06] Do not initialize variables with default values

```solidity
File: src/governance/ProfitManager.sol

443:    for (uint256 i = 0; i < gauges.length; ) {

467:    for (uint256 i = 0; i < gauges.length; ) {


File: src/loan/LendingTerm.sol

685:    for (uint256 i = 0; i < loanIds.length; i++) {


File: src/tokens/ERC20MultiVotes.sol

109:    uint256 low = 0;

442:    uint256 i = 0;


File: src/tokens/ERC20Gauges.sol

280:    for (uint256 i = 0; i < size; ) {

352:    for (uint256 i = 0; i < size; ) {

516:    uint256 i = 0;


File: src/core/CoreRef.sol

96:     for (uint256 i = 0; i < calls.length; i++) {
```
