### Low-01 : Current implementation of `renounceRole()` allows msg.sender to renounce key governor role and admin role, without ensuring roles are granted to other addresses first. In the case of governor keys compromised, key governor functions are at risk of DOS
In src/core/Core.sol,  at the time of deployment, key governor roles are granted to the deployer. The deployer will renounce their roles later on to multi-sig contracts.  But before renouncing their role, deployer will need to grant the governor role to the multi-sig first.

In the case of, a deployer key compromised or oversight during the `renounceRole()`  process, the key governor role account is at risk of being deleted to `address(0)`.

```solidity
    function renounceRole(bytes32 role, address account) public virtual override {
        require(account == _msgSender(), "AccessControl: can only renounce roles for self");
        _revokeRole(role, account);
    }
```
```solidity
    function _revokeRole(bytes32 role, address account) internal virtual {
        if (hasRole(role, account)) {
|>         _roles[role].members[account] = false;
            emit RoleRevoked(role, account, _msgSender());
        }
```
Recommendations:

Consider In `renounceRole(`), also check the member count for a role through `getRoleMemberCount()`. and ensure the count is greater than 1. 

### Low-02: Users can still vote even if a poll already reached the quorum, user will waste their gas in their case, since their transaction doesn't change the poll result.

In src/governance/LendingTermOffboarding.sol- `supportOffboard()`, users can vote to offboard a lending term. However, when a quorum is already reached before the poll expires, a user can still vote through `supportOffboard()`.  The transaction will go through and the state variable will be re-written to the same state as before, leading to user waste gas.

```solidity
    function supportOffboard(
        uint256 snapshotBlock,
        address term
    ) external whenNotPaused {
        require(
            block.number <= snapshotBlock + POLL_DURATION_BLOCKS,
            "LendingTermOffboarding: poll expired"
        );
        uint256 _weight = polls[snapshotBlock][term];
        require(_weight != 0, "LendingTermOffboarding: poll not found");
        uint256 userWeight = GuildToken(guildToken).getPastVotes(
            msg.sender,
            snapshotBlock
        );
        require(userWeight != 0, "LendingTermOffboarding: zero weight");
        require(
            userPollVotes[msg.sender][snapshotBlock][term] == 0,
            "LendingTermOffboarding: already voted"
        );
        userPollVotes[msg.sender][snapshotBlock][term] = userWeight;
        polls[snapshotBlock][term] = _weight + userWeight;
        if (_weight + userWeight >= quorum) {
            canOffboard[term] = true;
        }
        emit OffboardSupport(
            block.timestamp,
            term,
            snapshotBlock,
            msg.sender,
            userWeight
        );
    }
```
As seen above, note that in `supportOffboard()`  require statements only ensure that (1) the poll hasn't expired; (2) the poll is valid; (3) the user hasn't voted before for the poll. 

If the quorum for the poll is already reached `if (_weight + userWeight >= quorum)`  evaluates to true, then `canOffboard[term] = true;` .  However, if `canOffboard[term] ` is already true, `supportOffboard()`  will not check if the poll's quorum is already reached. And if the poll has not expired, the user will be able to continuously vote and `canOffboard[term] ` will be re-written to true. This is a wasteful operation and adding user weights after the term is already approved to be true doesn't matter for the state of the poll.

Recommendations:
In `supportOffboard()` , consider adding a require statement to check `canOffboard[term] `  to be false. For example: Add  `require(canOffboard[term] == false)`.
This prevents already approved poll (quorum reached) to be voted again.

### Low-03: `getRoleAdmin()`  is not overridden as other public functions, and will always return 0 bytes value. Consider overriding `getRoleAdmin()`  implementation to void the implementation.

GuildTimelockController.sol overwrites openzeppelin's TimelockController.sol which overwrites AccessControl.sol.  GuildTimelockController.sol overwrites the role access check to use `hasRole()`  and state variables stored in Core.sol. 

However, not all role access functions from openzeppelin's TimelockController.sol are overwritten.

`getRoleAdmin()`   from AccessControl.sol, is not overwritten in GuildTimelockController.sol.  And  `getRoleAdmin()`   is a public function, is publicly available in GuildTimelockController.sol.  When a user calls  `getRoleAdmin()` , the `_roles[role]` from AccessControl.sol will be called.  But since `_roles[role]`  storage from AccessControl.sol is not meant to be used,  `getRoleAdmin()`  will always return zero.

```solidity
//@audit `_roles` is a state variable in AccessControl.sol, and will not be written to. `getRoleAdmin()`  is not overwritten in GuildTimelockController.sol, calling `getRoleAdmin()` will return bytes32 0
    function getRoleAdmin(bytes32 role) public view virtual override returns (bytes32) {
        return _roles[role].adminRole;
    }
```
Recommendation:
In src/governance/GuildTimelockController.sol,  consider overriding `getRoleAdmin()`   to disable the method, for example reverting the function.

### NC-01: Incomplete comment.  AccessControlEnumerable also has a public `supportsInterface()`  function

In src/core/Core.sol,  there is a comment listing all the functions available in AccessControl.sol. But the comment is missing `supportsInterface()` function. Incomplete comments might be misleading in the code development and auditing process.

```solidity
    // AccessControlEnumerable is AccessControl, and also has the following functions :
    // hasRole(bytes32 role, address account) -> bool
    // getRoleAdmin(bytes32 role) -> bytes32
    // grantRole(bytes32 role, address account)
    // revokeRole(bytes32 role, address account)
    // renounceRole(bytes32 role, address account)
```

Recommendations:
Add `supportsInterface(bytes4 interfaceId)->bool`  in the comment.

