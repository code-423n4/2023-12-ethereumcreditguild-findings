### [L&#x2011;01]  Guardian can not revoke role in case of emergency which is against Natspec
**Context**: In [CoreRoles](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/core/CoreRoles.sol#L14)

**Description**:
Per `CoreRoles.sol`, `Guardian` role Natspec states,
 
> the protector role. Can pause contracts and revoke roles in an emergency.

However, there is no revoke function provided with access to `Guardian role`. By default, only Governor can revoke role in contracts, However, as the Natspec states the need for revoke role which should be used in case of emergency should be provided.

**Recommendation**:
Add `revoke` function to revoke roles in case of emergency and it should be accessed by `Guardian`

### [L&#x2011;02]  `_bufferCap` is not casted to `uint224` as stored for `bufferStored` in constructor

**Context**: In [RateLimitedV2.sol](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/utils/RateLimitedV2.sol#L48)

**Description**:
In `RateLimitedV2.sol`, `_bufferCap` as uint128 is passed in constructor() and later it is stored as `bufferStored` which is uint224 state variable. 

```solidity
uint224 public bufferStored;
```

`_bufferCap` is directly stored without casting to uint224 and it is best practise to safeCast as the contract has already utilized solmate's `SafeCastLib.sol` and this can be easily mitigated in contract.

**Recommendation**:
Use already imported solmate's `SafeCastLib.sol` to safeCast to uint224.

```diff

    constructor(
        uint256 _maxRateLimitPerSecond,
        uint128 _rateLimitPerSecond,
        uint128 _bufferCap
    ) {
        lastBufferUsedTime = block.timestamp.safeCastTo32();

        _setBufferCap(_bufferCap);
-        bufferStored = _bufferCap;
+        bufferStored = _bufferCap.safeCastTo224();

        require(
            _rateLimitPerSecond <= _maxRateLimitPerSecond,
            "RateLimited: rateLimitPerSecond too high"
        );
        _setRateLimitPerSecond(_rateLimitPerSecond);

        MAX_RATE_LIMIT_PER_SECOND = _maxRateLimitPerSecond;
    }
```

similarly, 

```diff
    function _updateBufferStored(uint128 newBufferCap) internal {
        uint224 newBufferStored = buffer().safeCastTo224();
        uint32 newBlockTimestamp = block.timestamp.safeCastTo32();

        if (newBufferStored > newBufferCap) {
-            bufferStored = uint224(newBufferCap); /// safe upcast as no precision can be lost when going from 128 -> 224
+            bufferStored = newBufferCap.safeCastTo224(); 
            lastBufferUsedTime = newBlockTimestamp;
        } else {
            bufferStored = newBufferStored;
            lastBufferUsedTime = newBlockTimestamp;
        }
    }
```

### [L&#x2011;03]  Set upper and lower bounds for `GuildGovernor.sol` setter functions
**Context**: In [GuildGovernor.sol at L-78--103](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/GuildGovernor.sol#L78-L103)

**Description**:
`GuildGovernor.sol` is the governor contract for on-chain governance of Ethereum Credit Guild. `setVotingDelay()`, `setVotingPeriod()`, `setProposalThreshold()` and `setQuorum()` are the setter functions which does not have upper and lower bound checks, therefore any value can be set. voting delay can indefinitely delayed or quorum can be even set to 0. These functions are called by governor role but it is the best practice to have these checks to overcome possible issues.

**Recommendation**:
Put upper and lower bound checks on `setVotingDelay()`, `setVotingPeriod()`, `setProposalThreshold()` and `setQuorum()`

### [L&#x2011;04]  Use block.timestamp instead of block.number for L2 chains deployment i.e Arbitrum
**Context**: In [GuildGovernor.sol at L-78--103](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/GuildGovernor.sol#L78-L103)

**Description**:

Per the contest readme.md, Contracts will also be deployed on Arbitrum,
> Deployment: we anticipate to launch on Ethereum mainnet & L2s like Arbitrum.

`setVotingDelay()`, `setVotingPeriod()`, `setProposalThreshold()` and `setQuorum()` are the setter functions in governor contract which used [block.number](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/GuildGovernor.sol#L58) as input arguments while setter voting, delay, quorum and threshold variables.

The issue here is with the use of `block.number` in these setter functions as according to Arbitrum official documentation which can be checked [here](https://docs.arbitrum.io/for-devs/concepts/differences-between-arbitrum-ethereum/block-numbers-and-time#ethereum-block-numbers-within-arbitrum)

> Accessing block numbers within an Arbitrum smart contract (i.e., block.number in Solidity) will return a value close to (but not necessarily exactly) the L1 block number at which the Sequencer received the transaction.

```diff
// some Arbitrum contract:
block.number // => returns L1 block number ("ish")
```
Therefore, block.number returns the most recently synced L1 block number. Using block.number in the above setter functions can lead to inaccurate timing as this is not reliable for shorter duration and the function block.number wont work properly as expected due to the Arbitrum official reasoning on shorter term use of block.number.

**Recommendation**:
Use block.timestamp instead of block.number for L2 chains

### [L&#x2011;05]  There is no functionality to `existRebase` in `SimplePSM.sol`
**Context**: In [SimplePSM.sol](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/SimplePSM.sol#L25)

**Description**:
`SimplePSM.sol` has functions like `mintAndEnterRebase()` where the user can enter rebase to earn the savings rate. However, there is no function to exit from rebase. In case, the user want to exit, he can not do that and has to permanently stucked with rebase. 

However, `CreditToken.sol` has [forceExitRebase](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/CreditToken.sol#L69) which can be used to exit the rebase. The rebasing reward can not be claimed by user as the exitRebase function from CreditToken contract which allows to mint rebasing rewards. The user entering the rebase wont be able to exit it with rebasing rewards which would defeat the purpose of entering rebase with `mintAndEnterRebase()` by users.

This could be considered as Medium severity since the important functionality is missing here. The issue is submitting as low as it could be the design decision.

**Recommendation**:
Add function to exit rebase by utlizing `CreditToken.forceExitRebase()` in SimplePSM contract

### [L&#x2011;06]  Incorrect YEAR in seconds will hugely affect lendingTerm creation

**Context**: In [LendingTermOnboarding.sol at L-134](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/LendingTermOnboarding.sol#L134)
In [LendingTerm.sol at L-68](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L68)
In [SurplusGuildMinter.sol at L-29](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/SurplusGuildMinter.sol#L29)

**Description**:

LendingTerm.YEAR() considers the number of seconds in year as `31557600` . However this is incorrect, the number of seconds in year is equal to `31536000`. There is difference of 21600 seconds which is roughly equal to 6 hours. 

This time difference of 6 hours will affect the  lendingTerm creation function in `LendingTermOnboarding.sol` which checks `maxDelayBetweenPartialRepay `  must be less than `LendingTerm.YEAR()` + 1 which is equal to 31557601 seconds.

```solidity

    /// @notice Create a new LendingTerm and initialize it.
    function createTerm(
        address implementation,
        LendingTerm.LendingTermParams calldata params
    ) external returns (address) {

   . . . some code


        require(
            // 31557601 comes from the constant LendingTerm.YEAR() + 1
            params.maxDelayBetweenPartialRepay < 31557601, // periodic payment every [0, 1 year]
            "LendingTermOnboarding: invalid maxDelayBetweenPartialRepay"
        );

   . . . some code

```
The max delay between partial pay should be validated exactly with one year seconds, however current implementation makes a user a loss of 6 hours between payments. This should not be the desired behavior and the user should be allowed to repay the loan within one year approximately. 

**Recommendation**:
Consider the number of seconds in YEAR as `31536000`  in contracts.

For example:

In `LendingTermOnboarding.sol`,

```diff


    /// @notice Create a new LendingTerm and initialize it.
    function createTerm(
        address implementation,
        LendingTerm.LendingTermParams calldata params
    ) external returns (address) {

   . . . some code


        require(
-            // 31557601 comes from the constant LendingTerm.YEAR() + 1
+            // 31536000 comes from the constant LendingTerm.YEAR() + 1
-            params.maxDelayBetweenPartialRepay < 31557601, // periodic payment every [0, 1 year]
+            params.maxDelayBetweenPartialRepay < 31536001 , // periodic payment every [0, 1 year]
            "LendingTermOnboarding: invalid maxDelayBetweenPartialRepay"
        );

   . . . some code

```

and In `LendingTerm.sol`

```diff

    /// @notice reference number of seconds in 1 year
-    uint256 public constant YEAR = 31557600;
+    uint256 public constant YEAR = 31536000;
```

and `SurplusGuildMinter.sol`,

```diff

    /// @notice reference number of seconds in 1 year
-    uint256 public constant YEAR = 31557600;
+    uint256 public constant YEAR = 31536000;
```

### [N&#x2011;01]  Roles should explicitely add `.._ROLL` for better readability
**Context**: In [CoreRoles.sol](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/core/CoreRoles.sol#L12-L77)

**Description**:
`CoreRoles.sol` is a library which consists of different roles used in `Ethereum Credit Guild` contracts. The roles name does not explicitly indicate as ROLE and while reading the contract it creates confusing state and code readability issue. `ROLE` as word should be added in role name. `Openzeppelin` also used such formate while naming roles. For example: ADMIN_ROLE, PROPOSER_ROLE , EXECUTOR_ROLE etc, can be checked [here](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/6ba452dea4258afe77726293435f10baf2bed265/contracts/governance/TimelockController.sol#L25-L27)

**Recommendation**:
Explicitely add .._ROLE in role name for better readability.

For example:
```diff
-    bytes32 internal constant GOVERNOR = keccak256("GOVERNOR_ROLE");
+    bytes32 internal constant GOVERNOR_ROLE = keccak256("GOVERNOR_ROLE");
```

