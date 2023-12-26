# [QA-01] Functions which update the value do not verify if the value was really changed

Whenever we update/set a new value of some state variable, it's a good practice to make sure that, the value is being indeed updated.
In the current implementation of `_setQuorum()` in `GuildGovernor.sol`, even when the `_quorum` won't be changed, function will still emit an `QuorumUpdated()` event - which might be misleading to the end-user.
E.g. let's assume that `_quorum = 123`. Calling `_setQuorum(123)` won't update/set any new quorum, but `QuorumUpdated()` will still be called.

Our recommendation is to add additional `require` check, to verify, that provided values are indeed different than the current ones:

```
require(_quorum  != newQuorum, "Nothing to update");
```

The same issue occurs in multiple of instances:
`CoreRef.sol`: `_setCore()`
`LendingTermOffboarding.sol`: `setQuorum()`
`ProfitManager.sol`: `setMinBorrow()`, `setGaugeWeightTolerance()`, `setProfitSharingConfig()`
`GuildVetoGovernor.sol`: `_setQuorum()`, `_updateTimelock()`
`ERC20MultiVotes.sol`: `_setMaxDelegates()`
`GuildToken.sol`: `setProfitManager()`
`SimplePSM.sol`: `setRedemptionsPaused()`, `setMintRatio()`, `setRewardRatio()`

# [QA-02] Lack of `address(0)` check in `setCore()` in `CoreRef.sol`

[File: CoreRef.sol](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/core/CoreRef.sol#L39)
```
  function setCore(
        address newCore
    ) external onlyCoreRole(CoreRoles.GOVERNOR) {
        _setCore(newCore);
    }

[...]

    function _setCore(address newCore) internal {
        address oldCore = address(_core);
        _core = Core(newCore);

        emit CoreUpdate(oldCore, newCore);
    }
```
There's no additional check which verifies, if `newCore` address, passed by the user is not `address(0)`.
Perform additional check: `require(newCore != address(0), "newCore cannot be addr(0)")`.


# [QA-03] Lack of `address(0)` check in `allowImplementation()` in `LendingTermOnboarding.sol`

[File: LendingTermOnboarding.sol](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/LendingTermOnboarding.sol#L92)
```
    function allowImplementation(
[...]
        implementations[implementation] = allowed;
[...]
```

There's no additional check which verifies, if `implementation` address, passed by the user is not `address(0)`.
Perform additional check: `require(implementation != address(0), "newCore cannot be addr(0)")`.

# [QA-04] Lack of `address(0)` check in `updateTimelock()` in `GuildVetoGovernor.sol`

[File: GuildVetoGovernor.sol](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/GuildVetoGovernor.sol#L93)
```
 function updateTimelock(
        address newTimelock
    ) external onlyCoreRole(CoreRoles.GOVERNOR) {
        _updateTimelock(newTimelock);
    }

    function _updateTimelock(address newTimelock) private {
        emit TimelockChange(timelock, newTimelock);
        timelock = newTimelock;
    }
```

There's no additional check which verifies, if `newTimelock` address, passed by the user is not `address(0)`.
Perform additional check: `require(newTimelock != address(0), "newTimelock cannot be addr(0)")`.


# [QA-05] Changing critical parameters should emit an event with both old and new value

[File: ProfitManager.sol](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/ProfitManager.sol#L179)
```
function setMinBorrow(
        uint256 newValue
    ) external onlyCoreRole(CoreRoles.GOVERNOR) {
        _minBorrow = newValue;
        emit MinBorrowUpdate(block.timestamp, newValue);
    }
```

`MinBorrowUpdate()` should emit both old (before the update) and new (after the update) values. The current implementation emits an event only with a new value.

The same issue occurs in multiple of instances:
`ProfitManager.sol`: `setGaugeWeightTolerance()`, `setProfitSharingConfig()`.
`GuildToken.sol`: `setProfitManager()`
`SimplePSM.sol`: `setMintRatio()`, `setRewardRatio()`


# [R-01] Rename `createRole()` in `Core.sol`

[File: Core.sol](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/core/Core.sol#L46)
```
/// @notice creates a new role to be maintained
    /// @param role the new role id
    /// @param adminRole the admin role id for `role`
    /// @dev can also be used to update admin of existing role
    function createRole(
        bytes32 role,
        bytes32 adminRole
    ) external onlyRole(CoreRoles.GOVERNOR) {
        _setRoleAdmin(role, adminRole);
    }
```

As the NatSpec comment states, `createRole()`: `can also be used to update admin of existing role`.
This suggests, that the name of the function should be changed to: `createOrUpdateRole()`.

# [R-02] `renounceRole()` in `Core.sol` should be overriden to disallow renouncing `CoreRoles.GOVERNOR` role
Since `Core` inherits from `AccessControlEnumerable`, it implements additional `AccessControl` functions. One of those functions is `renounceRole()`.

`CoreRoles.GOVERNOR` is the most important role in the protocol. Renouncing this role basically means that protocol won't be able to execute most of the operation.
Our recommendation is to override `renounceRole()` so that it won't be possible to accidently renounce this role:

```
    function renounceRole(bytes32 role, address callerConfirmation) public override {
       if (callerConfirmation  == CoreRoles.GOVERNOR && role == CoreRoles.GOVERNOR ) revert("Cannot renounce CoreRoles.GOVERNOR!");
       super.renounceRole(role, callerConfirmation);
    }
```


# [R-03] Minimum delay between proposals of onboarding of a given term cannot be updated

[File: LendingTermOnboarding.sol](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/LendingTermOnboarding.sol#L25)
```
uint256 public constant MIN_DELAY_BETWEEN_PROPOSALS = 7 days;
```

The minimum delay between proposals of onboarding of a given term is represented as `constant` variable: `MIN_DELAY_BETWEEN_PROPOSALS`. This implies, that this value cannot be updated by anyone.
Our recommendation is to change this variable to a non-constant one and implement a function which allows to modify its value in the future. 


# [R-04] Rename `allowImplementation()` in `LendingTermOnboarding.sol`

[File: LendingTermOnboarding.sol](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/LendingTermOnboarding.sol#L91)
```
/// @notice Allow or disallow a given implemenation
    function allowImplementation(
[...]
        implementations[implementation] = allowed;
[...]
```

Function `allowImplementation()` can either allow or disallow a given implementation, while its name suggests that it only allows new implementations. Our recommendation is to change the name of this function to: `allowOrDisallowImplementation()`.

# [R-05] Minimum number of CREDIT to stake cannot be updated

[File: SurplusGuildMinter.sol](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/SurplusGuildMinter.sol#L26)
```
uint256 public constant MIN_STAKE = 1e18;
```
The minimum number of CREDIT to stake is represented as `constant` variable: `MIN_STAKE`. This implies, that this value cannot be updated by anyone.
Our recommendation is to change this variable to a non-constant one and implement a function which allows to modify its value in the future. 

# [R-06] Some timestamp-related fields from events can be removed

There's no need to pass `block.timestamp` as event parameter, since this info can be get directly from off-chain (we can see when exactly event was emitted).
There are multiple of events which implements `timestamp` or `when` parameter and then pass `block.timestamp`. These fields can be removed, since there's no need to pass them into events.

```
LendingTermOffboarding.sol:
    event OffboardSupport(uint256 indexed timestamp, [...]
    event Offboard(uint256 indexed timestamp, address indexed term);
    event Cleanup(uint256 indexed timestamp, address indexed term);

LendingTermOnboarding.sol:
    event ImplementationAllowChanged(uint256 indexed when, [...]
    event TermCreated(uint256 indexed when, [...]

ProfitManager.sol:
    event GaugePnL(address indexed gauge, uint256 indexed when, int256 pnl);
    event SurplusBufferUpdate(uint256 indexed when, uint256 newValue);
    event TermSurplusBufferUpdate(uint256 indexed when, [...]
    event CreditMultiplierUpdate(uint256 indexed when, uint256 newValue);
    event ProfitSharingConfigUpdate(uint256 indexed when, [...]
    event ClaimRewards(uint256 indexed when, [...] 
    event MinBorrowUpdate(uint256 indexed when, uint256 newValue);    
    event GaugeWeightToleranceUpdate(uint256 indexed when, uint256 newValue);

ERC20RebaseDistributor.sol:
    event RebaseEnter(address indexed account, uint256 indexed timestamp);
    event RebaseExit(address indexed account, uint256 indexed timestamp);
    event RebaseDistribution(address indexed source, uint256 indexed timestamp, [...]    
    event RebaseReward(address indexed account, uint256 indexed timestamp, [...]

GuildToken.sol:
	event GaugeLoss(address indexed gauge, uint256 indexed when);
	event GaugeLossApply([...],  uint256 when);
	event TransfersEnabled(uint256 block, uint256 timestamp);
	event ProfitManagerUpdated(uint256 timestamp, address newValue);

AuctionHouse.sol:
	event AuctionStart(uint256 indexed when, [...]
	event AuctionEnd(uint256 indexed when, [...]    

LendingTerm.sol:
	event LoanOpen(uint256 indexed when, [...]
	event LoanCall(uint256 indexed when, bytes32 indexed loanId);
	event LoanClose( uint256 indexed when, [...]
	event LoanAddCollateral(uint256 indexed when,, [...]
	event LoanPartialRepay(uint256 indexed when, [...]

SimplePSM.sol:
	event Mint(uint256 indexed when, [...]
	event RedemptionsPaused(uint256 indexed when, bool status);   

SurplusGuildMinter.sol:
	event Stake(uint256 indexed timestamp, [...]
	event Unstake(uint256 indexed timestamp, [...]
	event GuildReward(uint256 indexed timestamp, [...]
	event MintRatioUpdate(uint256 indexed timestamp, uint256 ratio);
	event RewardRatioUpdate(uint256 indexed timestamp, uint256 ratio);
```



# [NC-01] Sort roles in an alphabetical order in `Core.sol`

[File: Core.sol](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/core/Core.sol#L18)
```
 _setRoleAdmin(CoreRoles.GOVERNOR, CoreRoles.GOVERNOR);
        _setRoleAdmin(CoreRoles.GUARDIAN, CoreRoles.GOVERNOR);
        _setRoleAdmin(CoreRoles.CREDIT_MINTER, CoreRoles.GOVERNOR);
        _setRoleAdmin(CoreRoles.RATE_LIMITED_CREDIT_MINTER, CoreRoles.GOVERNOR);
        _setRoleAdmin(CoreRoles.GUILD_MINTER, CoreRoles.GOVERNOR);
        _setRoleAdmin(CoreRoles.RATE_LIMITED_GUILD_MINTER, CoreRoles.GOVERNOR);
        _setRoleAdmin(CoreRoles.GAUGE_ADD, CoreRoles.GOVERNOR);
        _setRoleAdmin(CoreRoles.GAUGE_REMOVE, CoreRoles.GOVERNOR);
        _setRoleAdmin(CoreRoles.GAUGE_PARAMETERS, CoreRoles.GOVERNOR);
        _setRoleAdmin(CoreRoles.GAUGE_PNL_NOTIFIER, CoreRoles.GOVERNOR);
        _setRoleAdmin(
            CoreRoles.GUILD_GOVERNANCE_PARAMETERS,
            CoreRoles.GOVERNOR
        );
        _setRoleAdmin(
            CoreRoles.GUILD_SURPLUS_BUFFER_WITHDRAW,
            CoreRoles.GOVERNOR
        );
        _setRoleAdmin(
            CoreRoles.CREDIT_GOVERNANCE_PARAMETERS,
            CoreRoles.GOVERNOR
        );
        _setRoleAdmin(CoreRoles.CREDIT_REBASE_PARAMETERS, CoreRoles.GOVERNOR);
        _setRoleAdmin(CoreRoles.TIMELOCK_PROPOSER, CoreRoles.GOVERNOR);
        _setRoleAdmin(CoreRoles.TIMELOCK_EXECUTOR, CoreRoles.GOVERNOR);
        _setRoleAdmin(CoreRoles.TIMELOCK_CANCELLER, CoreRoles.GOVERNOR);
        [...]
```

`constructor()` sets multiple of roles by calling `_setRoleAdmin()` function. Since multiple of roles are being set at once, and the order of `_setRoleAdmin()` does not matter,  to increase the code readability, our recommendation is to set those roles in an alphabetical order.

E.g., in the current code-base, the very similar roles `RATE_LIMITED_CREDIT_MINTER`, `RATE_LIMITED_GUILD_MINTER`, are separated by `_setRoleAdmin(CoreRoles.GUILD_MINTER, CoreRoles.GOVERNOR);`. Sorting those roles in an alphabetical order would allow to avoid that issue and make the code readability clearer:

```
_setRoleAdmin(CoreRoles.CREDIT_GOVERNANCE_PARAMETERS,CoreRoles.GOVERNOR);
_setRoleAdmin(CoreRoles.CREDIT_MINTER, CoreRoles.GOVERNOR);
_setRoleAdmin(CoreRoles.CREDIT_REBASE_PARAMETERS, CoreRoles.GOVERNOR);
_setRoleAdmin(CoreRoles.GAUGE_ADD, CoreRoles.GOVERNOR);
_setRoleAdmin(CoreRoles.GAUGE_PARAMETERS, CoreRoles.GOVERNOR);
_setRoleAdmin(CoreRoles.GAUGE_PNL_NOTIFIER, CoreRoles.GOVERNOR);
_setRoleAdmin(CoreRoles.GAUGE_REMOVE, CoreRoles.GOVERNOR);
_setRoleAdmin(CoreRoles.GOVERNOR, CoreRoles.GOVERNOR);
_setRoleAdmin(CoreRoles.GUARDIAN, CoreRoles.GOVERNOR);
_setRoleAdmin(CoreRoles.GUILD_GOVERNANCE_PARAMETERS, CoreRoles.GOVERNOR);
_setRoleAdmin(CoreRoles.GUILD_MINTER, CoreRoles.GOVERNOR);
_setRoleAdmin(CoreRoles.GUILD_SURPLUS_BUFFER_WITHDRAW, CoreRoles.GOVERNOR);
_setRoleAdmin(CoreRoles.RATE_LIMITED_CREDIT_MINTER, CoreRoles.GOVERNOR);
_setRoleAdmin(CoreRoles.RATE_LIMITED_GUILD_MINTER, CoreRoles.GOVERNOR);
_setRoleAdmin(CoreRoles.TIMELOCK_CANCELLER, CoreRoles.GOVERNOR);
_setRoleAdmin(CoreRoles.TIMELOCK_EXECUTOR, CoreRoles.GOVERNOR);
_setRoleAdmin(CoreRoles.TIMELOCK_PROPOSER, CoreRoles.GOVERNOR);
```


The same issue occurs in `CoreRoles.sol`. Defined roles:

```
[...]
 bytes32 internal constant CREDIT_MINTER = keccak256("CREDIT_MINTER_ROLE");

    /// @notice can mint CREDIT within rate limits & cap
    bytes32 internal constant RATE_LIMITED_CREDIT_MINTER =
        keccak256("RATE_LIMITED_CREDIT_MINTER_ROLE");

    /// @notice can mint GUILD arbitrarily
    bytes32 internal constant GUILD_MINTER = keccak256("GUILD_MINTER_ROLE");

    /// @notice can mint GUILD within rate limits & cap
    bytes32 internal constant RATE_LIMITED_GUILD_MINTER =
        keccak256("RATE_LIMITED_GUILD_MINTER_ROLE");
[...]
```

should be re-ordered alphabetically.


# [NC-02] Add keccak256 values to roles in `CoreRoles.sol`

It's a good practice which increases the code-readability when `keccak256` from constants are pre-calculated and inserted as comments.
E.g. below line:

[File: CoreRoles.sol](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/core/CoreRoles.sol#L35C3-L35C4)
```
/// @notice can manage add new gauges to the system
    bytes32 internal constant GAUGE_ADD = keccak256("GAUGE_ADD_ROLE");
```

can be changed to:

```
/// @notice can manage add new gauges to the system
/// bytes32 internal constant GAUGE_ADD = 0xf213c52f17fbe7c12e60dedf809e77244577de25e041307d406900a545756831;
    bytes32 internal constant GAUGE_ADD = keccak256("GAUGE_ADD_ROLE");
```

# [NC-03] Incorrect Markdown formatting in README.md

[File: README.md](https://raw.githubusercontent.com/code-423n4/2023-12-ethereumcreditguild/main/README.md)
```
| [`governance/GuildTimelockController.sol`](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildTimelockController.sol| 27 |
```

Markdown syntax does not contain closing `)`, which means that the hyperlink is not properly rendered.
Above code should be changed to:

```
| [`governance/GuildTimelockController.sol`](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/main/src/governance/GuildTimelockController.sol)| 27 |
```

# [NC-04] Use previously defined constants instead of magic-numbers

[File: LendingTermOnboarding.sol](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/governance/LendingTermOnboarding.sol#L133-L134)
```
 // 31557601 comes from the constant LendingTerm.YEAR() + 1
            params.maxDelayBetweenPartialRepay < 31557601, // periodic payment every [0, 1 year]
```

Use previously defined `LendingTerm.YEAR() + 1`, instead of magic-number: `31557601`. Please notice, that `@src/loan/LendingTerm.sol` is already imported in `LendingTermOnboarding.sol`, which implies, we can use `LendingTerm.YEAR()` straightforwardly in the code-base.

# [NC-05] Use camel-case syntax for variable naming in `GuildVetoGovernor.sol`
Some of the variables does not use camel-case syntax.

```
File: GuildVetoGovernor.sol
175:	ProposalVote storage proposalvote = _proposalVotes[proposalId]; 
// should be: ProposalVote storage proposalVote = _proposalVotes[proposalId]; 

202:	ProposalVote storage proposalvote = _proposalVotes[proposalId];
// should be: ProposalVote storage proposalVote = _proposalVotes[proposalId];

250:	bytes32 queueid = _timelockIds[proposalId];
// should be: bytes32 queueId = _timelockIds[proposalId];

```

# [NC-06] `require` message should not be empty

Use descriptive messages which will explain why the function reverted.

[File: ERC20MultiVotes.sol](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/tokens/ERC20MultiVotes.sol#L503)
```
require(signer != address(0));
```

# [NC-07] Comment in `LendingTerm.sol` does not properly explain `YEAR` value

[File: LendingTerm.sol](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/LendingTerm.sol#L67-L68)
```
/// @notice reference number of seconds in 1 year
uint256 public constant YEAR = 31557600;
```

The comment suggests, that `YEAR` constant should be `365 * 24 * 60 * 60`, which is `31536000`. However, the value of `YEAR` is `31557600`, because it takes into a consideration a leap year. This means, that the calculations are: `365.25 * 24 * 60 * 60`, which indeed is `31557600`. The comment should be more descriptive about the `YEAR` value (it should explain, that it takes into consideration a leap year).

# [NC-08] Typos

[File: SurplusGuildMinter.sol](https://github.com/code-423n4/2023-12-ethereumcreditguild/blob/2376d9af792584e3d15ec9c32578daa33bb56b43/src/loan/SurplusGuildMinter.sol#L297)
```
// if the user has been slashed or isnt staking, there is nothing to do
```

`isnt` should be either `is not` or `isn't`.